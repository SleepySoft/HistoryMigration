# WHAT · 过滤器与索引

> HOW 细节见 [../how/06-filter.md](../how/06-filter.md)、[../how/07-index-pipeline.md](../how/07-index-pipeline.md)。

## 过滤器（History Filter Editor）

打开方式：View → History Filter Editor（快捷键 Ctrl+R，与 Record Editor 冲突）。

用户可定义一个过滤器，包含四个条件：

1. **Sources**：从哪些 .his 文件筛选（可多选）；
2. **Focus Label**：只看聚焦某要素的记录（event/people/location/organization，可留空）；
3. **Include Tags**：包含指定标签即入选（如 `people: 曹操`）；
4. **Exclude Tags**：包含指定标签即排除。

过滤器可保存为 `.hisfilter` 文件、重新加载。

**状态：半成品。**

- 核心过滤算法（`History.select_records`）完整且有测试固化行为；
- 但「把过滤结果显示到 Thread」的菜单项已被移除（注释标注 "Incomplete function, removed"），处理代码残留为死代码——**用户无法把过滤器应用到时间轴**；
- Check 按钮是空实现；include/exclude 输入非法格式时静默丢弃且无提示；
- `Load All Records` 载入内存后配合 filter 使用的设计路径（README:89）在 UI 上已不可到达。

## 索引（Index）

设计意图：把记录的时间+摘要（50 字符）+来源指针提取成轻量 index，供在线/窄带场景先展示概要、按需加载详情（README 概念 4）。FilterEditor 的 Generate Index 按钮可把过滤结果生成为 `.index` 文件。

**状态：断链不可用。**

- 索引生成脚本因参数签名不匹配无法运行；
- 生成的 `.index` 文件读不回来（回读函数缺失）；
- 界面上的 Load Index 菜单项已移除。

## 迁移对应

UniversalHistory 已实现 `FilterDialog`（按 source/focus/包含排除标签/时间范围筛选，结果挂到独立的 `__filter__` Thread）——覆盖了旧版「过滤器应用到时间轴」的未遂意图；index 概念演化为 `EventIndex` 渲染索引，不再有独立的 index 文件管线。
