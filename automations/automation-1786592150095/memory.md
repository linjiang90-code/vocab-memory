# 自动化执行记录：每日英语口语推送（词力词汇教练）

## 最近执行：2026-09-09
- **dayIndex = 28**（today 2026-09-09 − startDate 2026-08-13 + 1）
- **模式**：review（dayIndex 28 > introDays 20，随机复习，非新学）
- **阶段扩展**：未触发（nextExpansionDay=31 ≠ 28；expansionsDone=0，还差 3 天到扩展日）
- **选中句**：[91, 92, 93, 94, 95]（review 池按 mastery 升序+reviewCount 降序；91–95 此前未复习过 mastery=0 最高优先，全 daily/short）
  - s91 What do you mean by that?（澄清/追问, short）
  - s92 Just to be clear...（澄清/说明, short）
  - s93 I'm afraid I have to go now.（告别/离开, short）
  - s94 See you later!（告别, short）
  - s95 Take care!（告别/关心, short）
- **增强内容**：5 句 enh 均 COMPLETE（base 100 句预置 fullIpa/variants/scenes/grammar/pron，非空）
- **音频**：s91–s95.mp3 均存在，无需新生成
- **写回 master.json**：5 句 lastReviewed=2026-09-09、reviewCount 0→1、mastery 0→1、introduced 保持 true（today_new=0）
- **生成页**：run_daily.py 覆盖旧 gen_future 预览页（原 Aug 24）→ day2026-09-09.html（mtime 09:11）✓；gen_master_html.py 重生成 master.html（404KB，mtime 09:11）✓
- **服务**：端口 3279 回写服务 `curl /api/status` 返回 {"ok":true,"port":3279}，已运行无需重启
- **幂等守卫**：执行前校验 master.json 无 lastReviewed==2026-09-09 句 + day 页为旧预览页(Aug 24) → 今日首次推送，安全执行（无重复 mastery 自增）

## 最近执行：2026-09-10
- **dayIndex = 29**（today 2026-09-10 − startDate 2026-08-13 + 1）
- **模式**：review（dayIndex 29 > introDays 20，随机复习，非新学）
- **阶段扩展**：未触发（nextExpansionDay=31 ≠ 29；expansionsDone=0，还差 2 天到扩展日）
- **选中句**：[46, 47, 48, 49, 50]（review 池按 mastery 升序+reviewCount 降序；46–50 此前 mastery=1 优先轮转）
  - s46 Where is the nearest restroom?（问厕所在哪, short）
  - s47 I lost my wallet.（丢钱包, short）
  - s48 I need to see a doctor.（看病, short）
  - s49 Call the police, please.（报警, short）
  - s50 I've missed my flight.（误机, short）
- **增强内容**：5 句 enh 均 COMPLETE（fullIpa/variants(3)/scenes(3)/grammar/pron 全非空）
- **音频**：s46–s50.mp3 均存在，无需新生成
- **写回 master.json**：5 句 lastReviewed=2026-09-10、reviewCount 1→2、mastery 1→2、introduced 保持 true（today_new=0）
- **生成页**：run_daily.py 覆盖旧 gen_future 预览页 → day2026-09-10.html（mtime 09:07:04）✓；gen_master_html.py 重生成 master.html（404KB，mtime 09:07:19）✓
- **服务**：端口 3279 回写服务 `curl /api/status` 返回 {"ok":true,"port":3279}，已运行无需重启
- **幂等守卫**：执行前校验 master.json 无 lastReviewed==2026-09-10 句 + day2026-09-10.html 为 gen_future 预览页 → 今日首次推送，安全执行（无重复 mastery 自增）
- **连续学习**：streak=12 天（09-10 回溯至 08-30 连续）；累计 distinct review 日 16；introduced 100/100

## 最近执行：2026-09-08
- **dayIndex = 27**（today 2026-09-08 − startDate 2026-08-13 + 1）
- **模式**：review（dayIndex 27 > introDays 20，随机复习，非新学）
- **阶段扩展**：未触发（nextExpansionDay=31 ≠ 27；expansionsDone=0）
- **选中句**：[86, 87, 88, 89, 90]（review 池按 mastery 升序+reviewCount 降序；86–90 此前未复习过 mastery=0 最高优先）
  - s86 I couldn't agree more.（回应/赞同, short）
  - s87 Let me think about it.（犹豫/思考, short）
  - s88 I'll get back to you as soon as I confirm the details with my manager.（回应/回复, long）
  - s89 Could you speak up, please?（沟通/音量, short）
  - s90 I didn't catch your name.（沟通/没听清, short）
