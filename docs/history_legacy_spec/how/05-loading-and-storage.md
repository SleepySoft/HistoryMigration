# HOW · 加载、存储与内存库

> 模块：`History/core.py` 的 `HistoryRecordLoader`（715-918 行）与 `History`（961-1183 行）。

## 1. HistoryRecordLoader

错误码为字符串常量（`:717-724`）：`E_SUCCESS / E_FAIL / E_SOURCE_NOT_EXISTS / E_SOURCE_INVALID / E_SOURCE_READONLY / E_SOURCE_NOT_SUPPORT`（`E_SOURCE_READONLY` 定义了但从未使用）。哨兵 `INVALID_SOURCE='!@#$%&*?'`（`:726`）。

### 保存

- `to_source(source, records)`（`:730-738`）：source==INVALID_SOURCE → 拒绝；Web URL → 打印 "not support yet" 返回 `E_SOURCE_NOT_SUPPORT`；否则写本地。
- `to_local_source`（`:747-766`）：records 为 falsy → `E_SOURCE_INVALID`；单条记录自动包成列表；**utf-8 文本模式顺序拼接**各记录 `dump_record()`（记录间无显式分隔，靠 dump 自带换行）；相对路径先拼到 depot root；异常打印后返回 `E_FAIL`。**整文件覆写**。

### 加载

- `from_source`（`:794-799`）：`is_web_url`（startswith `http`/`ftp`，`:876-877`）分流 Web/本地。
- `from_web`（`:809-820`）：`requests.get` → utf-8 解码 → `from_text`，source 记为 URL；**无超时、无状态码检查**，异常仅打印返回 `{}`。
- `from_file`（`:822-834`）：打印 `| => Load record: <file>`，utf-8 读入；**异常静默返回 `{}`——加载失败与空文件不可区分**。
- `from_directory`（`:782-792`）：`enumerate_local_path` 递归 `os.walk`；suffix=None 时**不加后缀过滤，目录下所有文件都当 .his 解析**（`:907-918`）。
- `from_local_depot(depot)`（`:770-780`）：depot 是 depot root 下的**子目录名**，拼成 `<History>/depot/<depot>` 后按目录加载。
- `from_files`（`:801-807`）：逐文件加载后 dict update——同名 source 键后者覆盖前者。

### 路径工具

- `is_absolute_path` 同时认 Windows（ntpath）和 POSIX（posixpath）绝对路径（`:880-881`）。
- `get_local_depot_root` = `dirname(abspath(core.py))/depot`——**depot 位置硬编码在代码文件旁**（`:895-898`）。
- `normalize_source` 对以 depot root 开头的路径掐头（`len(root)+1`，假定单字符分隔符，`:890-892`）。
- main.py 菜单 Load Files 的前缀剥离 `f[len(depot_root):]`（`main.py:307`）会留下前导路径分隔符的"相对"路径，依赖 `posixpath.isabs` 的 OR 判断兜底——跨平台行为微妙。

## 2. depot 目录组织约定

- depot root 硬编码为 `History/depot/`；**每个子目录是一个 depot**（README 设计节：可按语言或内容分类）。
- 当前内容：`example/`（1 个文件）、`China_CN/`（8 个 .his）、`World_CN/`（6 个 .his），文件名即主题（如 `三国时期事件.his`），支持中文文件名。
- depot 枚举在 UI 层：`editor.py:728-731` 列 depot root 下所有子目录；depot 内记录文件递归 walk（`:733-738`）。
- 相对 source 一律相对 depot root 解析。

## 3. History 内存库（core.py:961-1183）

`{source: [records]}` 字典（`:970`）。**无信号/回调机制**——数据变更不通知任何监听者。

### 管理操作

- `remove_record(uuid)`（`:979-981`）：**跨所有 source** 全表扫描 pop 所有同 uuid 记录并打印。
- `upsert_records(source, records)`（`:987-1001`）：先按 uuid 全局删除旧记录（**替换式更新**），再追加到 source 列表末尾；注释说明顺序无所谓，布局前会按时间排序（`:994-995`）。
- `change_source`（`:1003-1007`）：仅当旧 source 存在且新 source 不存在时改名；`remove_source`、`reset_history` 直删。
- `load_source / load_depot / load_path`（`:1123-1148`）：加载后 `update` 进表，返回新增部分；**同 source 重复加载整体覆盖**。
- `save_source`（`:1152-1155`）：source 不在表中 → `E_SOURCE_NOT_EXISTS`。

### 高阶与查询

- `map(func)`（`:1018-1027`）；`pop(selector)`（`:1029-1038`）边遍历边 pop；`filter(func)`（`:1040-1058`）遇 None 记录打印警告跳过。
- `get_record_by_uuid`（`:1062-1065`，取首个匹配）、`get_record_by_source`、`get_records_by_sources`、`get_indices_by_sources`（现场生成索引）。
- 静态：`sort_records` 按 since 升序（`:1167-1169`）；`unique_records` 按 uuid 去重（后者覆盖前者，返回 dict_values 而非 list，`:1171-1173`）。
- `print_indexes` 是空实现（`:1162-1163`）。

### select_records（`:1078-1119`）

见 [06-filter.md](06-filter.md)。
