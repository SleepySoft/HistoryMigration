# HistoryMigration

HistoryMigration 是把旧版 `History` 桌面时间轴迁移到 `UniversalHistory` 的工作区。它本身不是一个新的 Python 包，而是通过 Git submodule 组织两个项目，并提供迁移分析、开发规范和入口说明。

## 仓库结构

| 路径 | 说明 |
| --- | --- |
| `History/` | 旧版 PyQt5 时间轴项目，保留 `.his` 解析器、编辑器和查看器，是迁移的参考实现。 |
| `UniversalHistory/` | 目标项目，基于 PyQt6、`JDNTimestamp` 和 `Workspace` 重新实现时间轴。 |
| `docs/` | 本仓库的补充文档，例如当前迁移状态。 |
| `migration_analysis.md` | 从 `History` 到 `UniversalHistory` 的代码分析、目标映射与阶段方案。 |
| `migration_design.txt` | 原始迁移目标和约束记录。 |

## 快速开始

### 1. 初始化 submodule

```bash
git clone --recurse-submodules <REPO_URL>
cd HistoryMigration
```

如果仓库已经克隆，可执行：

```bash
git submodule update --init --recursive
```

### 2. 安装 UniversalHistory 依赖

推荐使用 Python 3.10+ 和虚拟环境：

```bash
cd UniversalHistory
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m pip install requests
```

Windows PowerShell 使用：

```powershell
cd UniversalHistory
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m pip install requests
```

### 3. 启动桌面应用

```bash
python -m universal_history
```

应用默认会尝试加载 `History/depot/example/example.his`。因此应在完整工作区内运行，确保 `History/` 与 `UniversalHistory/` 是同级目录。

也可以安装为本地包：

```bash
cd UniversalHistory
python -m pip install -e .
universal-history
```

## 功能概览

- 统一时间模型：`JDNTimestamp` 使用微秒定点整数和外推格里高利历；天文纪年中 `0` 表示公元前 1 年。
- 旧格式兼容：`HisFileAdapter` 复用 `History` 解析器读取 `.his`，编辑和删除后可写回原文件。
- 工作区模型：`Event`、`EventIndex`、`Workspace` 分离数据、轻量索引和 UI，并通过信号刷新界面。
- 时间轴渲染：支持拖拽、滚轮浏览、`Ctrl+滚轮` 缩放、单点/持续事件布局、多 Thread 对照。
- 横纵切换：通过统一逻辑坐标与 `QTransform` 在横向和纵向间切换。
- 编辑与过滤：事件编辑器支持时间、地点、人物、组织、标签、标题、简介和事件正文；过滤器结果会复用单独的 `__filter__` Thread。

### 常用操作

| 操作 | 快捷键/方式 |
| --- | --- |
| 打开 `.his` | `Ctrl+O` |
| 打开事件编辑器 | `Ctrl+E` |
| 打开过滤器 | `Ctrl+L` |
| 适配当前数据范围 | `Ctrl+0` |
| 切换横/纵布局 | `Ctrl+T` |
| 打开 Thread Manager | `Ctrl+M` |
| 平移 | 左键拖拽，或滚轮沿时间轴滚动 |
| 缩放 | `Ctrl+滚轮` |
| 编辑事件 | 双击事件，或右键选择 `Edit event` |
| 新建/管理 Thread | 右键时间轴或 Thread 区域 |

## 运行测试

在 `UniversalHistory/` 中执行：

```bash
python -m unittest discover -s tests -v
```

项目当前使用标准库 `unittest`，没有强制引入 pytest。UI 测试可以在无桌面环境中尝试：

```bash
QT_QPA_PLATFORM=offscreen python -m unittest discover -s tests -v
```

## 架构概览

```text
.his file
  -> HisFileAdapter / History legacy parser
  -> Event / EventIndex / Workspace
  -> TimelineView -> ThreadLayout / TrackLayout
  -> PyQt6 painting
```

核心目录：

- `UniversalHistory/universal_history/chrono/`：JDN 时间、历史时间适配、农历桥接和刻度步进。
- `UniversalHistory/universal_history/models/`：事件、索引和工作区。
- `UniversalHistory/universal_history/adapters/`：`.his` 读写和未来格式适配层。
- `UniversalHistory/universal_history/render/`：坐标系、Thread/Track 布局和绘制。
- `UniversalHistory/universal_history/ui/`：主窗口操作、编辑器、过滤器和 Thread 管理对话框。

详细设计见 [`UniversalHistory/docs/core_design.md`](UniversalHistory/docs/core_design.md)、[`UniversalHistory/docs/zoom_design.md`](UniversalHistory/docs/zoom_design.md) 和 [`migration_analysis.md`](migration_analysis.md)。

## 迁移状态

当前状态摘要见 [`docs/PROJECT_STATUS.md`](docs/PROJECT_STATUS.md)。简而言之：

1. `History` 的数据格式、时间解析和布局经验已经迁移到 `UniversalHistory`。
2. 事件模型、`.his` 读写、PyQt6 时间轴、事件编辑器和基础过滤器已有实现。
3. Agent API、Web 前端、权限和协同编辑仍是预留阶段。

## 已知边界

- `HisFileAdapter` 依赖同级 `History/` 目录来复用旧解析器；单独分发 `UniversalHistory` 包时需要处理该依赖。
- 旧解析器导入 `requests`，因此当前运行环境需要额外安装该包。
- 目前持久化主要面向 `.his` 文本格式，JSON、数据库和网络源仍需扩展 Adapter 层。
- 过滤结果使用内存中的 `__filter__` Thread，不对应可写文件。
- 旧版 `History` GUI 继续使用 PyQt5，不建议在现代 Python 环境中作为迁移基础；它当前主要作为参考实现和解析器来源。

## 贡献与协作

开发前请阅读 [`AGENTS.md`](AGENTS.md)。修改行为时保持架构边界清晰：时间模型不使用浮点数，UI 不直接解析 `.his`，Adapter 不直接操作 Qt 绘制。提交前应运行相关测试，并同步更新受影响的文档。
