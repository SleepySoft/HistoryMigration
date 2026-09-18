# History → UniversalHistory 迁移分析与技术方案

> 基于 `migration_design.txt` 及对两个项目的代码阅读、测试运行结果整理。
> 当前阶段：**只分析、做记录、定方案，不改动业务代码。**

---

## 一、现状盘点

### 1.1 History（原项目）

| 维度 | 现状 |
|------|------|
| **时间模型** | 自定义 `TICK`（秒），以公元 1-01-01 00:00:00 为 0 点；公元前没有 0 年，-1 年 = 1 BC。实现 `Utility/HistoryTime.py`，支持自然语言解析、闰年、BC/AD 转换。 |
| **数据模型** | `HistoryRecord`（`core.py`），基于 `LabelTag`：key: tag1, tag2, ...；五要素标签为 `time/location/people/organization/event`，另有 `title/brief/uuid/source/author/tags`。 |
| **文件格式** | 自定义 `.his` 文本（`[START]: focus_label` 开头，标签行结尾），示例见 `depot/example/example.his`。 |
| **存储组织** | `depot/<category>/*.his`，由 `HistoryRecordLoader` 按文件/目录/Web URL 加载。 |
| **索引** | `HistoryRecordIndexer` 把完整记录提取为轻量 `index`（uuid/abstract/since/until/source）。 |
| **过滤** | `filter.py` 提供 `HistoricalFilter`，但 UI 完成度低，核心 `History.select_records()` 已实现按 uuid/source/focus_label/标签包含排除过滤。 |
| **渲染** | `viewer_ex.py` 的 `TimeAxis` + `HistoryIndexTrack`：支持横/竖向、无限缩放、多 Thread、Track 自动布局、单点/持续事件不同绘制、实时提示。 |
| **编辑** | `editor.py` 的 `HistoryRecordEditor`：文件/事件新建、时间/地点/人物/组织/事件编辑、锁定字段。 |
| **依赖** | `requests`、`pyqt5==5.15`、`PyInstaller`。 |
| **当前问题** | 保存目标文件边界不清；.his 自定义格式扩展性差；filter 几乎只有雏形；编辑后不能实时刷新 UI。 |

### 1.2 UniversalHistory（目标项目）

| 维度 | 现状 |
|------|------|
| **时间模型** | `JDNTimestamp`：微秒级定点整数，基于 JDN 0（-4713-11-24 12:00），采用天文纪年（0 = 1 BC），外推格里高利历，无浮点误差。 |
| **桥接层** | `PythonTimeBridge`（datetime/timestamp）、`LunarDateBridge`（农历/干支/生肖）。 |
| **刻度引擎** | `TickStepper`：从 1 天到 10 亿年的标准层级，按像素密度选择主/副刻度，对齐到真实历日。 |
| **坐标系** | `UniversalTimeAxis` / `main.py` 的 `RealTimeAxis`：统一逻辑坐标系 + QTransform 旋转，已实现横竖向切换、拖拽、缩放。 |
| **数据模型** | **暂无**。目前只是时间轴绘制 Demo，没有 Event、Record、Filter、Index、Editor。 |
| **测试** | `test_history_time_basic.py`、`test_comprehensive.py` 共 19 个用例全部通过（已验证）。 |
| **依赖** | 仅 `lunar_python`；演示界面需要 `PyQt5`（但 `requirements.txt` 未列出 PyQt5）。 |

### 1.3 已验证的运行结果

- **UniversalHistory 核心测试**：`python -m unittest test_history_time_basic test_comprehensive` → 19/19 通过。
- **History `core.py` 加载测试**：可正常解析 `example.his`，输出 6 条记录；`test_generate_index()` 因 `dump_to_file()` 签名问题报错（`indexes` 参数缺失），属于小 bug，不影响迁移评估。
- **PyQt5 安装问题**：在 Python 3.14 上 `pyqt5==5.15` 无法从二进制 wheel 安装（需要 qmake），原 `History` 的 GUI 在新 Python 环境已不易直接跑通。迁移时建议升级到 `PyQt6` 或改用 Web 前端。

---

## 二、目标映射：保留、改进、新增

