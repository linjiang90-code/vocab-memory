# 项目长期笔记：旅游与日常英语口语学习系统 (vocab-practice)

## 音频朗读铁律（反复踩坑，务必遵守）
- 所有朗读/跟读/慢速一律走**服务器预生成音频** `audio/s{id}.mp3`（id 形如 s1…s100）。
- 🔊 原声伴读 = playAudio(id)（1x）；🐢 慢速/跟读 = playAudio(id, 0.7)（playbackRate）。
- **语速选项全站统一为 1倍 / 0.8倍 / 0.7倍 / 0.6倍**（2026-08-19 改，原为 1.0x/0.75x/0.5x/0.4x）。涉及所有 `<select>` 语速框（day 页 speedSel + 每句 spdSel、master 页 speedSel + spdSel）及 🐢 硬编码值。改时 grep 全仓 `1.0x|0.75x|0.5x|0.4x` 与 `value="0.75"/"0.5"/"0.4"` 与 `playAudio(...,0.75)` 一并替换。
- **严禁在页面里用 `speechSynthesis` / Web Speech API**——远程设备（手机/无英文TTS的电脑）无英文语音包会静默失败，是多次「远程无声」的根因（s16 撇号无声、远程 🐢 全无声、master.html 朗读无声均源于此）。
- voice 包缺失时 speechSynthesis 不会报错只是不出声，极难排查；服务器音频同源跨设备必然可用。
- 生成器（push_day.py / push_today.py / run_daily.py / gen_master_html.py）与共享 cards.js 的新增音频代码必须走 VocabAudio（见下方「调速/慢速铁律」），不要回退到 `new Audio` + `playbackRate`。
- **Audio 元素必须在 DOM 中才能播放（关键！）**：远程浏览器（尤其手机）要求 `<audio>` 元素处于 DOM 树内才真正出声；仅用 JS 变量持有引用（全局缓存）不够——`cards.js` 旧版 `new Audio()` 只存全局 `__audioCache`、不入 DOM，导致 review/calendar **本地有声、远程静音**。`review.html/calendar.html` 朗读经 cards.js 改为 `new Audio()` 后 `__audioHost.appendChild(a)`（隐藏容器 `#__audioHost__` 挂到 body）已根治，并复用 `__audioCache` 避免重复创建。master.html 的 `<audio class="mta">` 也是 appendChild 进卡片 DOM 所以远程有声。
- **严禁** `new Audio()` 临时对象直接 `.play()`（无引用→被 GC 掐断），也**严禁**只缓存到全局变量而不入 DOM。
- 验证手段：用本机 Chrome 以移动端 UA+视口跑 puppeteer-core 无头测试，点 🔊 后断言 `#__audioHost__` 内生成 `audio/s{id}.mp3`、`.mp3` 请求发出且 `playing=true`，可复现远程行为。

