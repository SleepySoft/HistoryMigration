# HOW · 过滤器

> 模块：`History/filter.py`（435 行）+ `core.py` 的 `History.select_records`（1078-1119 行）与 `LabelTag.filter/includes`（361-391 行）。WHAT（用户视角）见 [../what/03-filter-and-index.md](../what/03-filter-and-index.md)。

## 1. HistoricalFilter 数据结构（filter.py:21-101）

继承 `LabelTag`——**过滤器本身是一份 LabelTag 文档**，固定四个 label：

| label | 语义 |
| --- | --- |
| `sources` | 文件路径列表（覆盖式设置） |
| `focus_label` | 单值（存成单元素 list） |
| `include_tags` | 每个条目是一段 `label: tag1, tag2` 文本（二级结构） |
| `exclude_tags` | 同上 |

- 写入 include/exclude 前用 `LabelTagParser().parse()` **校验每条是合法 LabelTag 语法**（`__check_tags_str`，`:94-101`）；**校验失败静默 return False 不写入**——且 `ui_to_filter` 不检查返回值，非法输入导致整组条件被丢弃且无提示。
- 读取时 `get_include_tags()/get_exclude_tags()` 把存的 tag 字符串以 `'; '.join(...)` 重新拼接再解析为 `{label: [tags]}` 字典（`:85-92`）；解析失败返回 `{}`（静默吞错）。
- 序列化复用 `LabelTag.dump_text()`；文件扩展名 `.hisfilter`。
- `get_filter()/set_filter()`（`:129-133`）是 `pass` 空实现——未完成的公开 API。

## 2. 过滤语义（core.py）

### select_records（`:1078-1119`）——四步 AND，任一为 None/空即放行

| 参数 | 语义 |
| --- | --- |
| `_uuid: str or [str]` | 单值等值匹配，集合则 `in`（`:1084-1090`） |
| `sources: str or [str]` | 按 source 键过滤（`:1093-1099`） |
| `focus_label: str` | 记录 focus 精确相等（`:1102-1105`） |
| `include_label_tags: dict` + `include_all: bool` | 包含过滤 |
| `exclude_label_tags: dict` + `exclude_any: bool` | 排除过滤 |

### LabelTag.includes（`:361-391`）

- `all=True`（AND）：要求**每个 label 存在且每个期望 tag 都在记录的该 label 下**，任一不满足即 False；
- `all=False`（OR）：任一命中即 True。
- `filter()`：include 不满足 → False；exclude 命中 → False。

### 应用层固定用法

FilterEditor 与 Thread filter 死代码均使用 **`include_all=False, exclude_any=True`**（`filter.py:328-331`；`main.py:505-509`）：**命中任意一条包含条件即入选；命中任意一条排除条件即排除**。

### 行为基准（迁移可直接复用的验收用例）

`core.py:1239-1265` 的 `test_history_filter` 固化：`{'tags':['tag1']}`→1 条；`{'tags':['tag3']}`→3 条；`{'tags':['tag5','odd']}` include_all→3 条；`{'tags':['tag1','even']}` include_all=False→3 条；不存在的 tag→0；`{'author':['Sleepy']}`→4 条。

## 3. FilterEditor 界面（filter.py:106-334）

### 布局（`:140-169`）

竖排四组：

1. **Source Selector**：EasyQListSuite（QListWidget + Add/Remove 按钮）；
2. 并排 **Include Label Tags** / **Exclude Label Tags**（各一个 EasyQListSuite）；
3. **Focus Label**：可编辑 QComboBox，预置 `''、event、people、location、organization`；
4. 底部四按钮：**Save / Load / Check / Generate Index**。

### 行为

- **Source Add**（`:187-196`）：文件多选框（起始 depot 根，`*.his`）；选中绝对路径若以 depot 根开头**裁成相对路径**保存；去重（set）刷新。Remove 删选中项（多选）。
- **Include/Exclude Add**（`:204-230`）：`QInputDialog.getText` 单行输入，原文直接追加（**此处不做语法校验**，校验延迟到保存时且失败静默丢弃）。
- **Save**（`:234-246`）：保存对话框 `*.hisfilter`；`dump_text()` 写盘——**未指定 encoding**，Windows 下中文按 GBK 写出而读端用 utf-8，潜在乱码（`:256` 用 `'rt'` 默认编码读）。
- **Load**（`:248-267`）：打开 `.hisfilter` → LabelTagParser 全文解析 → **失败仅 print 到 stdout**，无 UI 反馈；成功则 attach 并回填四个控件（include/exclude 字典转回 `label: tag1, tag2` 文本）。
- **Check**：`__on_btn_click_chk` 是 `pass`（`:269-270`）——**未完成**。
- **Generate Index**（`:272-285`）：保存对话框 `*.index`（文件类型描述误写为 "Filter Files (*.index)"）→ `load_filter_records()` 加载+过滤 → `HistoryRecordIndexer` 生成并写盘 → 弹 "Generate index for N records Done."。**记录数为 0 也照样生成空文件并提示成功**。

### 数据流（`:308-331`）

`load_source_records()` 按 sources 逐个加载（绝对路径走 `from_file`，相对路径拼 depot 根）；装入 `History`（无注入则用临时实例）；`select_records` 四层过滤。

## 4. filter → Thread 的断裂路径

- Thread 右键菜单的 `Use Filter`/`Load Index` 已被注释移除（"Incomplete function, removed"，`main.py:419-421`），处理分支残留为死代码（`:466-472, 497-510`）：死代码中 `opt_open_filter` 会把过滤结果转成 index 列表以 `{'filter': indexes}` 挂到 Thread，注释 `# TODO: Now it means to filter all records`。
- 入口：View → History Filter Editor（`main.py:323-326`）以 `WrapperQDialog` 包一层打开，不注入 history。

## 5. 迁移对应

UniversalHistory 的 `FilterDialog` 已实现按 source/focus/包含排除标签/时间范围筛选，结果挂到独立 `__filter__` Thread——覆盖旧版未遂意图。过滤语义应以 `test_history_filter` 为对齐基准。