| 来源需求 | 对应策略 |
|----------|----------|
| 无限时间轴 + 多 Thread 对照 | 用 `JDNTimestamp` 替换 `TICK`；保留 `HistoryIndexTrack` 布局思想；用 `UniversalTimeAxis` 的统一坐标变换替换原有横竖两套绘制。 |
| 横/竖向无缝切换 | 直接复用 `UniversalTimeAxis.get_coordinate_transform()` + `map_to_logical()` 方案。 |
| 单点/持续事件不同处理 | 保留 `HistoryIndexBar` 的 event（箭头/菱形）与 period（矩形）绘制及 track 分配策略。 |
| 灵活时间输入（含自然语言） | 保留 `HistoryTime.time_text_to_ticks()` 的解析逻辑，但输出改为 `JDNTimestamp`；最终提供“文本 → JDNTimestamp”适配器。 |
| 五要素提取/联查 | 保留 `LabelTag` 数据模型与 `select_records()` 过滤逻辑；为 Agent 提供结构化接口。 |
| 文件边界不清 | 引入 **Workspace / Session / Source** 三层概念：Workspace 是用户当前打开的集合，Session 是内存视图，Source 是具体文件/URL；新建记录时默认进入当前 Source，不再每次询问。 |
| 自定义格式 → 兼容层 | 保留 `.his` 作为“人类/AI 可读交换格式”；新增 **Adapter 层** `HisFileAdapter`，把 `.his` 加载为新的 `Event` 对象；未来可扩展 `JsonAdapter`、`HttpAdapter`。 |
| 过滤器 | 完善 `Filter` 引擎：时间范围 + 标签包含/排除 + focus_label + source；提供程序化 API 与 UI。 |
| AI/Agent 录入友好 | 定义 LLM 友好的 schema 与 prompt；提供 `AgentService`（创建/更新事件、查询、生成摘要），并通过事件总线通知 UI 刷新。 |
| 网页版 | 采用 **后端（Python/FastAPI）+ 前端（Web/Canvas）** 架构；后端复用核心模型与过滤，前端复用坐标/刻度思想。 |
| 权限管理 / 协同编辑 | 作为远期预留，先在接口上预留 `owner`、`visibility`、`branch`/`proposal` 字段，不实现。 |

---

## 三、核心差异与风险

### 3.1 时间表示转换

| 项目 | History | UniversalHistory |
|------|---------|------------------|
| 单位 | 秒（`TICK`） | 微秒（`int`） |
| 纪元 | 公元 1-01-01 00:00:00 = 0 | JDN 0 = -4713-11-24 12:00 = 0 |
| 公元前 | 无 0 年，-1 = 1 BC | 天文年，0 = 1 BC |
| 精度 | 秒 | 微秒 |

**转换方式**：不要直接做秒 → 微秒的线性偏移，因为纪元不同。应：

1. 用 `HistoryTime.tick_to_date_time_data(tick)` 得到 `(year, month, day, h, m, s)`；
2. 年份映射：`jdn_year = history_year + 1 if history_year < 0 else history_year`；
3. 调用 `JDNTimestamp.from_ymd_hms(jdn_year, month, day, h, m, s)`。

反向转换同理。由于 History 只有秒精度，转换后微秒补 0。

**风险点**：
- History 的 BCE 月日是其自定义推算（“公元前一日为公元前1年12月31日”），与 Proleptic Gregorian 在远古时期可能产生几天差异；对历史笔记场景影响小，但应在测试中验证常见年份。
- 没有 year 0 的跳跃需要显式处理，否则 1 BC ↔ 1 AD 会差 1 年。

### 3.2 数据模型差异

- History：`HistoryRecord` 同时承担“完整记录”和“索引”角色，`since/until` 在设置 `time` 标签时由文本解析得到。
- UniversalHistory：建议拆分：
  - `Event`：uuid、since/until（JDNTimestamp）、source、labels（dict[str, list[str]]）、content（title/brief/event）。
  - `EventIndex`：uuid、since/until、abstract、source（用于网络/大列表）。
  - `Workspace`：source → [Event] 的内存表，提供 upsert/remove/select。

### 3.3 渲染层差异

- History 的 `viewer_ex.py` 自己管理坐标映射、刻度、布局，代码量大且与 `TICK` 强绑定。
- UniversalHistory 已经抽象出 `AxisMetrics` 思想的前身（逻辑坐标系 + QTransform），并提供 `TickStepper` 计算刻度。
- **建议**：把 `HistoryIndexTrack` 的 layout 算法移植到基于 `UniversalTimeAxis` 的新 viewer 中，但把“布局计算”与“绘制”进一步解耦，便于 Web 前端复用。

### 3.4 依赖与运行环境

- Python 3.14 下 PyQt5 老旧版本无法安装；迁移时 GUI 层应考虑：
  - 桌面端：升级到 `PyQt6`（官方对 Python 3.14 支持更好）；
  - 或桌面端用 Web 视图（PyQt WebEngine / PySide）承载同一套前端。