## 调速/慢速铁律（2026-08-18 新增，关键！）
- **`<audio>.playbackRate` 在移动端会静默失效**：移动浏览器（含 Edge/Chromium 移动版、iOS Safari）在 `preload="none"` 懒加载、元数据就绪后会把 `playbackRate` 重置回 `defaultPlaybackRate`(1.0)；即使 JS 设置过 0.75，也会被冲掉 → 「慢速/调速还是一倍速」。这是 2026-08-18 全部页面调速失效的根因。
- **唯一正确做法：Web Audio 引擎 `audio-engine.js`**（已新增）。`window.VocabAudio = { play(id,rate,onEnd), loop(id,rate,reps,{onEnd}), stopId(id), stopAll(), isActive }`。内部用 `AudioContext` + `decodeAudioData` + `AudioBufferSourceNode.playbackRate.value`，该速率在**所有**浏览器（移动/桌面 Edge/Chromium/iOS）严格生效，不受懒加载影响。
- **【2026-08-19 下午补充】顺序/循环播放不再单纯依赖 `onended`**：真实 Edge/远程浏览器在程序化连续播放时 `onended` 经常不触发 → 「全部伴读只读一句 / 循环卡住 / 停止无效」。已改为每次播放用「缓冲时长÷(语速)+400ms」启动**兜底定时器**，`onended` 与定时器双保险、只触发一次；每次播放前 `ac()` 恢复 AudioContext。这样无论 `onended` 是否触发，playAll/loop/stop 都正常。验证：无头测试「禁用 onended」后 playAll 仍读完全部 5 句、停止生效、循环生效。
- 所有页面统一调用：`playAudio(id,rate)`/`VocabAudio.loop(sid, parseFloat(curSpeed)||1, reps, {onEnd})`；停止用 `VocabAudio.stopId(sid)`。已覆盖 master.html / review.html / calendar.html / day*.html / day1 / day2 / cards.js。
- 所有页面/生成器必须在主 `<script>` 前引入 `<script src="audio-engine.js"></script>`。**【2026-08-19 漏网踩坑】review.html / calendar.html 曾漏引入 audio-engine.js**，导致卡片 🔊 走 `VocabAudio.play` 但 `VocabAudio` 未定义 → 点击无声/无反应。已在 `gen_views_html.py` 的 review+calendar 两处 `<script src="mastery.js">` 前补 `<script src="audio-engine.js">`（cards.js 依赖 VocabAudio，必须在前）。**改任何页面模板都要 grep 确认 `audio-engine.js` 已出现在每个会调用播放的页面**。
- 生成器 gen_master_html.py / push_day.py / push_today.py / run_daily.py 已同步为 VocabAudio，避免重新生成回退到坏逻辑。
- **严禁**再写 `a.playbackRate = ...` 或 `new Audio(...).playbackRate`（移动端无效）。全站 grep 已确认无 `new Audio` / `playbackRate` / `speechSynthesis` 残留。
- **`stopId` 必须归一化 id（2026-08-19 踩坑）**：`audio-engine.js` 的 `stopKey` 内部要先 `normId(key)`（'1'/'s1' 都能命中）。否则 master/day 页调用 `VocabAudio.stopId(sid)` 传的是**无 `s` 前缀**的原始 id（如 `'1'`），去 `active['1']` 查找，而 `loop()`/`play()` 存入的是 `active['s1']` → 找不到 → 停止/🐢 点击无效、朗读继续。已修复：stopKey 首行 `key = normId(key)`。
- 验证调速：移动端 UA 跑 puppeteer-core，点 🐢 后断言 `playbackRate recorded: [0.75]`（DAY_SLOW_PASS）。
- 验证停止：移动端 UA 跑 puppeteer-core 加载 master.html，点开始→`isActive('1')=true`、按钮「⏹ 停止」；再点停止→`isActive('1')=false`、按钮「▶ 开始」（STOP_PASS ✅）。

## 运行/部署
- 静态服务 serve.py：监听 0.0.0.0:3279，无 Basic Auth（已取消）。已注册 Windows 任务计划程序 vocab-serve（AtStartup + SYSTEM，开机自启+崩溃自重启），并保留每5分钟保活自动化作二级看门狗。
- 端口 3279 经防火墙规则 vocab-practice-3279 放行，公网 IP 125.71.210.3。

## 页面分工
- index.html 首页：今日练习 + 100句总览 + 右上 已学回顾/学习日历。
- master.html：100句总览（每句 🔁 朗读循环，已改服务器音频）。
- day2026-08-XX.html：每日推送练习页（由 push_day.py 生成，服务器音频为主）。
- review.html / calendar.html：已学回顾/学习日历（gen_views_html.py 内嵌数据，无 fetch，任何打开方式都能显示，朗读走 cards.js speakId→playAudio）。
- day1 / day2 / day-demo：早期 demo/旧版，非主线但同样已统一为服务器音频。

## 生成器双轨铁律（2026-08-19 踩坑）
- 项目有**两个**日练习页生成器，各自维护独立 HTML 模板：
  - `push_day.py`：手动运行，模板**完整版**（导航/折叠/朗读控件/进度条全有）
  - `run_daily.py`：自动化定时任务运行，模板曾为**精简版**（缺上述功能）
- **修改日练习页 HTML 模板时必须同步两者**，否则自动化（run_daily.py）会覆盖手动修复的结果。
- 已修复：run_daily.py 的 PAGE 模板已替换为与 push_day.py 一致的完整版。后续改模板时 `grep` 两个文件确认一致。
- **第三个生成器 `gen_future.py`（2026-08-20 新增，预生成/提前预习用）**：它**不自己维护模板**，而是用正则从 `run_daily.py` 提取 `PAGE` 字符串 → 与每日自动化模板**自动同步**（改 run_daily 模板时 gen_future 无需手动改）。生成未来 day<date>.html + 侧车时**不碰 master.json** 的 introduced/mastery，真实日期到时 run_daily 无缝接管覆盖。
- **模板必须使用实际 UTF-8 字符，严禁 \uXXXX 转义（2026-08-22 踩坑）**：Python 源码中的 `\uXXXX` 在源码文本层面是字面量（仅 exec/赋值时解释器才解码）。gen_future.py 用正则提取源码模板时会拿到未解码字面文本 → HTML 里浏览器原样显示为乱码。run_daily.py 自身执行不受影响（解释器会解码），但任何文本级提取都会中招。已将 run_daily.py PAGE 模板 381 处 \uXXXX 全部解码为 UTF-8。后续往模板加中文时直接写汉字。