- **增强内容**：5 句 enh 均 COMPLETE（base 100 句预置 fullIpa/variants(3)/scenes(3)/grammar/pron 全非空）
- **音频**：s86–s90.mp3 均存在，无需新生成
- **写回 master.json**：5 句 lastReviewed=2026-09-08、reviewCount 0→1、mastery 0→1、introduced 保持 true（today_new=0）
- **生成页**：run_daily.py 覆盖旧 gen_future 预览页 → day2026-09-08.html（mtime 09:04:28）✓；gen_master_html.py 重生成 master.html（mtime 09:04:28）✓
- **服务**：端口 3279 回写服务 `curl /api/status` 返回 {"ok":true,"port":3279}，已运行无需重启
- **幂等守卫**：执行前校验 master.json 无 lastReviewed==2026-09-08 句 → 今日首次推送，安全执行（无重复 mastery 自增）

## 最近执行：2026-09-05
- **dayIndex = 24**（today 2026-09-05 − startDate 2026-08-13 + 1）
- **模式**：review（dayIndex 24 > introDays 20，随机复习，非新学）
- **阶段扩展**：未触发（nextExpansionDay=31 ≠ 24；expansionsDone=0）
- **选中句**：[71, 72, 73, 74, 75]（review 池按 mastery 升序+reviewCount 降序；66–100 组 mastery=1 优先轮转）
  - s71 I'm stuck in traffic, so I might be a few minutes late.（堵车迟到, long）
  - s72 How was your day?（问候, short）
  - s73 I had a long day at work.（工作闲聊, short）
  - s74 Could you do me a favor?（请求帮忙, short）
  - s75 Sure, no problem.（爽快答应, short）
- **增强内容**：5 句 enh 均 COMPLETE（源自 09-03 补学批量注入，fullIpa/variants/scenes/grammar/pron 全非空）
- **音频**：s71–s75.mp3 均存在，无需新生成
- **写回 master.json**：5 句 lastReviewed=2026-09-05、reviewCount 0→1、mastery 0→1、introduced 保持 true（today_new=0）
- **生成页**：run_daily.py 覆盖旧 gen_future 预览页 → day2026-09-05.html ✓；gen_master_html.py 重生成 master.html（404KB）✓
- **服务**：端口 3279 回写服务 `curl /api/status` 返回 ok，已运行，无需重启
- **幂等守卫**：执行前校验 master.json 无 lastReviewed==2026-09-05 句 → 今日首次推送，安全执行

## 最近执行：2026-09-03
- **dayIndex = 22**（today 2026-09-03 − startDate 2026-08-13 + 1）
- **模式**：review（dayIndex 22 > introDays 20，随机复习，非新学）
- **阶段扩展**：未触发（nextExpansionDay=31 ≠ 22）
- **选中句**：[16, 8, 9, 10, 11]（按 mastery 升序+reviewCount 降序取前5，全为 introduced 句）
  - s16 My luggage didn't arrive. (行李未到, short)
  - s8 Could you recommend a good local restaurant… (推荐餐厅, long)
  - s9 I was wondering if you'd like to grab a coffee… (邀约喝咖啡, long)
  - s10 I seem to have lost my way… (迷路问路, long)
  - s11 Where can I buy a ticket to the city center? (买票, short)
- **增强内容**：5 句 enh 均已齐备（fullIpa/variants/scenes/grammar/pron）
- **音频**：s8/s9/s10/s11/s16.mp3 均存在，无需新生成
- **写回 master.json**：5 句 lastReviewed=2026-09-03、reviewCount+1、mastery+1（s16: m1→2, rc→7；s8/s9/s10/s11: m1→2, rc→2）
- **生成页**：day2026-09-03.html（25.6KB）✓；gen_master_html.py 重生成 master.html（404KB）✓
- **服务**：端口 3279 回写服务 `curl /api/status` 返回 ok，已运行，无需重启
- **数据观察**：原始 learned=65/100，35 句（id 66–100，全 daily 主题，32 short+3 long）因 introDays 内漏跑未 introduced。
- **补学修复（同次执行追加）**：经核查，id 66–100 已具备完整 enh（fullIpa/variants/scenes/grammar/pron 均非空）+ 音频文件齐全，仅 introduced 标志未翻转。已将 35 句 introduced=true（introducedDay=22、lastReviewed=null、reviewCount=0、mastery=0、dueDate=null），使其以最高优先进入复习轮转（后续每日自动优先复习这批）。master.json 写回后重跑 gen_master_html.py + gen_views_html.py：learned 65→100/100，review.html=100 learned，calendar 已同步。修复可逆（改回 introduced=false 即可）。下次扩展日仍为 day31。

