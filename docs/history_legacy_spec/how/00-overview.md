# HOW 总览 —— 分层架构与数据流

> 本文件是 how/ 目录的「总」入口。HOW 描述软件的**完整行为**：每条规则标注 `文件:行号` 出处，均可作为迁移验收的核对项。缺陷汇总见 [98-known-defects.md](98-known-defects.md)，验收清单见 [99-migration-checklist.md](99-migration-checklist.md)。

## 分层架构

```
用户交互层   main.py（主窗口、菜单、右键菜单、快捷键）
             editor.py（记录编辑器）  filter.py（过滤器编辑器）
                │
控件层       viewer_ex.py（TimeAxis / TimeThreadBase / HistoryIndexTrack / HistoryIndexBar）
             viewer_utility.py（AxisMetrics / AxisMapping / TrackContext）
             ui_utility.py（ShadowLineEdit / WrapperQDialog 等）
                │
数据层       core.py（LabelTag / HistoryRecord / HistoryRecordLoader /
             HistoryRecordIndexer / History 内存库）
                │
算法层       Utility/HistoryTime.py（TICK、tick↔日期、自然语言解析）
             Utility/to_arab.py（中文数字转换）
```

关键架构事实：

- **无信号/回调机制**：`History` 内存库变更不会通知任何 UI——这是「编辑后不刷新」问题的结构性根源（[13-stale-ui-analysis.md](13-stale-ui-analysis.md)）。
- **右键菜单和键盘行为在 `main.py` 而非 `viewer_ex.py`**——「视图层」不等于 viewer_ex.py 一个文件。
- **`core.py` 无 PyQt 依赖**，但依赖 `requests`（Web 加载）和 `Utility.HistoryTime`。
- PyQt5；在新 Python（3.14）上 `pyqt5==5.15` 已无二进制 wheel 可装，旧 GUI 实际无法在现代环境运行。

## 全局数据流

```
.his 文件 ──from_text──▶ {source: [HistoryRecord]}（History 内存库）
                              │
                ┌─────────────┼──────────────────┐
                ▼             ▼                  ▼
         select_records   to_index()          save_source
         （四步过滤）     （索引快照）        （整文件覆写 utf-8）
                │             │                  │
                ▼             ▼                  ▼
           FilterEditor   Thread 显示        .his 文件
           （Generate     （快照！编辑
            Index）        后不同步）
```

## 主题索引

| 主题 | 文件 |
| --- | --- |
| 时间系统（TICK/闰年/转换/偏移/格式化） | [01-time-system.md](01-time-system.md) |
| 自然语言时间解析 | [02-natural-language-time-parsing.md](02-natural-language-time-parsing.md) |
| 数据模型（HistoryRecord/label/index 投影/dump） | [03-data-model.md](03-data-model.md) |
| .his 文件格式 | [04-his-file-format.md](04-his-file-format.md) |
| 加载与存储、depot、内存库 | [05-loading-and-storage.md](05-loading-and-storage.md) |
| 过滤器 | [06-filter.md](06-filter.md) |
| 索引管线 | [07-index-pipeline.md](07-index-pipeline.md) |
| 时间轴渲染（坐标/刻度/缩放/绘制/悬停） | [08-timeline-rendering.md](08-timeline-rendering.md) |
| Thread/Track 布局算法 | [09-thread-track-layout.md](09-thread-track-layout.md) |
| 交互总表（右键/双击/快捷键/菜单） | [10-interactions.md](10-interactions.md) |
| 编辑器 | [11-editor.md](11-editor.md) |
| 主窗口 | [12-main-window.md](12-main-window.md) |
| 编辑后不刷新问题分析 | [13-stale-ui-analysis.md](13-stale-ui-analysis.md) |
