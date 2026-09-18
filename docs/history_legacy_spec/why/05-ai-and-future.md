# WHY · AI 方向与未完成的设想

> 出处：`README.md` 更新与通知节（16-29 行）、IDEA 区（199-211 行）、`editor.py:308`、`History/Doc/img-nlp-comments.png`。

## NLP/LLM 是多年前就承诺的方向

README 更新与通知节讲述了一个时间跨度故事：作者数年前在代码注释里留下一句话（截图存于 `Doc/img-nlp-comments.png`），承诺用自然语言处理提取历史事件要素；「而现在，终于到了可以兑现的时候」。

编辑器里的四个 **Auto Detect** 按钮是这个承诺的占位：全部弹 "Not implemented" 提示，文案为 `'In feature, we will use NLP or LLM to recognize the %s information from main text'`（`editor.py:308`，"In feature" 是 "In future" 的笔误）。

作者的下一步构想（README:27）：

> 如何做一个真正可以在商业和学术上使用的标准历史时间轴，以及如何借助 LLM 让大家无阻力的构造自己的历史知识体系。

## README IDEA 区的未完成设想

| 设想 | 出处 |
| --- | --- |
| 对于时间暧昧事件（特别是远古时代）的处理 | `README.md:201` |
| 使用相对时间的事件的处理 | `:202` |
| 使用不同的时间基准（公元、万历、民国等） | `:203` |
| 使 index 更小却能索引更多内容；极小 index 下的检索 | `:205-206` |
| 网页端的展示界面（js） | `:207` |
| 使用 NLP 算法提取四要素 tag；将提取的时间数字化 | `:210-211` |
| 多 thread 界面设计：label:tags 格式 filter、filter 预设/URL 传递 | `:194-197` |
| 合并文件：多个小文件合并成大文件 | `:188` |
| 一年内多起事件的显示重叠问题；同一时代人物太多空间不足 | `:184-185` |
| 五要素外其它 tags 的编辑功能 | `:190` |

## 对迁移的启示

这些设想逐条对应了 UniversalHistory 的迁移目标（根目录 `migration_design.txt`）：

- 「NLP/LLM 提取」→ **AI/Agent 录入友好**（与 Agent 对话过程中由 Agent 整理并添加事件，界面同步更新）；
- 「网页端展示」→ **Web 前端**；
- 「不同时间基准」→ UniversalHistory 的桥接层（农历/干支已实现，年号纪元可扩展）；
- 「filter 预设」→ 过滤器功能的完善。

即：**迁移不是推翻重来，而是兑现旧项目拖欠多年的承诺**。理解这一点有助于在 Agent Service 设计（`migration_analysis.md` Phase 4）时回到作者的本意——「无阻力地构造自己的历史知识体系」。
