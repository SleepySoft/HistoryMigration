# HOW · 索引管线（当前断链）

> 模块：`History/core.py` 的 `HistoryRecordIndexer`（923-956 行）与 `History/indexer.py`（41 行，命令行脚本）。设计意图（WHY）见 [../why/03-data-philosophy.md](../why/03-data-philosophy.md) 的 Index 节。

## 1. 索引内容

`to_index()`（`core.py:516-532`）生成 focus=`index` 的精简记录：

- 复制 uuid / since / until / source；
- abstract 取 title→brief→event 首个非空者，strip 后**截断 50 字符**存为 `abstract` label。

dump 时 index 记录额外写出 `since:`/`until:`（`HistoryTime.tick_to_years(...)[0]` 的**整数年份**）和 `source:`（`:588-591`）；`dump_to_file` 每条以 compact 模式（`; ` 分隔）单行输出 + `\n`（`:946-952`）。

## 2. API

- `index_path(directory)`（`:930-934`）：整目录加载后逐 source 生成索引，返回 `{source: [index_record]}`。
- `index_records(records)`（`:936-944`）：list/tuple/set → 索引 list；dict → 逐值索引；其他类型 → 空 list。
- `dump_to_file(indexes, file)`（`:946-952`）：utf-8 写文件。
- `load_from_file`（`:954-956`）：直接复用 `HistoryRecordLoader.from_file`。

## 3. 断链一：生成管线无法运行

`indexer.py`（全文 41 行）：无参数默认索引 depot `China_CN`，有参数逐目录索引；然后调 `indexer.dump_to_file('depot/history.index')`——**两个 bug**：

1. `index_path` 的返回值被丢弃（indexer 无状态，`__init__` 为空，`:927-928`）；
2. `dump_to_file` 签名是 `(indexes, file)` 两个参数，只传一个 → TypeError。

`core.py` 自带测试 `test_generate_index`（`:1270-1274`）同样坏（把字符串路径传给 indexes 形参）；`core.py` 的 `main()`（`:1287-1297`）跑到该测试必崩。

## 4. 断链二：index 文件写出来读不回

- 写出端：`since:`/`until:` 是**整数年份**（`tick_to_years(...)[0]`）；
- 回读端：`HistoryRecord.set_label_tags` 对 `since`/`until` label 调 `HistoryTime.decimal_year_to_tick`（`:505-510`）——**该函数在新 HistoryTime.py 中不存在**（旧 `recycled/history_time.py` 时代遗留，新模块未实现）→ 加载任何含 since/until 标签的 .index 文件必抛 AttributeError。

两端本就约定"十进制年份"，但新时间系统没有对应转换函数。

## 5. 迁移决策

**不必修复旧管线，也不复刻断链。** UniversalHistory 的 `EventIndex` 直接持有 `JDNTimestamp` 于内存中，作为渲染/网络传输的轻量投影；不存在独立的 .index 文件格式。若未来需要网络索引传输，应在 Adapter 层定义新格式，并以「写出-回读 round-trip」为强制验收。
