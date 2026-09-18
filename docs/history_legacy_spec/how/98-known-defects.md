# HOW 总结一 · 已知缺陷 / 怪癖 / 未完成项汇总

> 迁移决策建议：**修复** = 新实现中修正；**复刻** = 为兼容而保留；**废弃** = 不再实现；**观察** = 可能随技术栈更换自然消失。
> 凡未列于此表的行为，均应视为需要保留的既有功能。

| # | 位置 | 问题 | 建议 |
| --- | --- | --- | --- |
| 1 | `core.py:506,509` + `:588-591` | index 回读缺 `decimal_year_to_tick`，index 文件写出读不回 | 修复（EventIndex 直接持有 JDNTimestamp，无此问题） |
| 2 | `core.py:294` | 重复 label 只保留最后一次 tags（条件比较对象写错，恒为 True） | 修复 |
| 3 | `core.py:464-470` | `people()/location()/organization()` 调用不存在的方法，必抛 AttributeError | 修复 |
| 4 | `core.py:456` | `duplicate_from` 引用复制 label_tags，改副本改原件 | 修复 |
| 5 | `core.py:635-636` | 时间解析失败 since=until=0.0（float，与 int TICK 类型不纯） | 修复 |
| 6 | `core.py:342` | `add_tags`/`list_unique` 用 set 去重**打乱 tag 顺序** | 修复（保序去重） |
| 7 | `core.py:794-834` | 加载失败与空文件不可区分（异常静默返回 {}） | 修复 |
| 8 | `core.py:861,871` | dict 键（原始 source）与记录内 source（归一化相对路径）不一致 | 修复 |
| 9 | `core.py:907-918` | 目录加载不做后缀过滤，depot 里任何文件都被当 .his 解析 | 修复 |
| 10 | `core.py:283-284` | 反斜杠自身不转义；含空格的 tag 也被 `"""` 包裹 | 修复（写出侧）；**注意保持读回兼容** |
| 11 | `core.py:946-952` + `indexer.py` | 索引生成脚本签名不匹配 + 返回值丢弃，管线无法运行 | 废弃（旧 .index 管线） |
| 12 | `main.py:367-392` | 方向键滚动坏损（`self.timer`/`self.keys_pressed` 不存在 + 不 repaint） | 修复 |
| 13 | `main.py:311-312` | Load Depot 空实现 | 修复 |
| 14 | `main.py:100,121-127,181-187` | Axis Appearance Setting 构造即 TypeError，横纵切换/Thread 配置无法到达 | 修复（UniversalHistory 已实现 Ctrl+T 与 ThreadManagerDialog） |
| 15 | `main.py:263,269` | 两个 View 菜单项快捷键都是 Ctrl+R（冲突） | 修复 |
| 16 | `main.py:337-356` | Help/About 空壳 | 废弃或修复 |
| 17 | `main.py:419-421,466-510` | Load Index/Use Filter 菜单移除、死代码残留；filter→Thread 路径断 | 修复（FilterDialog 已实现全链路） |
| 18 | `main.py:307` + `core.py:880-887` | 路径前缀剥离留下前导分隔符，跨平台行为微妙 | 修复（pathlib） |
| 19 | `main.py:314-317` | Load All 同步阻塞 UI 线程 | 修复 |
| 20 | `main.py:445-447` | Add Thread 颜色计数先自增再取模，首个 Thread 跳过调色板第一个颜色 | 复刻与否均可（无功能影响） |
| 21 | `main.py:459-462` | Set Track Width 无范围钳制（ThreadEditor 里有 10-100 钳制，不一致） | 修复 |
| 22 | `main.py:477` | Load File 对话框过滤器文字误写为 "Filter Files (*.his)" | 修复 |
| 23 | `viewer_utility.py:230` | `AxisMetrics.offset` 把 wide_offset 加到 longitudinal_until（平移变拉伸） | 修复（新渲染层已重写） |
| 24 | `viewer_utility.py:283-289` | `has_space` 不检测「新 bar 完全包含已有 bar」（依赖长优先排序规避） | 修复（新布局显式处理） |
| 25 | `viewer_ex.py:529-548` | 同位置事件 3px 错位补偿被重排覆盖，重叠问题未解 | 修复（新布局策略见 `migration_analysis.md` §8.5） |
| 26 | `viewer_ex.py:1275,1346` | 刻度绘制 `assert paint_tick <= tick_since` 在边缘 tick 可能崩溃 | 修复 |
| 27 | `viewer_ex.py:319-333` | 事件文字无截断/省略号策略，长标题显示不全 | 修复（截断 + Tooltip + 点击展开，§8.6） |
| 28 | `viewer_ex.py:860-861` | `jump_to_tick` 空 stub | 修复 |
| 29 | `viewer_ex.py:813` | `set_axis_offset` 越界不生效但仍 repaint | 修复 |
| 30 | `viewer_ex.py:150-156` | 命中测试遍历全部 axis_items 而非仅可见项 | 修复 |
| 31 | `viewer_ex.py:946-948` | 滚轮横向分量 angle_x 读取后未使用 | 观察 |
| 32 | `editor.py:477-478` | focus=time 分支漏设 `input_valid`，选 Time 倾向永远无法 Apply | 修复 |
| 33 | `editor.py:374-377,502` | Apply 重建记录导致 UI 未暴露的 label（author、自定义 tags）丢失；隐藏 tags 行写入空 tag | 修复 |
| 34 | `editor.py:512-526` | 回填不还原 focus radio，旧记录重编时 focus 默默变为 Event | 修复 |
| 35 | `editor.py:528-531` 等 | New Event / Cancel / 切换下拉框均不提示未保存修改 | 修复 |
| 36 | `editor.py:353-361` | Del Event 无确认且立即落盘；remove_record 跨所有 source 按 uuid 删 | 修复（应加确认、限定 source） |
| 37 | `editor.py:806-807` | 每次 Apply/Del 弹模态消息框，连续录入体验差 | 修复 |
| 38 | `editor.py:620` | depot 下拉框只构造时枚举一次，运行期不刷新 | 修复 |
| 39 | `editor.py:176-177` | 「Label Tag Editor」tab 为空表占位（Front/Back 卡片计划未完成） | 废弃或实现 |
| 40 | `editor.py:308` | Auto Detect 四个按钮均为占位；文案 "In feature" 笔误 | 修复（Agent 录入的真正入口） |
| 41 | `DateTimePicker.py:65` | 演示代码解包数量错误 | 废弃 |
| 42 | `filter.py:245,256` | .hisfilter 保存未指定编码（Windows GBK）与 utf-8 读端不一致 | 修复（统一 utf-8） |
| 43 | `filter.py:269-270,129-133` | Check 按钮、get/set_filter 空实现 | 废弃或修复 |
| 44 | `filter.py:33-43,300-306` | include/exclude 语法校验失败静默丢弃整组条件且无提示 | 修复 |
| 45 | `filter.py:285,261` | Generate Index 对空结果也提示成功；Load 失败无 UI 反馈 | 修复 |
| 46 | `HistoryTime.py:98` | 「至今」在模块 import 时求值一次，长进程内过期 | 修复 |
| 47 | `HistoryTime.py:184-202` | `format_tick` 丢失公元前信息；show_date+show_time 时月日与时间无空格 | 修复（新刻度已加 BC 前缀） |
| 48 | `HistoryTime.py:44,400-410` | TICK_WEEK 注释错误；year_ticks/year_days docstring 写反 | 修复 |
| 49 | `HistoryTime.py:299-304` | 无「年」字样的数字段一律当公元年份（散文数字误识别） | **复刻**（用户明确要容错） |
| 50 | `HistoryTime.py:94-99` | 「世纪→00」粗糙替换（21 世纪→2100 年而非区间） | 复刻或改进 |
| 51 | `HistoryTime.py:774-795` | `offset_date_time` 无跨年（无 0 年）修正，与 `offset_ad_second` 语义不一致 | 修复（新实现统一走 JDNTimestamp） |
| 52 | `HistoryTime.py:640-647` | `tick_to_days` 等 assert 只接受正区间，BCE 全靠调用方取绝对值，易误用 | 修复 |
| 53 | `README.md:55` | 双击弹出的编辑器窗口可能跑到后台（疑 PyQt5 bug） | 观察（PyQt6 下可能自然消失） |
| 54 | `README.md:178-211` | 未完成 IDEAS：filter 预设/URL 传递、NLP 提取、相对时间、多时间基准、合并文件 | 设计预留（`migration_analysis.md` Phase 4/5） |
| 55 | 多处 | 大量注释掉的旧实现/调试输出；死代码（CommonMainWindow、EasyQListSuite 等） | 废弃（不迁移） |
