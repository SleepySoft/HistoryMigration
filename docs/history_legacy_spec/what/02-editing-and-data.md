# WHAT · 事件编辑与数据组织

> HOW 细节见 [../how/11-editor.md](../how/11-editor.md)、[../how/05-loading-and-storage.md](../how/05-loading-and-storage.md)。

## 事件编辑（Historical Record Editor）

打开方式：View → Historical Record Editor（Ctrl+R）、双击时间轴上的事件条、或 Thread 右键 → New Record。

界面左侧是 depot/文件浏览器（选择分类目录和 .his 文件，可重命名文件），右侧是记录编辑器：

- **五要素录入**：Time（时间）、Location（地点）、Participant（人物）、Organization（组织）、Event（事件正文）五行字段，外加标题（Title）、简介（Brief）、正文（Description）。
- **时间输入**：支持自然语言——「公元前1000年1月1日 - 公元2000年10月30日」「前12世纪」「184年2月-184年8月」「至今」均可识别；输入时悬浮提示实时显示解析结果供核对；Calendar 按钮弹日历选择器（公元前时间不可用，会回退为当前时间）。
- **倾向（focus）**：五选一单选（默认 Event），表示这条记录「以什么为视角」；所选倾向对应的字段为必填（Time 永远必填）。
- **锁定（Lock）**：勾选后新建下一条记录时该字段保留原值——便于连续录入同一地点/人物的系列事件。
- **新建**：`New Event` 在当前文件中新建；`New File` 新建到新文件（保存时才弹出另存对话框询问位置）。
- **保存**：`Apply`（Ctrl+S）写回文件并弹结果提示；`Save and New` 保存后立即开始下一条。
- **删除**：`Del Event` 删除当前记录——**无确认且立即写盘**。
- **浏览记录**：下拉框按时间顺序列出当前文件的所有记录（`[年份] uuid` 格式）。

已知使用缺陷（HOW 中有完整清单）：

- 切换记录、新建、取消时**不提示未保存修改**，直接丢弃；
- Apply 后 UI 上未暴露的字段（author、自定义 tags）会丢失；
- 倾向选择不会被旧记录还原；
- 编辑后**时间轴不会实时刷新**（需重新载入）。

## 数据组织（depot）

- 所有数据放在程序目录下的 `depot/` 中，每个子目录是一个分类（如 `China_CN/`、`World_CN/`），称为一个 depot。
- 数据文件是纯文本 `.his`，可用任何编辑器打开，可放 Git 管理。
- **载入**：File → Load Files（Ctrl+F）多选文件载入内存；File → Load All Records（Ctrl+A）加载全部 depot（数据多时界面长时间不响应）；File → Load Depot（Ctrl+D）是空实现。载入内存后需在 Thread 上右键 Load File 才会显示到时间轴。
