# 项目长期笔记：旅游与日常英语口语学习系统 (vocab-practice)

## 音频朗读铁律
- 朗读/跟读/慢速一律走服务器预生成音频 `audio/s{id}.mp3`（id 形如 s1…s1000）。
- 语速统一 1/0.8/0.7/0.6 倍：🔊=playAudio(id) 1x，🐢=playAudio(id,0.7)。
- 严禁 speechSynthesis/Web Speech API（远程设备无英文TTS→静默失败，极难排查）。
- `<audio>` 必须 appendChild 进 DOM 才出声（远程浏览器要求）；严禁 new Audio() 临时对象直接 play 或只存全局变量不入 DOM。
- 统一用 Web Audio 引擎 `audio-engine.js`(`window.VocabAudio`：play/loop/stopId/stopAll/isActive)。移动端 `<audio>.playbackRate` 静默失效，必须用 AudioBufferSourceNode.playbackRate。所有页面在主 script 前引 `audio-engine.js`；stopId 内部先 normId('1'/'s1' 都命中)。顺序播放用 onended+兜底定时器双保险。
- 验证：puppeteer-core 移动端 UA，点 🔊 断言 `#__audioHost__` 内生成 `audio/s{id}.mp3` 且 playing=true。
- **互斥播放（2026-11-10 修）**：`play()`/`loop()` 起播前必须 `stopAll()`，否则点第二句时第一句继续响。`stopAll(keepId)` 的 keepId 用于 loop 保留自己按钮态；`gen` 代次令牌作废「解码期间已被新点击取代」的旧请求（解码是异步的，不加会竞态叠加）；回退 `<audio>` 路径用 `stopAllFallback(keepId)` 同步互斥；`resetRunBtns(keepId)` 让旧页面按钮也能自动复位。回归测试：`node test_audio_exclusive.js`（4 场景）。
- **日页内联 JS 改造要批量补历史页**：模板改动只影响新生成页，`day*.html` 已有 100+ 个；用字符串替换批量修（**别用 push_day 重生成——会 mastery+1 / 改 introduced，污染学习数据**）。改完跑 `node --check` 校验内联脚本语法。
- 音频按需生成：run_daily 选中句缺音频时自动调 `gen_one.py`（VENV_PY + edge-tts）；新激活批次可提前批量预生成。
- **音色统一配置（2026-11-11 起）**：音色/语速只写在 `voice_config.py`（`VOICE`/`RATE`/`PITCH`），`gen_one.py`、`gen_audio.py`、`gen_audio_force.py` 全部 import 它，**禁止再在脚本里硬编码音色**。当前 **`en-US-AvaNeural` 女声 + `rate=-15%`**（用户 2026-11-11 选定，要求吐词清晰/好听/语速适中/连读别太快）；旧男声 Andrew、更早 Aria 备份在 `audio_backup_aria/`（已 gitignore）。
- **换音色必须全量重录**：`venv 解释器 regen_voice.py`（默认重录 1..activeCount+50，可传区间；并发3+重试3+写 .tmp 再 `os.replace` 原子替换，避免播放中读到半个文件）。重录后核对条数/大小/mtime，并确认服务端 `Cache-Control: no-store`（serve.py end_headers 对全类型生效，mp3 也不会被缓存，无需 bump BUILD）。
- **客户端缓存铁律（2026-11-12 二次加固，最终形态）**：企业微信内置浏览器（PC 端 CEF / 手机端 X5）的**媒体缓存按 URL 路径建键**，无视 `Cache-Control: no-store`，**也不认 `?v=` query** —— 所以昨天换 Ava 女声后内嵌浏览器仍放旧男声，外部浏览器却正常（用户实测确认）。
  - 最终解法：**路径级版本段** `audio/b<BUILD>/sNNN.mp3`。引擎从自身 `<script src="audio-engine.js?v=xxx">` 读版本号（单一事实来源），不硬编码。
  - `sync_audio_version.py` 用**硬链接**生成 `audio/b<BUILD>/` 目录（不额外占磁盘，gitignore 已忽略 `audio/b*/`）→ 新路径在磁盘上真实存在，**旧版 serve.py 未重启也能当静态文件返回**，这是免重启生效的关键。
  - `bump_build.py`：一键同步 ① serve.py BUILD ② 全部页面/模板引擎引用 ③ 音频版本目录；`regen_voice.py` 重录后**自动调用**（`--no-bump` 可跳过）。换音色只需改 `voice_config.py` + 跑 regen_voice.py，不会再漏。
  - serve.py 另有 `/audio/<段>/sNNN.mp3 → audio/sNNN.mp3` 剥段路由兜底（改动需重启生效）。
  - 引擎 404 自动回退扁平路径 `audio/sNNN.mp3`（场景6 已覆盖），保证「页面先更新、服务后重启」窗口期朗读不挂。
  - 推送卡片链接改为 `day<date>.html?v=<BUILD>`（`wecom_push._day_link()`），绕过内置浏览器 HTML 缓存。
  - 排查口诀：磁盘 mtime → 现场重录字节数 → curl 服务端 → 都对了就是客户端缓存；此时**换路径**才是解，清缓存（`debugx5.qq.com` / 企业微信设置→清缓存）只是治症状。