## 最近执行：2026-09-02
- **dayIndex = 21**（today 2026-09-02 − startDate 2026-08-13 + 1）
- **模式**：review（dayIndex 21 > introDays 20，随机复习，非新学）
- **阶段扩展**：未触发（nextExpansionDay=31 ≠ 21）
- **选中句**：[16, 3, 4, 6, 7]（按 mastery 升序+reviewCount 降序取前5）
  - s16 My luggage didn't arrive. (mastery 0→1, 行李未到)
  - s3 I'd like to book a table for two. (订位)
  - s4 It's nice to meet you. (初识)
  - s6 What do you usually do on weekends? (周末)
  - s7 Is breakfast included in the price? (含早)
- **增强内容**：5 句 enh 均已齐备（fullIpa/variants/scenes/grammar/pron）
- **音频**：s3/s4/s6/s7/s16.mp3 均存在，无需新生成
- **写回 master.json**：5 句 introduced=true、lastReviewed=2026-09-02、reviewCount+1、mastery+1
- **生成页**：day2026-09-02.html（24.5KB）✓；gen_master_html.py 重生成 master.html（404KB）✓
- **服务**：端口 3279 回写服务 `curl /api/status` 返回 ok，已运行，无需重启

## 关键约定（跨次执行一致）
- 回写服务端口 **3279**（非 8765）；所有 api 用 http://127.0.0.1:3279/
- 驱动脚本 `run_daily.py` 已封装选句+增强+音频+当日页+写回+重生成 master.html
- ENH 字典仅覆盖 id 6–10；其余句 enh 由历史运行写回 master.json
- 新学阶段 dayIndex≤20 顺序取未引入句；>20 进入随机复习池
- 扩展日：dayIndex == nextExpansionDay(31/61/91…) 触发 +50 句，NEW_SENTENCES 空时跳过

## 本次执行：2026-09-02（二次触发 → 跳过防重复）
- 启动即检测到 day2026-09-02.html 已于 08:53 生成；master.json 中 5 句 lastReviewed=2026-09-02（s3/s4/s6/s7/s16）状态完整，音频与 enh 齐全，**判定为重复触发**。
- **未重跑 run_daily.py**：该脚本无幂等守卫，重跑会换选不同 5 句 + 重复 mastery 自增，损坏数据。直接复用既有推送结果交付任务卡。
- 服务 3279 正常（/api/status ok）；扩展未触发（nextExpansionDay=31≠21）。
- 数据观察：learned=65/100，introDays=20 内未全学完（约 7 天自动化漏跑），35 句未 introduced 且 review 阶段不再进入新学池 → 后续建议补学或加幂等守卫。

## 最近执行：2026-09-04
- **dayIndex = 23**（today 2026-09-04 − startDate 2026-08-13 + 1）
- **模式**：review（dayIndex 23 > introDays 20，随机复习，非新学）
- **阶段扩展**：未触发（nextExpansionDay=31 ≠ 23；expansionsDone=0）
- **选中句**：[66, 67, 68, 69, 70]（review 池按 mastery 升序+reviewCount 降序；此批为 09-03 补学的 66–100 组，mastery=0 最高优先）
  - s66 Are you free this weekend?（周末邀约, short）
  - s67 Sorry, I can't make it.（婉拒, short）
  - s68 Maybe another time.（改期, short）
  - s69 What's the weather like today?（天气闲聊, short）
  - s70 It's raining cats and dogs.（天气闲聊, short）
- **增强内容**：5 句 enh 均 COMPLETE（fullIpa/variants(3)/scenes(3)/grammar/pron 全非空，源自 09-03 补学批量注入）
- **音频**：s66–s70.mp3 均存在，无需新生成
- **写回 master.json**：5 句 lastReviewed=2026-09-04、reviewCount 0→1、mastery 0→1、introduced 保持 true（today_new=0）
- **生成页**：run_daily.py 覆盖旧 gen_future 预览页 → day2026-09-04.html（23.5KB，原 24.6KB）✓；gen_master_html.py 重生成 master.html（404KB）✓
- **服务**：端口 3279 回写服务 `curl /api/status` 返回 ok，已运行，无需重启
- **幂等守卫**：执行前校验 master.json 无 lastReviewed==2026-09-04 句 → 确认今日首次推送，安全执行（避免重复 mastery 自增）

## 最近执行：2026-09-06
- **dayIndex = 25**（today 2026-09-06 − startDate 2026-08-13 + 1）
- **模式**：review（dayIndex 25 > introDays 20，随机复习，非新学）
- **阶段扩展**：未触发（nextExpansionDay=31 ≠ 25；expansionsDone=0）
- **选中句**：[76, 77, 78, 79, 80]（review 池按 mastery 升序+reviewCount 降序；76–80 此前未复习过 mastery=0 最高优先）
  - s76 I'm really sorry to hear that; is there anything I can do to help?（慰问, long）
  - s77 Congratulations on your promotion!（祝贺, short）
  - s78 Thank you for inviting me.（致谢, short）
  - s79 You're welcome.（回应致谢, short）
  - s80 Excuse me, could you pass the salt?（餐桌礼仪, short）
