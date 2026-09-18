# WHY · 数据格式与模型的设计哲学

> 出处：`History/depot/example/example.his:21-30`（event 块自述）、`README.md` 设计节（75-84 行）、`core.py:407-418` docstring。

## 为什么是纯文本自定义格式

`example.his` 第 21-30 行的示例记录本身就是一份格式宣言，自述动机：

- 数据格式应**跨平台**，人人可写自己的编辑器/查看器；
- 数据**可拆分、可合并**，支持 tags 索引；
- 文本格式可存 GitHub，便于 **review / merge**（README:139-141 的开放协作节呼应：fork 后提交 pull request，记录里加 `author` label 署名）。

即：`.his` 不只是存储格式，而是**协作媒介**——纯文本 + Git 就是它的协同编辑方案。

## 为什么用 LabelTag 模型

所有数据（记录、过滤器、索引）统一为 `label: tag1, tag2, ...` 的 LabelTag 文档。README Dev Note 记录了「统一 LabelTag、Index 及 event」是刻意完成的重构（`README.md:160`）。好处：

- 一个解析器通吃三类文档（.his / .hisfilter / .index）；
- label 开放扩展——任意自定义 label 都会被保留，格式不被 schema 锁死；
- 人类可直接读写，无需工具。

## 为什么是五要素 + focus_label

README 需求 5：「能识别事件描述中的时间、地点、人物、组织、事件，并以此为索引进行关联查找和浏览」。

`HistoryRecord` docstring（`core.py:407-418`）规定五要素 label：`time/location/people/organization/event`。`focus_label` 表示**这条记录的观察视角**——`[START]: people` 意味着这条记录聚焦人物，「以人观史」。技术上 focus label 还承担语法角色：它是每条记录的**结束标记**（记录在遇到等于 focus 的 label 行时关闭），因此 dump 时强制压轴输出。

## 为什么 Index 与 Record 分离

README 设计节概念 4（82-83 行）：

> 软件的设计上考虑到在线和分布式的使用场景。由于空间和带宽的限制，展现给客户的不可能是完整的内容。因此软件支持将记录的时间和摘要提取出来作为索引，并指向真正内容。当用户需要查看详情的时候，主要内容才会被加载。

Index = uuid + since/until + source + 50 字符摘要。本地浏览直接 Load File 是后期的妥协（README:137 承认「初期的设计中，Thread 应该导入 Index 而非直接导入文件」）——**这个妥协正是「编辑后视图不刷新」问题的根源**（见 [how/13-stale-ui-analysis.md](../how/13-stale-ui-analysis.md)）。

同时 README:83 指出 filter 与 index 的分工：「filter 只能应用于已载入的记录……index 包含的信息过少，网页及分布场景我们难以使用 filter」。

## 为什么是 depot 目录组织

README 设计节概念 4（depot）：「为了管理方便，我们将所有数据放置在软件目录下的 depot 目录中，depot 目录中的每个文件夹称为一个 depot。记录文件可以按语言或内容分类放置在不同的 depot 中。」

depot root 硬编码在代码文件旁（`core.py:895-898`）——**数据与程序同居**，是「个人笔记工具」而非「分发型软件」的定位体现。迁移分析已将「保存目标文件边界不清」列为要改进的痛点（根目录 `migration_design.txt:11`）。

## 对迁移的启示

- 纯文本 + Git 协作的理念在 UniversalHistory 中保留：`.his` 作为「人类/AI 可读交换格式」继续存在，由 Adapter 层隔离（`migration_analysis.md` 决策表）。
- 五要素 + focus 的语义完整保留进 `Event.labels`。
- Index 的「瘦身投影」思想演化为 `EventIndex`；但旧版 index 的读写断链缺陷（见 [how/07-index-pipeline.md](../how/07-index-pipeline.md)）不应复刻。
- 「数据与程序同居」被 Workspace/Source 分层取代：source 可以是任意路径的文件，不再锁定 depot。