## 语料与选句机制（2026-09-10 起）
- **语料 1000 句**（ids 1-1000，原始 100 句 + 900 新句，0 重复；新句来源 sentences_pool_1..9.py，由 expand_corpus.py 幂等合并）。
- **每日 5 句随机**：`random.Random(dayIndex)` + `sample(可用ids, 5)`，同日幂等、跨日随机；run_daily / push_day / gen_future 三处公式必须完全一致（改一处必须同步另两处）。
- **每 30 天解锁 +50**：`meta.activeCount` 在 `nextExpansionDay`（已从 31 走到 **61**，即 2026-10-12）起每次 +50 封顶 1000，while 循环可补漏跑。**当前 activeCount=150、expansionsDone=1**。新句被随机抽中即视为引入（introduced 置位）。
- 语料已**全量预置**（ids 1-1000）。增强 ENH 已撰写 **200/1000**（ids 1–200 带完整 fullIpa/variants/scenes/grammar/pron；2026-10-10 回填 151–200 应对 Day61=10-12 解锁），ids 201–1000 待对应解锁批次（+50/30天）逐步回填。run_daily.py 内 ENH 字典仅覆盖 id 6–10，是历史兜底（被选中句缺 enh 时走它）。
- 分类体系沿用「主题/子类」中文标签（travel：机场/酒店/餐厅/购物/观光/应急…；daily：寒暄/邀约/情绪/表达/俗语…）。

## 运行/部署
- serve.py 监听 0.0.0.0:3279（无 Basic Auth）。Windows 任务计划程序 vocab-serve（AtStartup+SYSTEM 开机自启+崩溃自重启），每5分钟保活自动化二级看门狗。端口 3279 防火墙放行，公网 125.71.210.3。
- 沙箱禁用 system 工具，AI 无法重启服务；改 serve.py/生成器后需用户在「任务计划程序」/End+/Run 或重启电脑。
- 外部活文档接入（2026-09-09）：`GET /cars-2026.html` 动态转发 `CARS_SRC`（车展项目 `2026-08-21-13-30-20/2026市售车型统计表.html`），每次现读、始终最新；仓内 `cars-2026.html` 仅是重启前的快照兜底，路由优先。

## 页面分工
- index.html 首页；master.html 100句总览(🔁循环)；day2026-08-XX.html 每日练习(push_day.py/run_daily.py 生成)；review.html/calendar.html 已学回顾/日历(gen_views_html.py 内嵌数据、无 fetch)；day1/day2/day-demo 旧版。

## 生成器双轨 + 模板铁律（关键）
- 两个日页生成器：push_day.py(手动,完整版) + run_daily.py(自动化,完整版，须与 push_day 一致)。改日页模板必须同步两者。
- **收尾动作也必须双轨对齐（2026-09-22 修）**：push_day.py 收尾会 ①写 `day<date>.json` 侧车 ②去重更新 `days.json` ③重生成 master.html ④重生成 gen_views_html.py。**run_daily.py 原缺 ① ② ④**（只重生成 master.html），导致 09-16 起日页有页无侧车 → calendar.html 那几天点不动、也不进「最近练习」。现已给 run_daily.py 补齐四步。**新增/修改日页收尾逻辑时，两端必须一起改。**
- gen_future.py(预生成/预习)：正则从 run_daily.py 提取 PAGE 模板字符串（非执行）→ 与每日自动化自动同步；生成未来 day<date>.html+侧车，不碰 master.json。参数 `gen_future.py 2026-08-22 1` 可单补某日。它另有自愈块（源码尾部）：为「有日页无侧车」的日期反解 ids 补侧车，并把 ≤今天且未登记的日期写进 days.json → **日历出现死格/最近练习缺日时，先跑 gen_future.py 自愈，再跑 gen_views_html.py**。
- **模板必须用真实 UTF-8 字符，严禁 \uXXXX / \U00XXXXXXXX 转义**：gen_future 取源码字面量，未解码转义会原样写进 HTML 变乱码。run_daily.py 执行时解释器会解码(自生成正常)，但文本级提取会中招。往模板加中文/emoji 直接写汉字和符号。
- 排查模板转义残留：Python `open(f,'rb')` + `re.compile(rb'\\\u[0-9a-fA-F]{4}|\\\U00[0-9a-fA-F]{8}')` 字节级扫描（Read/Grep 会自动解码隐藏问题）。

