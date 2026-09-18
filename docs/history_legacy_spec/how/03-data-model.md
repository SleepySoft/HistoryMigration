# HOW · 数据模型

> 模块：`History/core.py` 的 `LabelTag` / `HistoryRecord` 部分。WHY 见 [../why/03-data-philosophy.md](../why/03-data-philosophy.md)。

## 1. LabelTag 容器（core.py:302-359）

`label → tags` 的字典容器，是记录、过滤器、索引的共同基类。

- `set_label_tags(label, tags)`：覆盖式；所有 tag 先 `strip()`。
- `add_tags`（`:335-342`）：用 `list_unique`（set 实现）去重——**会打乱 tag 顺序**。
- `attach`：合并另一 LabelTag；下方注释掉的代码（`:315-319`）才是重复 label 的正确合并实现。
- `dump_text`（`:348-359`）：`label: tag1, tag2\n` 格式；tags 为空的 label 跳过。
- `filter(include, include_all, exclude, exclude_any)` / `includes`（`:361-391`）：过滤语义见 [06-filter.md](06-filter.md)。边界怪癖：`include_all=True` 时若某 label 的期望 tags 为空列表，结果取决于前面 label 是否命中——边界语义脆弱。
- **bug**：`label_tags_list_to_dict`（`:294`）`if label not in label_tags_list` 应为 `label_tags_dict`——str 与 tuple 列表比较恒为 True → 同一记录内**重复 label 只保留最后一次的 tags**（前一个的 tags 丢失）。

## 2. HistoryRecord 字段（core.py:407-710）

继承 LabelTag。实例字段（`__init__`，`:420-426`）：

| 字段 | 默认 | 语义 |
| --- | --- | --- |
| `__uuid` | `uuid.uuid4()` | 唯一 ID；dump 时为空自动重新生成（`:583-584`） |
| `__since` / `__until` | 0 | 时间范围 TICK；由 `time` label 解析而来，或 `since`/`until` label 直写 |
| `__focus_label` | `''` | 聚焦的五要素之一（或 `index`）；决定记录在哪一行关闭、dump 时哪个 label 压轴 |
| `__record_source` | 构造参数 | 来源（文件路径/URL）；depot 内文件归一化为相对路径（`normalize_source`，`:890-892`） |

## 3. label 语义（`set_label_tags`，core.py:494-514）

- **五要素**：`time`、`location`、`people`、`organization`、`event`（docstring `:411`）。
- **可选公共 label**：`title`、`brief`、`uuid`、`source`、`author`、`tags`。
- **拦截型 label**（不进入 label_tags 字典）：
  - `uuid` → 直接设 uuid（取 `tags[0]`；空 tags 会 IndexError，`:500`）；
  - `time` → `__try_parse_time_tags`（`:629-645`）：所有 tag `,` 拼接后交 `HistoryTime.time_text_to_ticks`，取 min/max 为 since/until；解析不出则 since=until=**0.0**（float，与 TICK=int 类型不一致）；
  - `since`/`until` → 调 `HistoryTime.decimal_year_to_tick`——**该函数不存在，是断链 bug**（见 [07-index-pipeline.md](07-index-pipeline.md)）；
  - `source` → 设 record_source。
- 其余任意 label 走 `LabelTag.add_tags`（保留、去重、不保序）。
- 便捷访问器：`time()` 正常；**`people()/location()/organization()` 调用不存在的 `self.tags()`——必抛 AttributeError**（`:464-470`），全仓库无人调用所以从未暴露。`title()/brief()/event()` 标注返回 str 实际返回 list，调用方都套 `LabelTagParser.tags_to_text` 使用。

## 4. Index 投影（`to_index()`/`index_for()`，core.py:516-532）

生成 focus=`index` 的精简记录：复制 uuid/since/until/source；abstract 取 title→brief→event 首个非空者，strip 后**截断 50 字符**存为 `abstract` label。

## 5. dump_record 写出规则（core.py:551-601）

1. **label 排序**（`__get_sorted_labels`，`:647-657`）：优先集 {time, people, location, organization} 按字母序（实际输出顺序 location, organization, people, time）→ 其余 label 字母序 → 尾部集 {brief, event, title} 字母序。
2. 移除 uuid 单独输出；focus label 强制移到末尾。
3. 输出 `[START]: focus`；focus 为空默认 `event`（`:567-568`）。
4. index 记录额外输出 `since:`/`until:`（`tick_to_years(...)[0]` 的整数年）和 `source:`（`:588-591`）。
5. 公共 label 按序输出（tags 为空跳过）。
6. focus label 缺失/为空时补写 `focus: end`（`:598-599`）。
7. `compact=True` 用 `'; '` 代替换行（单行记录，供 index 文件）。

## 6. 其他行为

- `period_adapt(since, until)`（`:534-535`）：区间重叠判定 `self.since <= until and self.until >= since`（闭区间，边界相接算重叠）。
- `duplicate_from`（`:456`）：`self.__label_tags = record.__label_tags` 是**引用复制非深拷贝**——改副本会改原件（bug）。
- `__str__`（`:694-710`）：普通记录与 index 记录两种打印格式。