- `lunar_python` 在 Python 3.14 下可正常安装并运行。

---

## 四、推荐架构

```text
UniversalHistory/                 # 以 UniversalHistory 为基座扩展
├── core/
│   ├── __init__.py
│   ├── jdn_timestamp.py          # 现有 JDNTimestamp（重命名/整理）
│   ├── tick_stepper.py           # 现有 TickStepper
│   └── event.py                  # Event / EventIndex / Workspace
├── time/
│   ├── __init__.py
│   ├── bridges.py                # PythonTimeBridge + LunarDateBridge
│   └── history_time_adapter.py   # History TICK ↔ JDNTimestamp 转换器
├── io/
│   ├── __init__.py
│   ├── base.py                   # SourceAdapter 抽象接口
│   ├── his_adapter.py            # .his 读写适配器
│   ├── json_adapter.py           # 未来 JSON/YAML 适配器
│   └── http_adapter.py           # 未来网络 API 适配器
├── query/
│   ├── __init__.py
│   └── filter.py                 # 标签/时间/source/focus 过滤引擎
├── render/
│   ├── __init__.py
│   ├── axis.py                   # 通用时间轴（复用 UniversalTimeAxis）
│   ├── layout.py                 # Thread / Track / 自动布局
│   └── painter.py                # 绘制 event / period bar
├── agent/
│   ├── __init__.py
│   ├── schema.py                 # LLM 结构化输出 schema
│   └── service.py                # Agent 录入/查询服务
├── web/                          # 远期：后端 + 前端
│   ├── api.py
│   └── static/
└── tests/
    ├── test_time_conversion.py
    ├── test_his_adapter.py
    └── test_filter.py
```

**`History/` 保留为参考/遗留实现**，不直接改动；只从中抽取可复用算法（如自然语言时间解析、Track 布局）。

---

## 五、分阶段实施路线

### Phase 0：时间基础（最小可验证）✅ 已完成

1. 整理 `JDNTimestamp`、`TickStepper`、bridges 到 `chrono/`。
2. 实现 `chrono.history_time_adapter`：
   - `history_tick_to_jdn(tick: int) -> JDNTimestamp`
   - `history_record_time_range(record) -> (since, until)`
3. 编写转换测试：覆盖 BC/AD 边界、闰年、自然语言常见年份。
4. **验收**：所有时间往返测试通过（19/19），新增 BC/AD 边界测试通过。

### Phase 1：数据兼容层 ✅ 已完成

1. 设计并实现 `models.event`：`Event` / `EventIndex` / `Workspace`。
2. 实现 `adapters.his_adapter`：`HisFileAdapter` 读取 `.his` 文件 / depot / 目录，输出新模型。
3. 复用 `HistoryTime.time_text_to_ticks()` 的解析能力，在适配层转换为 `JDNTimestamp`。
4. **验收**：`tests/test_his_adapter.py` 通过，能正确加载 `History/depot/example/example.his` 并验证 6 条事件的时间、标签、source；**同时支持写回 `.his` 并保持可加载**。

当前运行结果：
```bash
cd UniversalHistory
python -m unittest test_history_time_basic test_comprehensive -v
python -m unittest discover -s tests -v
# 共 28 个测试全部通过
```

### Phase 2：新 Viewer 引擎 ✅ 初始实现已完成

1. 基于 `UniversalTimeAxis` 的坐标变换实现 `TimelineCanvas`。
2. 接入 `TickStepper` 绘制刻度。
3. 移植 `HistoryIndexTrack` 的 Thread/Track 自动布局算法。
4. 实现 event / period bar 绘制（横竖向统一）。
5. **验收**：能打开 example 数据，完成拖拽、缩放、横竖切换、单点/持续事件显示。

### Phase 3：编辑器与过滤（下一步）

1. 把 `HistoryRecordEditor` 适配到 `Event` 模型。
2. 实现 `Filter` 引擎与 UI（或先提供 API）。
3. 解决“保存到哪个文件”问题：Workspace 当前 source 作为默认；新增时可选 depots/files。
4. 实现修改后实时刷新 viewer（通过 Workspace 变更事件）。
5. **验收**：新建/编辑/删除事件，时间轴实时更新；filter 能按标签/时间筛选。

### Phase 4：Agent 与 Web（下一步）

1. 定义 Agent API（REST/WebSocket）与 LLM schema。
2. 实现 `AgentService`：
   - 接收自然语言 → 解析为事件；
   - 查询事件；
   - 把新增/修改事件广播到 UI。