## 日历提前预习（2026-08-20 新增）
- `gen_future.py` 预生成明天起 N 天（默认22）day 页后，日历自动识别「有 day<date>.json 侧车、晚于今天、且不在 days.json」的日期为 `previewDays`，橙色高亮+可点击 `showDay` 直接学习，页头「可预习」快捷条。
- 真实自动化每日跑到该日时会把它登记进 days.json → 日历由橙(预览)转绿(已练习)，过渡无缝。
- 选句与真实自动化完全一致（随机公式见「语料与选句机制」节）。
- **孤儿日容错（2026-08-25 修）**：若每日自动化漏跑，某天有侧车却没写进 days.json、且日期已过 → 日历成空格/点不了。gen_views_html.py 已加 `recovered_days`（有侧车+日期≤今天+未登记→绿色可点击、无 🔓 横幅），JS 点击守卫 `isPrac||isPrev||isRec`。出现死格先查 days.json 是否漏登。

## 乱码根因最终定论（2026-08-24）
- 现象：day2026-08-22 起未来页中文/emoji 字面量乱码(\U0001f5a3、\u9996\u9875)。曾误判为 charset/浏览器缓存（已排除）。
- 真因：run_daily.py PAGE 模板中文(\uXXXX 4位)+emoji(\U00XXXXXXXX 8位)以转义字面量存储；gen_future 提取未解码→写 HTML。已两次解码修全(先4位漏8位)，重生成25页，磁盘+线上均 0 转义。
- serve.py 辅助防缓存：*.html 302 跳转 ?v=BUILD + charset=utf-8 + no-store（BUILD=**20261010a**，2026-10-10 千人库扩容+模板重写后撞缓存；改 serve.py 需重启生效）。线上验证 day2026-08-22/09-11 均 302→干净内容。

## 企业微信推送（2026-11-09 接入）
- 用户用的是**智能机器人 · API 模式（长连接）**，不是群机器人 Webhook。协议：`wss://openws.work.weixin.qq.com` → `aibot_subscribe{bot_id,secret}`（失败码 **853000 = 凭证错**）→ 30s `ping` 保活 → 收 `aibot_msg_callback` → 推送用 `aibot_send_msg`（chatid/chat_type/msgtype=markdown/`markdown.content`，≤20480B，30 条/分钟）。
- 模块：`wecom_bot.py`（长连接客户端，aiohttp；**必须先起读循环再发订阅**，否则回执没人消费会被误报成 ack timeout）、`wecom_push.py`（组装每日 5 句卡片；优先走本地 `/api/wecom/push`，服务不可用时降级直连并重试 3 次）。
- `serve.py` 常驻持有长连接：`GET /api/wecom/state`、`POST /api/wecom/push`，按关键词自动回卡（今日/明天）。改后需重启 vocab-serve。
- 凭证 `wecom_config.json`、chatid `wecom_state.json`，**均在 .gitignore**（repo 公开）。chatid=`LinJiang`、chat_type=1（单聊）。主动推送前置条件：用户须先给机器人发过消息。
- **已知冲突**：WorkBuddy.exe 自己也连着同一机器人（PID 可查，netstat 116.31.68.62:443），该机器人同时只允许 1 条长连接 → 互踢 + 偶发 ack timeout，因此推送必须重试。若长期不稳，替代方案是群机器人 Webhook（纯 HTTP，无独占）。
- 每日 9:00 自动化只备数据（生成页+预览），**不再推企业微信**；卡片由 13 条 `英语提醒·HH:30`（09:30–21:30）DAILY 任务统一推送。
- **每小时提醒**：`automation_update` 的 rrule **不支持多值 BYHOUR**；`FREQ=HOURLY;BYMINUTE=30` 会被接受但**按创建时间锚定**（BYMINUTE 被忽略）；只有 `FREQ=DAILY;BYHOUR=h;BYMINUTE=m` 精确。故用 13 条单点任务。
- `wecom_push.py` 4 形态轮换 `mode=(hour-9)%4`：0 全览 / 1 单句精讲 / 2 中译英自测 / 3 发音要点；`--window` 时段守卫（非 9–21 点返回 skipped）；`_resolve_day()` 当天侧车缺失时回退最近一期并注明。
- **卡片 markdown 铁律**：aibot（智能机器人）的 markdown **不支持任何 HTML**，`<font color="comment">` 只属群机器人 Webhook，用了会被原样显示成标签文本（用户报「乱码」就是这个）。只准用 `#`/`**`/`>`/`[]()` 标准语法，音标行用 `　🔈 far /fɑr/` 纯文本；`wecom_push.py` 已有 `_sanitize_md()` 正则兜底清洗标签。

## serve.py 两份铁律
- 生成器内存锁：serve.py 启动 import gen_views_html 锁内存，POST /api/mastery 回写后 importlib.reload 再 main()。改任何 gen_*.py/共享页面必须重启 vocab-serve 否则线上跑旧模块。
- 假死：端口 LISTENING 但 curl 返回 000 → taskkill 重启。serve.py 已加 while True 自重启 + 端口占用即 exit。

## 掌握度自评跨页同步
- mastery.js(window.VocabMastery)：assess/setBar/refreshAll/widgetHTML/getMap。控件 id 用数字字符串(mbar-4 不带 s)，API 用数字 id。所有列句页面可交互自评走 /api/mastery 实时同步 master.json。改样式：master.html 内联 + cards.css .assess 两处。
