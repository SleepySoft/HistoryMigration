# Project Status

更新时间：2026-09-18。

## 当前阶段

`UniversalHistory` 已经越过“仅时间轴 Demo”阶段，具备最小可用的桌面时间轴应用。旧版 `History` 继续作为参考实现和 `.his` 解析器来源。

## 已完成

### 时间模型

- `JDNTimestamp` 使用微秒定点整数表示时间。
- 使用外推格里高利历和天文纪年。
- 提供 Python `datetime` 桥接与农历/干支桥接。
- `TickStepper` 可按像素密度选择时间刻度。

### 数据与格式

- `Event` 承载 UUID、source、起止时间和标签。
- `EventIndex` 承担轻量渲染索引。
- `Workspace` 聚合多个 source，并提供增加、更新、删除、查询和信号通知。
- `HisFileAdapter` 可读取 `History` 的 `.his` 文件，并把编辑和删除结果写回原 source。
- 旧 `HistoryTime` 的自然语言时间解析通过适配层转换为 `JDNTimestamp`。

### 桌面应用

- `TimelineView` 支持 Thread 布局、平移、缩放、悬停提示、双击编辑和右键菜单。
- 支持左右两侧多 Thread、Thread share、横纵切换和适配当前数据范围。
- `EventEditor` 可新建、更新和删除事件，并写回 source。
- `FilterDialog` 可按 source、focus label、包含/排除标签和时间范围筛选。
- `ThreadManagerDialog` 可调整 Thread、方向、份额和轴偏移。

## 未完成

- Agent API：尚未提供 REST/WebSocket 接口，也没有 LLM schema 和事件广播协议。
- Web 前端：尚未实现。
- 非 `.his` 持久化：JSON、数据库和网络 Adapter 仍是设计预留。
- 权限与协同：未实现 owner/visibility、proposal 或 review 模型。
- 部署打包：当前仓库提供源码运行方式，尚未建立正式发布产物。

## 依赖与环境注意

`UniversalHistory` 运行需要：

```text
PyQt6>=6.0,<7.0
lunar_python>=1.0
requests>=2.0
```

`requests` 来自旧版 `History/core.py` 解析器的导入依赖。建议使用独立虚拟环境。启动和 Adapter 解析都假定完整工作区中的 `History/` 与 `UniversalHistory/` 同级。

## 最近验证

在 2026-09-18 使用 Python 3.11 和独立虚拟环境安装 `PyQt6`、`lunar_python` 和 `requests` 后执行：

```bash
QT_QPA_PLATFORM=offscreen python -m unittest discover -s tests -v
```

结果：61 个测试全部通过。

## 最新设计裁决（记录于 UniversalHistory/docs/spec/）

1. **Track 是正式概念层级**：Thread > Track > Item，Track 的轨数公式、分配顺序、末轨兜底属布局行为规格（`spec/how/07-layout.md` §0）。
2. **Index 是简化记录而非独立概念**：仅保留正文之外的摘要 + 指针，为减小传输尺寸而设计，由 Event 现场派生（`spec/how/04-models.md` §2）；旧版 .index 文件管线废弃。
3. **事件归属与文件管理重新设计**（不沿用旧版混乱）：事件恰好归属一个 source、归属可见、写回原 source、新建默认归属当前 Thread 绑定 source（`spec/how/14-event-ownership.md`）。
4. **以时间轴为主界面**：新建/编辑/删除从轴上下文发起，编辑器降级为轴的从属单事件对话框（`spec/how/15-timeline-centric-editing.md`）。

## 下一步建议

1. 建立可脱离 sibling 仓库导入的解析器边界，评估将旧 `.his` 解析逻辑迁移进 `UniversalHistory` 或独立包。
2. 为 `Workspace` 和 `HisFileAdapter` 补充保存冲突、空 source、重复 UUID 等边界测试。
3. 定义 Agent Service 的最小 JSON schema 和 UI 同步信号/事件流。
4. 引入 FastAPI/WebSocket 最小后端，复用 `Event`、`EventIndex` 和 `Workspace` 语义。
