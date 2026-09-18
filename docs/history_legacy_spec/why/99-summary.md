# WHY 总结 —— 核心决策速查表与遗产启示

> 本文件收束 why/ 目录：全部设计决策一览 + 每条决策对 UniversalHistory 的意义。

## 决策速查表

| # | 决策 | 理由（一句话） | 出处 | 在 UniversalHistory 中的去向 |
| --- | --- | --- | --- | --- |
| 1 | 自造 TICK（整数秒时间） | datetime 有 0 年缺失和范围限制 | `HistoryTime.py:1-6` | 升级为 `JDNTimestamp`（微秒定点 + 天文纪年） |
| 2 | 「真实刻度 vs 历法标签」二分 | TICK 连续无限，历法是投射层 | `HistoryTime.py:3-5` | 保留，成为 chrono 层核心哲学 |
| 3 | BCE 月日镜像、仅供参考 | 远古历法本身失真，追求连续即可 | `HistoryTime.py:5` | 被外推格里高利历取代；验收不要求 BCE 月日精确一致 |
| 4 | 解析容错优先 | 录入痛点是「认不出」而非「误识别」 | `README.md:97,104-105` | **保留**：适配层复用旧解析管线 |
| 5 | 自研解析器，不用 dateutil | 解析过程不可控，必须识别历史日期 | `HistoryTime.py:167-168` | 保留（通过适配层） |
| 6 | 纯文本 .his 格式 | 跨平台、可拆分合并、Git review/merge | `example.his:21-30` | 保留为交换格式，Adapter 层隔离 |
| 7 | LabelTag 统一三类文档 | 一个解析器通吃 .his/.hisfilter/.index | `core.py:175-177` | 仅 .his 保留；filter/index 进模型层 |
| 8 | 五要素 + focus_label | 以人/地/组织观史的关联查找 | `core.py:407-418` | 保留进 `Event.labels` |
| 9 | Index 与 Record 分离 | 在线/窄带场景的摘要+指针 | `README.md:82` | 演化为 `EventIndex`（断链缺陷不复刻） |
| 10 | depot 目录组织、数据与程序同居 | 个人笔记工具定位 | `core.py:895-898` | 被 Workspace/Source 分层取代 |
| 11 | 布局稳定性优先于省计算 | 同一事件永远在同一列 | `viewer_ex.py:344-353` | **保留**：Track 分配策略是核心资产 |
| 12 | 单点事件固定第 0 轨 | 宽度恒定 vs 随缩放变，混排不稳 | `viewer_ex.py:346-348` | **有意放宽**：单点事件以虚拟像素区间统一分配（`migration_analysis.md` §8.5） |
| 13 | 刻度按人类习惯单位递进 | 年→月→周→日→时 | `README.md:173-175` | 保留并标准化为 `TickStepper` 层级金字塔 |
| 14 | transverse/longitudinal 抽象 | 横纵共用布局代码 | `viewer_utility.py:124-139` | 由统一逻辑坐标 + QTransform 完成（旧版未遂） |
| 15 | 刻度吸附（像素→精确 tick） | 映射精度影响悬停体验 | `viewer_ex.py:932-941` | 待在新实现中确认等价行为 |
| 16 | NLP/LLM 提取五要素 | 多年前承诺的方向 | `editor.py:308`、README:20 | 成为迁移核心目标（Agent 录入） |
| 17 | 纯文本 + Git 即协作方案 | fork + pull request + author 署名 | `README.md:139-141` | 演化为远期 proposal/review 模型（Phase 5） |

## 三条最重要的遗产

1. **「同一事件永远在同一列」**——布局稳定性是用户信任的基石，任何新布局算法都必须先满足它再谈优化。
2. **「文章中复制的时间文字都能认出来」**——录入零摩擦是数据积累的引擎，解析容错度只能增不能减。
3. **「数据归用户所有」**——纯文本、本地优先、可 Git 协作；Agent 和 Web 是增强而非接管（用户已澄清 local-first 路线，见 `migration_analysis.md` §8.2）。

## 已被用户反馈修正的决策

迁移过程中用户明确调整了两处（`migration_analysis.md` 第八章），文档评审时注意不要拿旧决策当验收标准：

- 时间转换：不需要 TICK 双向精确转换器，只在载入阶段解析自然语言文本（§8.1）；
- 桌面 vs 网页：桌面端（Qt6）为主力，local-first；网页版作为可选只读/共享视图（§8.2）。
