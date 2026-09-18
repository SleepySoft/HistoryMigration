# HOW · 「编辑后不实时刷新」问题分析

> 这是旧版最著名的缺陷，也是 UniversalHistory Workspace 信号机制的立项依据。README:91 作者自述：「当前软件在编辑内容后可以不会实时更新，这点会在以后的版本改进」；0.4.0 计划含「支持内容改变后实时更新绘制，并在右键中增加刷新功能」（README:136）。

## 现象

无论通过菜单打开编辑器还是双击事件条编辑，Apply 保存成功后，时间轴上显示的仍是旧内容；需重新 Load File 才能看到修改。

## 三层原因（均有代码证据）

### 1. Thread 持有的是索引快照而非活数据

Load File / Load Index 时调用 `HistoryRecordIndexer.index_records()`，它通过 `HistoryRecord.to_index()` 为每条记录**新建一个 index 记录副本**（只含 uuid/since/until/source/abstract 摘要，`core.py:516-532`）。此后内存记录的修改不会反映到 Thread 的索引副本上。

根源在设计妥协：README:137 承认「初期的设计中，Thread 应该导入 Index 而非直接导入文件」——Index 本为分布式场景设计（见 [../why/03-data-philosophy.md](../why/03-data-philosophy.md)），本地浏览直接载 record 是后来的补丁。

### 2. 菜单打开的编辑器与时间轴零联动

`on_menu_record_editor`（`main.py:319-321`）不给 `HistoryEditorDialog` 传 agent → Apply 只走对话框默认 agent（写文件 + 弹框 + 刷新浏览器），**没有任何代码通知 TimeAxis 更新**；TimeAxis 也没有监听 History 数据变化的机制——`History` 类无信号/回调。

### 3. 双击路径的 repaint 也是徒劳

双击索引条弹出的编辑器在 Apply 后会 `self.repaint()`（`viewer_ex.py:1524-1534`：TimeAxis 作为 editor_agent，on_apply 里先调对话框保存再 close + repaint）——但 repaint 重绘的仍是**旧索引快照**，内容依旧不更新。

## 迁移对策（已实现）

UniversalHistory 的分层正是对此的结构性回应：

- `Workspace` 是内存唯一数据源，提供增加、更新、删除、查询和**信号通知**；
- UI 通过信号响应数据变更——编辑后时间轴立即刷新；
- `EventIndex` 是渲染用的轻量索引，由模型层派生而非独立快照副本。

验收核对：在 UniversalHistory 中新建/编辑/删除事件后，时间轴**必须**无刷新操作即时更新。
