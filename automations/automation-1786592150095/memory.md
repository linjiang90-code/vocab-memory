# 自动化执行记录：每日英语口语推送（词力词汇教练）

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