- **增强内容**：5 句 enh 均 COMPLETE（base 100 句预置 fullIpa/variants/scenes/grammar/pron，非空）
- **音频**：s76–s80.mp3 均存在，无需新生成
- **写回 master.json**：5 句 lastReviewed=2026-09-06、reviewCount 0→1、mastery 0→1、introduced 保持 true（today_new=0）
- **生成页**：run_daily.py 覆盖旧 gen_future 预览页 → day2026-09-06.html ✓；gen_master_html.py 重生成 master.html（mtime 09:07）✓
- **服务**：端口 3279 回写服务 `curl /api/status` 返回 ok，已运行，无需重启
- **幂等守卫**：执行前校验 master.json 无 lastReviewed==2026-09-06 句 → 今日首次推送，安全执行（无重复 mastery 自增）
- **剩余扩展窗口**：nextExpansionDay=31（还有 6 天），到 day31 将 +50 句进入随机池

## 最近执行：2026-09-07
- **dayIndex = 26**（today 2026-09-07 − startDate 2026-08-13 + 1）
- **模式**：review（dayIndex 26 > introDays 20，随机复习，非新学）
- **阶段扩展**：未触发（nextExpansionDay=31 ≠ 26；expansionsDone=0）
- **选中句**：[81, 82, 83, 84, 85]（review 池按 mastery 升序+reviewCount 降序；81–85 此前未复习过 mastery=0 最高优先）
  - s81 This food is delicious!（日常/食物赞美, short）
  - s82 I'm on a diet.（日常/节食, short）
  - s83 I feel a bit under the weather.（日常/身体不适, short）
  - s84 Can you believe it?（日常/感叹, short）
  - s85 That sounds great!（日常/回应, short）
- **增强内容**：5 句 enh 均 COMPLETE（base 100 句预置 fullIpa/variants/scenes/grammar/pron，非空）
- **音频**：s81–s85.mp3 均存在，无需新生成
- **写回 master.json**：5 句 lastReviewed=2026-09-07、reviewCount 0→1、mastery 0→1、introduced 保持 true（today_new=0）
- **生成页**：run_daily.py 覆盖旧 gen_future 预览页（原 Aug 24）→ day2026-09-07.html（mtime 09:08:45）✓；gen_master_html.py 重生成 master.html（mtime 09:08:45）✓
- **服务**：端口 3279 回写服务 `curl /api/status` 返回 {"ok":true,"port":3279}，已运行无需重启
- **幂等守卫**：执行前校验 master.json 无 lastReviewed==2026-09-07 句 → 今日首次推送，安全执行（无重复 mastery 自增）

## 最近执行：2026-09-11
- **dayIndex = 30**（today 2026-09-11 − startDate 2026-08-13 + 1）
- **模式**：review（dayIndex 30 > introDays 20，随机复习，非新学）
- **阶段扩展**：未触发（nextExpansionDay=31 ≠ 30；expansionsDone=0，还差 1 天到扩展日）
- **选中句**：[4, 38, 70, 79, 80]（review 池按 mastery 升序+reviewCount 降序轮转）
  - s4 It's nice to meet you.（寒暄/初次见面, daily, mastery 3→3 rc 3）
  - s38 Do you accept credit cards?（购物/支付, travel, mastery 2 rc 2）
  - s70 It's raining cats and dogs.（闲聊/天气, daily, mastery 2 rc 2）
  - s79 You're welcome.（礼貌/回应, daily, mastery 2 rc 2）
  - s80 Excuse me, could you pass the salt?（餐桌/礼貌, daily, mastery 2 rc 2）
- **增强内容**：5 句 enh 均 COMPLETE（fullIpa/variants(3)/scenes(3)/grammar/pron 全非空）
- **音频**：s4/s38/s70/s79/s80.mp3 均存在，无需新生成
- **写回 master.json**：5 句 lastReviewed=2026-09-11、reviewCount 各 +1、mastery 各 +1（s4: 2→3，其余 1→2）、introduced 保持 true（today_new=0）
- **生成页**：run_daily.py → day2026-09-11.html（23.5KB，mtime 09:04）✓；gen_master_html.py 重生成 master.html（1000 句，mtime 09:04）✓
- **服务**：端口 3279 回写服务 `curl /api/status` 返回 {"ok":true,"port":3279}，已运行无需重启
- **幂等守卫**：执行前校验 master.json 无 lastReviewed==2026-09-11 句 + day2026-09-11.html 不存在 → 今日首次推送，安全执行（无重复 mastery 自增）
- **学习总结**：streak=13 天（09-11 回溯至 08-30 连续）；累计 distinct review 日 17；learned 100/1000；今日新学 0、复习 5
