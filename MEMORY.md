# 项目长期笔记：旅游与日常英语口语学习系统 (vocab-practice)

## 音频朗读铁律
- 朗读/跟读/慢速一律走服务器预生成音频 `audio/s{id}.mp3`（id 形如 s1…s100）。
- 语速统一 1/0.8/0.7/0.6 倍：🔊=playAudio(id) 1x，🐢=playAudio(id,0.7)。
- 严禁 speechSynthesis/Web Speech API（远程设备无英文TTS→静默失败，极难排查）。
- `<audio>` 必须 appendChild 进 DOM 才出声（远程浏览器要求）；严禁 new Audio() 临时对象直接 play 或只存全局变量不入 DOM。
- 统一用 Web Audio 引擎 `audio-engine.js`(`window.VocabAudio`：play/loop/stopId/stopAll/isActive)。移动端 `<audio>.playbackRate` 静默失效，必须用 AudioBufferSourceNode.playbackRate。所有页面在主 script 前引 `audio-engine.js`；stopId 内部先 normId('1'/'s1' 都命中)。顺序播放用 onended+兜底定时器双保险。
- 验证：puppeteer-core 移动端 UA，点 🔊 断言 `#__audioHost__` 内生成 `audio/s{id}.mp3` 且 playing=true。

## 运行/部署
- serve.py 监听 0.0.0.0:3279（无 Basic Auth）。Windows 任务计划程序 vocab-serve（AtStartup+SYSTEM 开机自启+崩溃自重启），每5分钟保活自动化二级看门狗。端口 3279 防火墙放行，公网 125.71.210.3。
- 沙箱禁用 system 工具，AI 无法重启服务；改 serve.py/生成器后需用户在「任务计划程序」/End+/Run 或重启电脑。

## 页面分工
- index.html 首页；master.html 100句总览(🔁循环)；day2026-08-XX.html 每日练习(push_day.py/run_daily.py 生成)；review.html/calendar.html 已学回顾/日历(gen_views_html.py 内嵌数据、无 fetch)；day1/day2/day-demo 旧版。

## 生成器双轨 + 模板铁律（关键）
- 两个日页生成器：push_day.py(手动,完整版) + run_daily.py(自动化,完整版，须与 push_day 一致)。改日页模板必须同步两者。
- gen_future.py(预生成/预习)：正则从 run_daily.py 提取 PAGE 模板字符串（非执行）→ 与每日自动化自动同步；生成未来 day<date>.html+侧车，不碰 master.json。参数 `gen_future.py 2026-08-22 1` 可单补某日。
- **模板必须用真实 UTF-8 字符，严禁 \uXXXX / \U00XXXXXXXX 转义**：gen_future 取源码字面量，未解码转义会原样写进 HTML 变乱码。run_daily.py 执行时解释器会解码(自生成正常)，但文本级提取会中招。往模板加中文/emoji 直接写汉字和符号。
- 排查模板转义残留：Python `open(f,'rb')` + `re.compile(rb'\\\u[0-9a-fA-F]{4}|\\\U00[0-9a-fA-F]{8}')` 字节级扫描（Read/Grep 会自动解码隐藏问题）。

## 日历提前预习（2026-08-20 新增）
- `gen_future.py` 预生成明天起 N 天（默认22）day 页后，日历自动识别「有 day<date>.json 侧车、晚于今天、且不在 days.json」的日期为 `previewDays`，橙色高亮+可点击 `showDay` 直接学习，页头「可预习」快捷条。
- 真实自动化每日跑到该日时会把它登记进 days.json → 日历由橙(预览)转绿(已练习)，过渡无缝。
- 选句与未来自动化一致：首个未引入句起按 id 每5句一批；超总句数/超 introDays 进复习预览（按 id 轮转取5句，使各未来日展示不同句式）。
- **孤儿日容错（2026-08-25 修）**：若每日自动化漏跑，某天有侧车却没写进 days.json、且日期已过 → 日历成空格/点不了。gen_views_html.py 已加 `recovered_days`（有侧车+日期≤今天+未登记→绿色可点击、无 🔓 横幅），JS 点击守卫 `isPrac||isPrev||isRec`。出现死格先查 days.json 是否漏登。

## 乱码根因最终定论（2026-08-24）
- 现象：day2026-08-22 起未来页中文/emoji 字面量乱码(\U0001f5a3、\u9996\u9875)。曾误判为 charset/浏览器缓存（已排除）。
- 真因：run_daily.py PAGE 模板中文(\uXXXX 4位)+emoji(\U00XXXXXXXX 8位)以转义字面量存储；gen_future 提取未解码→写 HTML。已两次解码修全(先4位漏8位)，重生成25页，磁盘+线上均 0 转义。
- serve.py 辅助防缓存：*.html 302 跳转 ?v=BUILD + charset=utf-8 + no-store（BUILD=20260824b；改 serve.py 需重启生效）。线上验证 day2026-08-22/09-11 均 302→干净内容。

## serve.py 两份铁律
- 生成器内存锁：serve.py 启动 import gen_views_html 锁内存，POST /api/mastery 回写后 importlib.reload 再 main()。改任何 gen_*.py/共享页面必须重启 vocab-serve 否则线上跑旧模块。
- 假死：端口 LISTENING 但 curl 返回 000 → taskkill 重启。serve.py 已加 while True 自重启 + 端口占用即 exit。

## 掌握度自评跨页同步
- mastery.js(window.VocabMastery)：assess/setBar/refreshAll/widgetHTML/getMap。控件 id 用数字字符串(mbar-4 不带 s)，API 用数字 id。所有列句页面可交互自评走 /api/mastery 实时同步 master.json。改样式：master.html 内联 + cards.css .assess 两处。
