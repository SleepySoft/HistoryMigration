# WHAT 总结 —— 功能状态矩阵

> 本文件收束 what/ 目录：全部功能逐项的状态判定，及与 UniversalHistory 的对应关系（对应关系依据 `docs/PROJECT_STATUS.md` 与 `migration_analysis.md`）。

## 状态图例

- ✅ 可用：功能完整，日常使用无大碍；
- ⚠️ 可用但有缺陷：主路径可用，存在数据/体验缺陷；
- 🔶 半成品：核心实现存在，但 UI 链路不完整；
- ❌ 坏损：功能无法到达或必崩溃；
- ⬜ 空壳：菜单/按钮存在但无实现。

## 功能状态矩阵

| 功能 | 状态 | 说明 | UniversalHistory 对应 |
| --- | --- | --- | --- |
| 无限时间轴浏览 | ✅ | 拖拽/滚轮平移 | TimelineView 平移 |
| 锚定缩放 | ✅ | Ctrl+滚轮，鼠标时间点不动 | Ctrl+滚轮缩放 |
| 刻度自动换档 | ✅ | 48 档，年→月→周→日→时递进 | TickStepper 标准层级 |
| 公元前显示 | ✅ | 刻度 `BC xxxx`；但 format_tick 丢失 BC 信息 | 天文纪年完整支持 |
| 悬停实时提示 | ✅ | 十字线 + 提示框，持续事件显示第N年/共M年 | 悬停提示 |
| 单点/持续事件双形态 | ✅ | 五边形 vs 矩形 | 保留并改进（卡片式设计预留） |
| 双击编辑事件 | ✅ | 弹模态编辑器 | 双击编辑 + 右键菜单 |
| 多 Thread 对照 | ✅ | 两侧任意多条、宽度均分 | 左右多 Thread + Thread share |
| Track 自动布局 | ✅ | 长优先、末轨兜底 | 移植并改进（单点不再固定首轨） |
| 横/纵切换 | ❌ | 对话框崩溃无法到达 | 已实现（Ctrl+T） |
| 轴位置调整 | ❌ | 同上 | 轴偏移（ThreadManagerDialog） |
| 五要素编辑 | ⚠️ | focus=time 永远无法 Apply；Apply 丢失未暴露字段 | EventEditor |
| 字段锁定 | ✅ | 连续录入保值 | 待确认 |
| 时间自然语言输入 | ✅ | 容错管线 + 悬浮提示核对 | 适配层复用旧解析 |
| 文件载入（Load Files） | ✅ | 多选载内存 | Ctrl+O 打开 .his |
| Load Depot | ⬜ | 空实现 | —（source 概念取代 depot） |
| Load All Records | ⚠️ | 可用但阻塞 UI | — |
| 编辑后实时刷新 | ❌ | 索引快照 + 无信号机制，从不刷新 | Workspace 信号驱动刷新（核心改进） |
| 过滤器定义与保存 | 🔶 | 可编辑可存盘，无法应用到 Thread | FilterDialog 已实现全链路 |
| 索引生成/回读 | ❌ | 生成脚本与回读均断链 | EventIndex（内存索引，无文件管线） |
| Thread 配置界面 | ❌ | ThreadEditor 半成品不可到达 | ThreadManagerDialog |
| 方向键滚动 | ❌ | 坏损（NameError 级 bug） | 待确认 |
| Help / About | ⬜ | 空壳 | — |
| 退出确认 | ✅ | 中文确认框 | 待确认 |

## 结论

- 旧版**可用主体** = 浏览 + 录入 + 多线索对照，这正是 `migration_design.txt` 要求「保留 History 原有功能和特性」的核心集；
- **坏损/半成品**（横纵切换、过滤上轴、索引、Thread 配置、编辑刷新）恰好对应迁移目标中的「需要改进」项，且全部已在 UniversalHistory 中重新实现或有明确方案；
- 迁移验收不应以「旧版能跑」为对照（PyQt5 在新 Python 上已装不上），而应以本目录 + `how/` 的行为规格为对照。
