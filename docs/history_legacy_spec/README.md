# History（旧版）WHY / WHAT / HOW 行为规格

本目录是旧版 `History`（PyQt5 桌面时间轴应用）的结构化规格，作为迁移到 `UniversalHistory` 的**行为验收依据**。内容基于对 `History/` 源码的逐文件通读整理，所有行为标注 `文件:行号` 出处。

## 目录结构

```
history_legacy_spec/
├── why/    —— 为什么这样做：动机、需求、设计哲学
├── what/   —— 软件是什么：功能清单与状态
└── how/    —— 怎么做：完整行为规格（本文档主体）
```

三个子目录各自按「总（`00-overview`）→ 分（逐主题）→ 总（`99-summary`）」组织。

## 索引

### why/ —— 设计动机

| 文件 | 内容 |
| --- | --- |
| [00-overview.md](why/00-overview.md) | 总览：项目一句话、需求清单、设计哲学速览 |
| [01-origin-and-requirements.md](why/01-origin-and-requirements.md) | 项目起因与 8 条原始需求 |
| [02-time-philosophy.md](why/02-time-philosophy.md) | 时间系统决策：自造 TICK、容错优先、自研解析 |
| [03-data-philosophy.md](why/03-data-philosophy.md) | 数据决策：纯文本 .his、五要素、Index 分布式 |
| [04-rendering-philosophy.md](why/04-rendering-philosophy.md) | 渲染决策：布局稳定性、单点首轨、几何抽象 |
| [05-ai-and-future.md](why/05-ai-and-future.md) | NLP/LLM 长期方向与未完成 IDEAS |
| [99-summary.md](why/99-summary.md) | 总结：核心决策速查表与对新项目的启示 |

### what/ —— 功能总览

| 文件 | 内容 |
| --- | --- |
| [00-overview.md](what/00-overview.md) | 总览：功能域清单与整体状态 |
| [01-timeline-and-threads.md](what/01-timeline-and-threads.md) | 时间轴浏览与多线索对照 |
| [02-editing-and-data.md](what/02-editing-and-data.md) | 事件编辑与数据载入/depot |
| [03-filter-and-index.md](what/03-filter-and-index.md) | 过滤器与索引 |
| [99-status-matrix.md](what/99-status-matrix.md) | 总结：功能状态矩阵（可用/坏损/半成品）及与 UniversalHistory 的对应 |

### how/ —— 行为规格

| 文件 | 内容 |
| --- | --- |
| [00-overview.md](how/00-overview.md) | 总览：分层架构与数据流 |
| [01-time-system.md](how/01-time-system.md) | TICK 定义、闰年、tick↔日期算法、偏移、格式化 |
| [02-natural-language-time-parsing.md](how/02-natural-language-time-parsing.md) | 自然语言时间解析管线与中文数字转换 |
| [03-data-model.md](how/03-data-model.md) | HistoryRecord 字段、label 语义、index 投影、dump 规则 |
| [04-his-file-format.md](how/04-his-file-format.md) | .his 词法语法、实例对照、容错行为 |
| [05-loading-and-storage.md](how/05-loading-and-storage.md) | 加载/存储/depot 约定/History 内存库 |
| [06-filter.md](how/06-filter.md) | HistoricalFilter、select_records 语义、FilterEditor |
| [07-index-pipeline.md](how/07-index-pipeline.md) | 索引生成与断链缺陷 |
| [08-timeline-rendering.md](how/08-timeline-rendering.md) | 坐标映射、刻度体系、缩放平移、横纵切换、绘制、悬停 |
| [09-thread-track-layout.md](how/09-thread-track-layout.md) | Thread/Track 布局算法 |
| [10-interactions.md](how/10-interactions.md) | 右键菜单、Record 交互、菜单与快捷键总表 |
| [11-editor.md](how/11-editor.md) | 编辑器完整行为 |
| [12-main-window.md](how/12-main-window.md) | 主窗口行为 |
| [13-stale-ui-analysis.md](how/13-stale-ui-analysis.md) | 「编辑后不实时刷新」问题的三层原因分析 |
| [98-known-defects.md](how/98-known-defects.md) | 总结：36 条已知缺陷/怪癖/未完成项及迁移决策建议 |
| [99-migration-checklist.md](how/99-migration-checklist.md) | 总结：迁移验收要点速查 |

## 使用方式

- **迁移验收**：以 `how/99-migration-checklist.md` 为总清单，逐项核对新实现；细节回溯到对应主题文件。
- **缺陷决策**：`how/98-known-defects.md` 中每条标注「修复 / 复刻 / 废弃」建议，迁移时应显式决策而非无意照搬。
- **设计延续性**：`why/` 中的设计意图在 `UniversalHistory` 中是否仍成立，应逐条确认（部分已被用户反馈修正，见根目录 `migration_analysis.md` 第八章）。

## 相关文档

- 迁移动机：根目录 `migration_design.txt`
- 目标映射与阶段方案：根目录 `migration_analysis.md`
- 当前进度：`docs/PROJECT_STATUS.md`
