# WHY 总览 —— 动机全景与设计哲学速览

> 本文件是 why/ 目录的「总」入口：先给出全局图景，细节分篇展开，`99-summary.md` 收束为决策速查表。

## 一句话定位

`History` 是作者为**个人历史知识整理**自研的桌面时间轴工具：网上找不到能「在同一坐标轴上同时显示单点事件与时间段、可按五要素关联、支持多线索对照、数据归用户所有」的软件，因此自己动手（`History/README.md` 起因节）。

## 需求驱动

8 条原始需求决定了软件的全部主要功能，逐条见 [01-origin-and-requirements.md](01-origin-and-requirements.md)。其中三条是灵魂：

- **单点事件与时间段同轴显示** —— 直接决定了数据模型的 `since/until` 和渲染层的双形态绘制；
- **五要素（时间/地点/人物/组织/事件）识别与关联** —— 决定了 LabelTag 数据模型与 focus_label 概念；
- **多 filter 同视图对照** —— 决定了 Thread 体系与过滤器设计。

## 设计哲学速览

| 领域 | 核心决策 | 一句话理由 | 分篇 |
| --- | --- | --- | --- |
| 时间 | 自造 TICK（整数秒），不用 datetime | datetime 有 0 年缺失和范围限制，历史时间需正负无穷 | [02](02-time-philosophy.md) |
| 时间 | 解析容错优先 | 「文章中直接复制下来的时间文字都能顺利识别」 | [02](02-time-philosophy.md) |
| 时间 | 自研解析器，不用 dateutil | 「解析过程不可控，必须能识别历史日期」 | [02](02-time-philosophy.md) |
| 数据 | 纯文本 .his 格式 | 跨平台、人人可写工具、可存 GitHub 做 review/merge | [03](03-data-philosophy.md) |
| 数据 | 五要素 + focus_label | 一条记录可「以人观史」「以地观史」 | [03](03-data-philosophy.md) |
| 数据 | Index 与 Record 分离 | 面向在线/分布式窄带场景的摘要+指针 | [03](03-data-philosophy.md) |
| 渲染 | 布局稳定性优先于省计算 | 动态布局会让同一事件滚动时出现在不同列 | [04](04-rendering-philosophy.md) |
| 渲染 | 单点事件固定第 0 轨 | 单点宽度恒定、持续宽度随缩放变，混排破坏稳定 | [04](04-rendering-philosophy.md) |
| 渲染 | transverse/longitudinal 抽象 | 横纵布局共用同一套代码 | [04](04-rendering-philosophy.md) |
| 未来 | NLP/LLM 提取五要素 | 多年前就在代码注释里承诺的方向 | [05](05-ai-and-future.md) |

## 阅读建议

- 关心「迁移到 UniversalHistory 时哪些理由仍然成立」→ 先读 [99-summary.md](99-summary.md) 的遗产启示节；
- 关心某个具体决策的来龙去脉 → 按上表进入分篇。
