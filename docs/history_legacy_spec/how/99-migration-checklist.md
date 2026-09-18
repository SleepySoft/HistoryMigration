# HOW 总结二 · 迁移验收要点速查

> 以下用户可感知行为应逐项在 UniversalHistory 中核对（测试或人工）。每项给出旧版出处与核对方法。
> 缺陷项的处置决策见 [98-known-defects.md](98-known-defects.md)；设计意图是否仍成立见 [../why/99-summary.md](../why/99-summary.md)。

## 1. 时间解析管线

旧版出处：[02-natural-language-time-parsing.md](02-natural-language-time-parsing.md)。

- [ ] 管线顺序：删「约」→ 预替换（元月/正月/世纪/至今）→ 方括号保护 → 分隔符统一 → 4 种标准格式 → BCE 子串判定（bc/bce/公元前/前/距今/史前）→ 中文转阿拉伯 → 正则取首个年/月/日 → 钳制（年 0→1、月 [1,12]、日 [1,当月天数]）→ 多段取 min/max；
- [ ] 容错语义保留（无关文字忽略、无「年」数字段当公元年份）；
- [ ] 中文数字：大写/小写/异体、十百千万亿兆两级单位、离散数字串；
- [ ] TICK→JDN 桥接：无 0 年偏移 `jdn_year = history_year + 1 if history_year < 0 else history_year`；只在载入阶段发生，不要求双向精确（`migration_analysis.md` §8.1）。

## 2. 刻度与缩放

旧版出处：[08-timeline-rendering.md](08-timeline-rendering.md) §4-5。

- [ ] 主刻度最小 50px 密度阈值；
- [ ] 年为刻度时副刻度为月；月→周（7 天）；周→日；日→时；
- [ ] 副刻度距下一主刻度不足半步即省略（月末/年末不重叠）；
- [ ] 公元前刻度 `BC xxxx` 前缀；
- [ ] Ctrl+滚轮缩放锚定鼠标指向的时间点不动；
- [ ] 普通滚轮每格 1/4 主格；
- [ ] 刻度吸附：鼠标在刻度像素上取精确时间。

## 3. 布局

旧版出处：[09-thread-track-layout.md](09-thread-track-layout.md)。

- [ ] 长按续时长降序参与分配；
- [ ] 全量重排保证滚动稳定（同一事件永远在同一列）；
- [ ] Track 数 = round(Thread宽 / 最小轨宽)，最少 1 条；
- [ ] 空间不足时 fallback（旧版末轨兜底；新版按 §8.5 策略）；
- [ ] 重叠判定严格相交、首尾相接不算冲突；
- [ ] 单点事件虚拟像素区间参与统一分配（新设计，**有意放宽**旧版「固定首轨」）。

## 4. 交互

旧版出处：[10-interactions.md](10-interactions.md)。

- [ ] 右键菜单：空白处 Add Thread（按点击侧）；Thread 上 Load File / New Record / Set Track Width / Add Thread On Left/Right / Remove This Thread；
- [ ] 双击事件条打开编辑器；悬停十字线 + 提示框；
- [ ] 持续事件提示「标题(第N年/共M年) : [起 - 止]」，N 随鼠标位置变化；
- [ ] 编辑后时间轴**即时刷新**（旧版缺陷，新版必须修复，见 [13-stale-ui-analysis.md](13-stale-ui-analysis.md)）；
- [ ] 横纵切换可用（旧版坏损，新版 Ctrl+T）。

## 5. 编辑器

旧版出处：[11-editor.md](11-editor.md)。

- [ ] Time 永远必填；focus 必填校验（focus=event 时 title/brief/event 至少一个）；
- [ ] Lock 语义：新建记录时锁定字段保值；
- [ ] 「保存到哪个文件」在保存时才问（新文件路径）；
- [ ] 记录列表按时间排序；
- [ ] Apply 不丢失 UI 未暴露的字段（author、自定义 tags——旧版缺陷）；
- [ ] 删除需确认（旧版无确认直接落盘——缺陷）。

## 6. 数据兼容（红线）

旧版出处：[04-his-file-format.md](04-his-file-format.md)。

- [ ] `depot/example/example.his` 6 条记录解析结果一致（时间、标签、source、focus）；
- [ ] `depot/China_CN/`、`depot/World_CN/` 真实数据可加载（中文文件名、自然语言时间段、`event: end` 收尾）；
- [ ] 写回 `.his` 保持可加载，不静默改写用户数据（`"""` 包裹、focus 压轴、`end` 补齐规则一致）；
- [ ] 过滤器语义对齐 `core.py:1239-1265` 的 `test_history_filter` 基准。

## 7. 现成验收用例

- `core.py:1190-1265`：`test_token_parser_case_normal`（词法）、`test_history_basic`（记录读写）、`test_history_filter`（过滤语义）——注意 `test_generate_index` 本身是坏的，不要纳入基准。
- `to_arab.py:125-190`：中文数字转换测试。
- UniversalHistory 侧：61 个 unittest（2026-09-18 验证全过，见 `docs/PROJECT_STATUS.md`）。