3. 构建最小 Web 后端（FastAPI）和前端 Canvas 时间轴。
4. **验收**：通过 API 或 LLM 对话新增事件，桌面端和网页端同步可见。

### Phase 5：高级特性（预留接口）

1. 权限：记录增加 `owner/visibility`。
2. 协同：增加 `proposal/branch/review` 模型。
3. 不实现具体逻辑，但数据库/接口预留字段。

---

## 六、关键设计决策建议

| 决策点 | 建议方案 | 理由 |
|--------|----------|------|
| **时间模型** | 全面采用 `JDNTimestamp` | 标准、无精度损失、跨历法桥接已就绪。 |
| **是否保留 `.his` 格式** | 保留作为交换格式，但新增 `HisFileAdapter` | 保证旧数据可读、AI/人类友好；未来可扩展 JSON。 |
| **GUI 技术栈** | 桌面端先用 `PyQt6`；网页版用 FastAPI + Canvas | PyQt5 在 Python 3.14 已不可用；Web 版与桌面端可共用后端模型。 |
| **横竖向实现** | 复用 `UniversalTimeAxis` 的统一逻辑坐标系 + QTransform | 避免 History 中大量 `if layout == ...` 分支。 |
| **布局算法** | 移植 `HistoryIndexTrack` | 其 Track 分配策略已考虑稳定不闪烁，是核心资产。 |
| **AI 接口** | 先定义 JSON schema + 函数调用，再接入 LLM | 让 Agent 录入成为一等公民，而不是后期补丁。 |

---

## 七、下一步建议

当前最稳妥、风险最低的下一步是 **Phase 0 + Phase 1**：

1. 创建 `UniversalHistory/core/`、`UniversalHistory/time/`、`UniversalHistory/io/` 等包结构。
2. 实现 `HistoryTimeAdapter` 与转换测试。
3. 实现 `Event` 数据模型与 `HisFileAdapter`，加载 example 数据并输出新结构验证。

这样做的好处：
- 不破坏现有 History GUI；
- 先把“时间”和“数据”这两个最大风险点验证清楚；
- 为后续 Viewer/Editor 提供干净、标准的模型基础。

**暂不推荐的路线**：
- 直接改写 `History/viewer_ex.py` 或 `HistoryTime.py` 去兼容 `JDNTimestamp`——会让两个项目耦合，且 History 的 GUI 在 Python 3.14 已跑不起来。
- 一上来就写 Web 版——缺少稳定的数据和渲染抽象，会重复踩 History 的坑。

---

## 八、针对用户反馈的修正

> 用户进一步澄清：没有数据以 History `TICK` 持久化，唯一需要保留的是**自然语言时间文本 → 时间**的解析能力，且只发生在载入阶段，不必过分严谨。

### 8.1 关于时间转换

- **不需要为 `TICK` 写双向精确转换器**。
- 保留 `HistoryTime.time_text_to_ticks()` 的解析逻辑，但在 `HisFileAdapter` 里直接把解析出的 `(year, month, day, ...)` 交给 `JDNTimestamp.from_ymd_hms()` 即可。
- 由于用户不要求严格等价，只要“历史文章里复制下来的时间文字能认出来”，远古/史前时间允许存在与 Proleptic Gregorian 的细微偏差。

### 8.2 关于桌面端 vs 网页端

- 用户指出：网页要能承载操作，需要启动后端，比较重。
- **建议：以桌面端（Qt6）为主力编辑/浏览工具，走本地优先（local-first）路线；网页版作为可选的只读或共享视图，通过后端按需启用。**
- 这样既能保留“双击即可编辑、直接操作本地文件”的轻量体验，又为未来 Web 共享留下接口。

### 8.3 关于 QT5 → QT6

- 当前 Python 3.14 已无法安装 `pyqt5==5.15`；迁移到 `PyQt6` 是必然选择。
- 主要改动点：
  - 枚举访问方式：`Qt.AlignHCenter` → `Qt.AlignmentFlag.AlignHCenter`；
  - `QFontMetrics.width()` → `QFontMetrics.horizontalAdvance()`；
  - `exec_()` → `exec()`；
  - `QApplication.desktop()` → `QApplication.primaryScreen()` / `QWidget.screen()`；
  - 模块导入从 `PyQt5.*` 改为 `PyQt6.*`。
- 这些属于机械替换，风险可控。

### 8.4 关于层次命名与语义

为避免新旧概念混淆，建议统一术语表：

