# WHAT 总览 —— 软件能做什么

> 本文件是 what/ 目录的「总」入口：功能域清单与整体状态。WHAT 回答「用户能用这个软件做什么」；实现细节（HOW）在 [../how/](../how/)，设计动机（WHY）在 [../why/](../why/)。

## 软件形态

Windows/Linux/macOS 桌面应用（PyQt5），从 `History/main.py` 启动。主窗口中央是一个无限时间轴，轴两侧可放置任意多条「Thread」（历史线索）并排对照；数据是本地 `.his` 纯文本文件，按 depot 目录分类组织。

## 功能域清单

| 功能域 | 用户能做什么 | 状态 | 分篇 |
| --- | --- | --- | --- |
| 时间轴浏览 | 拖拽/滚轮平移、Ctrl+滚轮缩放（锚定鼠标时间点）、刻度随缩放自动换档（万年→小时）、悬停显示十字线与实时提示 | 可用 | [01](01-timeline-and-threads.md) |
| 横/纵切换 | 时间轴横向/纵向显示、轴位置可调 | **坏损**（设置对话框崩溃，功能无法到达） | [01](01-timeline-and-threads.md) |
| 多线索对照 | 轴两侧任意多条 Thread、每条挂不同文件、轨宽可调、背景色轮换 | 可用 | [01](01-timeline-and-threads.md) |
| 事件显示 | 单点事件（箭头五边形）与持续事件（矩形）不同形态、双击打开编辑器 | 可用 | [01](01-timeline-and-threads.md) |
| 事件编辑 | 五要素录入、倾向(focus)必填校验、字段锁定、按时间排序浏览记录、新建/删除/另存文件 | 可用（有缺陷） | [02](02-editing-and-data.md) |
| 数据载入 | 多选文件载入内存、全 depot 加载、depot 分类浏览 | 可用（Load Depot 空实现、Load All 阻塞） | [02](02-editing-and-data.md) |
| 过滤器 | 按 source/focus/标签包含排除筛选，生成索引文件 | 半成品（应用到 Thread 的路径已断） | [03](03-filter-and-index.md) |
| 索引 | 从记录生成轻量 index（时间+摘要+指针） | **断链**（生成与回读均坏） | [03](03-filter-and-index.md) |
| 帮助 | Help / About 菜单 | 空壳 | [99](99-status-matrix.md) |

## 整体评价

- **浏览与录入**（时间轴 + 编辑器）是完整可用的主体，承载了作者多年的实际使用（depot 中有中国史、世界史真实数据）。
- **过滤与索引**停在半成品：核心算法（`History.select_records`）完整且有测试，但 UI 链路断裂。
- **横纵切换**曾实现但已坏损；UniversalHistory 中已重新实现（Ctrl+T）。

详细状态矩阵（含与 UniversalHistory 的逐项对应）见 [99-status-matrix.md](99-status-matrix.md)。
