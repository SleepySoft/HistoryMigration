# HOW · Thread / Track 布局算法

> 模块：`History/viewer_ex.py` 的 `TimeThreadBase` / `HistoryIndexTrack`（108-548 行）。设计纲领（WHY：稳定性优先）见 [../why/04-rendering-philosophy.md](../why/04-rendering-philosophy.md)。

## 1. 概念

- **Thread**（`TimeThreadBase`/`HistoryIndexTrack`）：时间轴一侧的一条独立事件带，有自己的背景色和宽度；左右侧各可容纳多条，宽度由所在侧空间**均分**（不可单独设置）。
- **Track**（`TrackContext`）：Thread 内部的平行轨道。类 docstring：「The width of this thread divides track width is the track count」（`viewer_ex.py:108`）。

## 2. Thread 管理

- `add_history_thread(thread, align=ALIGN_RIGHT, base_thread=None)`（`:874-897`）：可指定相对 `base_thread` 插入其前（align LEFT）或后；触发 repaint。
- `remove_history_thread` / `remove_all_history_threads`（`:899-912`）。
- 背景色：右键 Add Thread 时按 `THREAD_BACKGROUND_COLORS[自增计数 % 16]` 轮换——**计数器先 +1 再取模，第一个 Thread 用第 2 个颜色**（`main.py:444-447`）。

## 3. Track 数量规则（arrange_items，`:429-443`）

- `track_count = round(Thread宽 / min_track_width)`（四舍五入），**最少 1 条**；
- `track_width = Thread宽 / track_count`（重新均分，实际轨宽未必等于设定值）；
- `min_track_width` 默认 = `REFERENCE_TRACK_WIDTH = 50`（`:111,367`），右键 "Set Track Width" 可改；
- track 数变化才重建 track（`__flag_build_track`）；任何 arrange 都置 layout/arrange flag（`:438-443`）。

## 4. 数据流与分步 flag

- `set_thread_event_indexes({source: [indices]})`（`:357,372`，20250126 由 list 改为 dict）：清空后逐 index 建 `HistoryIndexBar` 加入 axis_items 并登记 `__index_bar_table`（`:386-397`）。**注意：Thread 持有的是索引快照而非活数据**（见 [13-stale-ui-analysis.md](13-stale-ui-analysis.md)）。
- `refresh()`（`:131-138`）：按「item 时间区间与当前可视区间相交」（`item_since <= until and item_until >= since`，**含边界**）筛出 paint_items，然后 `arrange_items()`。
- 排序：布局输入按**持续时长降序**（长的优先，`:506-507`）；注释掉的旧代码保留了被否决方案「开始时间越早优先级越高」及取舍说明（`:457-461`）。
- 分步 flag：`build_track → layout_track → layout_items → arrange_items`（`:379-382,463-475`）——全量重排但最小化重复计算。

## 5. 分配算法（`__layout_track_items`，`:505-537`）

1. 输入按持续时长降序；
2. **轨 0**（最贴近时间轴）：**跳过所有持续事件**（`i==0 and since!=until: continue`）；单点事件**无条件**放入轨 0（`:521-523`）；
3. **其余轨**：持续事件若 `track.has_space()`（时间区间无严格重叠）则放入，否则留给下一轨；
4. **Fallback：最后一轨无条件接收所有剩余事件**（`i == len-1` 时忽略 has_space，`:523-525`）——空间不足时事件在末轨互相重叠绘制；
5. 放入即 `take_space_for` + `arrange_item(track.metrics)`（计算像素矩形并裁剪到 thread 纵向范围）；
6. **重叠补偿 hack**（`:529-535`）：若当前 bar 像素矩形与前一个完全相同，计数累加并 `shift_item(-3*overlap_count, 0)`——意图错开 3px。但因 `AxisMetrics.offset` 的 bug（`viewer_utility.py:230`）且随后 `__arrange_track_items` 对可见 bar 重新 `arrange_item` 会**重置该位移**（`:539-548`），**此补偿对可见事件基本无效**。README TODO:183「如何优化一年内多起事件的显示（重叠的问题）」印证此问题未解。

`__arrange_track_items`（`:539-548`）：缩放/滚动后对每条 track 中属于 paint_items 的 bar 按 track metrics 重算像素矩形。

## 6. Track 几何（`__layout_track`，`:483-503`）

- 纵向+ALIGN_RIGHT（默认）：轨 i 位于 `[axis_right + i*w, axis_right + (i+1)*w]`，向右依次排开；ALIGN_LEFT 则向左（transverse 数值递减，`:490-495`）。
- 横向模式左右镜像翻转（`:496-502`）。
- 残留 TODO："How to do it with the same interface"（`:488`）。

## 7. 单点 vs 持续事件差异汇总

| 维度 | 单点事件（since==until） | 持续事件 |
| --- | --- | --- |
| 轨道 | 永远轨 0 | 轨 1 起按空间分配，末轨兜底 |
| 像素宽度 | 恒定 = 轨宽（铺满整轨） | 随缩放变化 |
| 纵向像素范围 | 时间点为中心两侧各扩半个轨宽（正方形点击区） | since/until 映射像素并裁剪 |
| 图形 | 带 10px 箭头的五边形（指向轴） | 纯色矩形 |
| 背景色 | 浅灰 `(243,244,246)` | 青绿 `(185,227,217)` |
| 字体 | 6pt | 8pt |
| 提示文本 | `标题 : [年]` | `标题(N/M) : [起 - 止]` |

## 8. 迁移要点

- UniversalHistory 保留「长优先、全量重排、稳定不闪烁」内核；
- **有意放宽**：单点事件不再固定首轨，而是以基于屏幕像素的虚拟区间参与统一 Track 分配（`migration_analysis.md` §8.5）；末轨兜底改为空间不足时的 fallback；
- 「单点事件的现代 UI 呈现」（固定长卡片 + 截断 + 悬停 Tooltip + 点击展开、密集时聚簇）是 §8.6 的预留增强。