## 日历提前预习（2026-08-20 新增）
- `gen_future.py` 预生成明天起 N 天（默认22）day 页后，日历自动识别"有 day<date>.json 侧车、晚于今天、且不在 days.json"的日期为 `previewDays`，橙色高亮+可点击 `showDay` 直接学习（同享音频/自评/详情），页头「可预习」快捷条。
- 真实自动化每日跑到该日时会把它登记进 days.json → 日历由橙(预览)转绿(已练习)，过渡无缝。
- 选句与未来自动化一致：首个未引入句起按 id 每5句一批；超总句数/超 introDays 进复习预览（按 id 轮转取5句，使各未来日展示不同句式）。

## serve.py 生成器内存锁铁律（2026-08-19 踩坑，关键！）
- **现象**：改了生成器（如 gen_views_html.py 新增 mastery.js 引用）重跑后，文件在磁盘上正确，但一访问页面/一回写自评，review/calendar 又变回旧版（缺 mastery.js）。
- **根因**：`serve.py` 启动时 `import gen_views_html`，把生成器模块**锁进内存**；`POST /api/mastery` 成功后会调用 `gen_views_html.main()` 重建 review/calendar。若服务**未重启**，内存里仍是旧模块 → 用旧代码覆盖刚生成的文件。即使手动重跑生成器，下次回写又会被覆盖。
- **已修复**：`serve.py` 的 POST 处理里改为 `import importlib; importlib.reload(gen_views_html); gen_views_html.main()`（每次回写都用**磁盘最新源码**）。
- **铁律**：**改动任何生成器（gen_*.py）或共享页面后，必须重启 vocab-serve 服务**（任务计划程序或杀掉监听 3279 的 python 进程再起 serve.py）。否则线上跑的是旧内存模块，改动"看似不生效"。
- 重启方式：结束监听 0.0.0.0:3279 的 python.exe 进程，再 `python serve.py`（后台常驻）；若用任务计划程序 vocab-serve，需 /End + /Run（schtasks 在沙箱可能被禁用，可让用户在"任务计划程序"里手动重启或重启电脑）。

## serve.py 假死排查与自重启（2026-08-19 补充）
- **"打不开"症状**：端口 `netstat` 仍显示 LISTENING、进程活着，但 `curl http://127.0.0.1:3279/` 返回 `HTTP 000`（空响应）。即"占端口却不处理请求"。此时 `taskkill /F /PID <pid>` 杀掉再重启即可。
- **已增强 serve.py**：主循环加 `while True` 自重启（serve_forever 异常→3s 后拉起）；绑定端口前捕获 `OSError(地址已占用)`→直接 exit（避免和每5分钟保活自动化抢端口、自重启空转）；忽略 SIGPIPE。
- 启动命令：`C:/Users/Win10/.workbuddy/binaries/python/versions/3.13.12/python.exe serve.py`（Git Bash 无 setsid，用 Bash 工具 run_in_background 机制常驻；勿用 setsid/nohup，环境中不存在）。
- 若远程/本地都打不开：先 `curl -s -m5 -o /dev/null -w "%{http_code}" http://127.0.0.1:3279/` 确认是 000 还是 200；000 即假死，杀进程重启。

## 掌握度自评跨页同步（2026-08-19 新增）
- 共享模块 `mastery.js`（`window.VocabMastery`）：`assess(sid,action,btn)` / `setBar` / `refreshAll` / `widgetHTML` / `getMap`。
- 交互控件结构：`<div class="assess" data-mid="N">…<div class="mbar" id="mbar-N"><div class="mfill">…</div></div><span class="mbadge" id="mb-N">N/5</span><button class="c/f/u" onclick="VocabMastery.assess('N','clear|fuzzy|unknown',this)">…</button><span class="astat" id="as-N">`。
- 所有列出句子的页面（master 总览 / review 已学回顾 / calendar 学习日历 / day* 当日练习）都**可交互自评**，并走 `POST/GET /api/mastery` 与 master.json 实时同步，各页互相同步。
- 控件 id 约定用**数字字符串**（mbar-4，不带 s 前缀）；API 用数字 id。
- index.html 是入口菜单（不列具体句子），仅显示回写服务状态，不需逐句自评。
- 修改自评控件样式：master.html 内联 style + `cards.css` 的 `.assess` 两处都要改。