| 名称 | 语义 | 对应旧代码 |
|------|------|-----------|
| **Event / Record** | 一条历史记录，包含时间、地点、人物、组织、事件等要素 | `HistoryRecord` |
| **Index** | 轻量记录，仅用于展示（uuid/abstract/since/until/source） | `HistoryRecord`（focus='index'） |
| **Source** | 数据来源，可以是 `.his` 文件、URL、数据库记录等 | `source` 字段 |
| **Depot** | 本地文件仓库，对应磁盘上的一个目录 | `depot/` |
| **Workspace** | 当前内存中打开的所有 Source 及其 Event 集合 | `History` 类 |
| **Thread** | 时间轴一侧的展示线索，可放在左侧或右侧 | `HistoryIndexTrack` / `TimeThreadBase` |
| **Track** | Thread 内的横向/纵向轨道，用于避免事件重叠 | `TrackContext` |
| **Axis** | 中央时间轴 | `TimeAxis` / `UniversalTimeAxis` |

### 8.5 关于单点事件与时间段事件的布局

原 `HistoryIndexTrack` 把**单点事件强制放在第 0 个 Track**，不够灵活。用户希望：如果有更好的动态排版，单点事件不必局限于最靠内的时间轴。

**建议的新布局策略**：

1. 所有事件（单点/持续）统一参与 Track 分配。
2. 单点事件在纵轴（时间轴方向）上只有“一个瞬间”，但为了视觉不重叠，给它一个**基于屏幕像素的虚拟纵向区间**（例如以时间点为中心、±N 像素）。
3. 持续事件在纵轴上的区间就是 `[since, until]`。
4. Track 分配时，只判断两条记录在时间轴投影区间是否重叠（对单点事件使用其虚拟像素区间转换后的时间近似值）。
5. 如果 Thread 空间足够，单点事件可以像持续事件一样分布在任意 Track 上；空间不足时才 fallback 到最后一个 Track。
6. 保持“长持续事件优先、按持续长度降序排列”的稳定性策略，避免滚动时布局抖动。

这样既能保留原算法“稳定不闪烁”的优点，又能更灵活地利用 Thread 空间。

### 8.6 关于单点事件的现代 UI 呈现（用户补充）

用户提出：单点事件不一定要画成菱形/箭头并占用整条 Track，可以参考现代 UI 的紧凑展示方式：

- **固定长度卡片 + 截断文字**：每个单点事件显示为一个固定宽度/高度的小卡片，只展示标题/摘要前 N 个字，超出部分省略。
- **悬浮提示**：鼠标悬停时显示完整摘要Tooltip。
- **点击展开**：单击后展开为完整详情面板（侧边栏、浮层或弹窗）。

**设计建议**：

1. **统一把单点事件当作“纵向长度为 0、横向有固定像素尺寸的条”参与 Track 布局**。
   - 其纵向投影按当前缩放比例折算为一个很小的“时间宽度”。
   - 这样在布局算法上它与持续事件完全一致，只是参数不同。
2. **渲染时显示为圆角卡片/Chip**，而不是原来的菱形箭头。
   - 卡片中心对齐到时间点。
   - 文字截断并显示 `…`。
3. **交互分层**：
   - 悬停：全局十字准线 + Tooltip（显示完整时间 + 摘要）。
   - 单击：展开详情（可用侧边 Drawer 或浮层，避免遮挡时间轴）。
   - 双击：直接打开编辑器（保留原习惯）。
4. **当单点事件非常密集时**，可进一步做**聚合/聚簇**：同一小段时间内的多个点事件合并为一个“N 个事件”的簇，点击簇再展开列表。该功能可作为 Phase 3 之后的增强。

**优点**：
- 更现代、信息密度更高；
- 单点事件不再因为 Track 不够而被挤到最内侧；
- 展开机制解决了文字空间不足的问题，同时保持时间轴整洁。

---

## 九、备注

- `History/core.py` 中 `HistoryRecordIndexer.dump_to_file()` 的 `test_generate_index()` 调用缺少 `indexes` 参数，是原项目的小 bug，迁移时可顺手修正或忽略。
- `History/core.py` 的 `LabelTagParser.label_tags_list_to_dict()` 中 `if label not in label_tags_list` 疑似应为 `if label not in label_tags_dict`，当前未触发问题是因为 label 重复概率低；迁移到新解析器时应避免该 bug。
- 已创建本地虚拟环境 `.venv` 并安装 `lunar_python` / `requests`；PyQt 因版本问题未安装，迁移时统一处理。
