# 项目长期笔记：旅游与日常英语口语学习系统 (vocab-practice)

## 音频朗读铁律（反复踩坑，务必遵守）
- 所有朗读/跟读/慢速一律走**服务器预生成音频** `audio/s{id}.mp3`（id 形如 s1…s100）。
- 🔊 原声伴读 = playAudio(id)（1x）；🐢 慢速/跟读 = playAudio(id, 0.75)（playbackRate）。
- **严禁在页面里用 `speechSynthesis` / Web Speech API**——远程设备（手机/无英文TTS的电脑）无英文语音包会静默失败，是多次「远程无声」的根因（s16 撇号无声、远程 🐢 全无声、master.html 朗读无声均源于此）。
- voice 包缺失时 speechSynthesis 不会报错只是不出声，极难排查；服务器音频同源跨设备必然可用。
- 生成器（push_day.py / push_today.py / run_daily.py / gen_master_html.py）与共享 cards.js 的新增音频代码必须复用上述模式。
- **Audio 元素必须在 DOM 中才能播放（关键！）**：远程浏览器（尤其手机）要求 `<audio>` 元素处于 DOM 树内才真正出声；仅用 JS 变量持有引用（全局缓存）不够——`cards.js` 旧版 `new Audio()` 只存全局 `__audioCache`、不入 DOM，导致 review/calendar **本地有声、远程静音**。`review.html/calendar.html` 朗读经 cards.js 改为 `new Audio()` 后 `__audioHost.appendChild(a)`（隐藏容器 `#__audioHost__` 挂到 body）已根治，并复用 `__audioCache` 避免重复创建。master.html 的 `<audio class="mta">` 也是 appendChild 进卡片 DOM 所以远程有声。
- **严禁** `new Audio()` 临时对象直接 `.play()`（无引用→被 GC 掐断），也**严禁**只缓存到全局变量而不入 DOM。
- 验证手段：用本机 Chrome 以移动端 UA+视口跑 puppeteer-core 无头测试，点 🔊 后断言 `#__audioHost__` 内生成 `audio/s{id}.mp3`、`.mp3` 请求发出且 `playing=true`，可复现远程行为。

## 运行/部署
- 静态服务 serve.py：监听 0.0.0.0:3279，无 Basic Auth（已取消）。已注册 Windows 任务计划程序 vocab-serve（AtStartup + SYSTEM，开机自启+崩溃自重启），并保留每5分钟保活自动化作二级看门狗。
- 端口 3279 经防火墙规则 vocab-practice-3279 放行，公网 IP 125.71.210.3。

## 页面分工
- index.html 首页：今日练习 + 100句总览 + 右上 已学回顾/学习日历。
- master.html：100句总览（每句 🔁 朗读循环，已改服务器音频）。
- day2026-08-XX.html：每日推送练习页（由 push_day.py 生成，服务器音频为主）。
- review.html / calendar.html：已学回顾/学习日历（gen_views_html.py 内嵌数据，无 fetch，任何打开方式都能显示，朗读走 cards.js speakId→playAudio）。
- day1 / day2 / day-demo：早期 demo/旧版，非主线但同样已统一为服务器音频。
