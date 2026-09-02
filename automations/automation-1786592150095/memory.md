# 自动化执行记录：每日英语口语推送（词力词汇教练）

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
