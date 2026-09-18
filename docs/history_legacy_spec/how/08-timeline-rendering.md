# HOW · 时间轴渲染

> 模块：`History/viewer_ex.py` 的 `TimeAxis`（555-1541 行）+ `History/Utility/viewer_utility.py`。WHY（布局哲学）见 [../why/04-rendering-philosophy.md](../why/04-rendering-philosophy.md)。

## 1. 基础设施（viewer_utility.py）

- **字体**：`event_font` = 微软雅黑 6pt（单点事件），`period_font` = 微软雅黑 8pt（持续事件）（`:23-29`）。
- **颜色表**（`:34-41`）：`AXIS_BACKGROUND_COLORS` 6 色，实际只用索引 2 = `(252,157,154)` 粉红（`viewer_ex.py:1257`）；`THREAD_BACKGROUND_COLORS` 16 色供 Thread 背景轮换。
- **常量**：`LAYOUT_HORIZON=1`、`LAYOUT_VERTICAL=2`、`ALIGN_LEFT=4`、`ALIGN_RIGHT=8`（`:45-51`）。
- **AxisMapping**（`:59-115`）：范围 A↔B 线性映射，双向换算；分母为 0（容差 1e-9）返回 0.0；`set_range_ref(ref_a, ref_b, origin_a, origin_b)` 以「参考长度+原点」配置映射。
- **AxisMetrics**（`:122-258`）：矩形 = transverse_left/right（横向限度）+ longitudinal_since/until（纵向范围）+ scale_since/until（时间刻度范围）+ align + layout。`rect()` 横向时 transverse_left > transverse_right（镜像）；`value_to_pixel()/pixel_to_value()` 走内部 AxisMapping。**bug**：`offset()` 中 `longitudinal_until += wide_offset` 应为 `long_offset`（`:230`）。
- **TrackContext**（`:265-294`）：记录一条 track 的 metrics 与已分配 bar 列表；`has_space(since, until)` 判**严格相交**（首尾相接不算冲突；不检测「新 bar 完全包含已有 bar」——依赖长优先排序规避）；`take_space_for` 去重置尾。

## 2. TimeAxis 初始化与默认值

- 默认纵向布局；默认时间范围 0 ~ 2000 年（`viewer_ex.py:787`）。
- 轴位置比例 0.5（居中）；轴区半宽 30px，两侧各留 10px（`:731-732,1148-1149`）。
- 最小窗口 800×600；`setMouseTracking(True)`（`:789-791`）。
- 实时提示默认开启，字体微软雅黑 8pt。
- `MAIN_SCALE_MIN_PIXEL=50`（一个主刻度最少 50px，`:715-716`）；刻度档限制默认 1 天 ~ 1000 万年；时间范围限制默认无。

## 3. 坐标映射

- 三层换算：tick →（`AxisMapping`，比例 = `page_tick : page_pixel`）→ 相对像素偏移 → 加 `scroll + offset` 得屏幕像素；反算用 `b_to_a`（`:1108`）。
- `check_update_pixel_scale()`（`:1091-1108`）：`scale_pixel = 页面纵向像素 / scale_per_page`，不足 50px 强制 50；变化时置 `__scale_updated`。
- `check_update_scroll_offset()`（`:1110-1134`）：pending 的 seeking 对齐；时间范围限制 clamp；总偏移变化时把 scale 范围设为 `[b_to_a(offset), b_to_a(offset+页长)]`。
- **刻度吸附**：`tick_from_point`（`:932-941`）先查 `__optimise_pixel`（绘制刻度时记录的「像素→精确 tick」表），命中返回精确刻度 tick，否则线性反算。该表每次绘制刻度前清空、逐刻度填入。

## 4. 刻度体系

### Scale 结构（`:564-630`）

- 三要素：`main_scale_offset` / `sub_scale_offset`（均为 (年，月，日，时，分，秒) 六元组）+ `scale_per_page`（每屏主格数）。
- `rough_offset_tick()`：粗算 tick（年×TICK_YEAR + 年//4×TICK_DAY 闰年补偿 + 月×30天 + …），仅用于档位比较；实际刻度推进走 `offset_ad_second`（精确日历运算，处理无 0 年）。
- `estimate_closest_scale(tick)`（`:585-598`）：分解为 6 元组，从最高位起遇到第一个非零 offset 位后低位清零（月/日清零改 1），非零位向下取整到 offset 整数倍——求「≤ 当前显示起点的最近主刻度」。
- `format_main_scale_text`（`:606-630`）：按最高非零位选格式——年 `%04d`、月 `%04d/%02d`、日 `%04d/%02d/%02d`、时 `%02dH`、分/秒 `%02d:%02d:%02d`；**公元前年份取绝对值加 `BC ` 前缀**。

### STEP_LIST（`:632-704`）——48 档，跨数量级的「接缝」档全部被注释掉

| 主刻度 | 副刻度 | 每屏格数 |
| --- | --- | --- |
| 1000 万年 | 100 万年 | 1 档 |
| 100 万 / 10 万年 | 10 万 / 1 万 | 递进 |
| 1 万年 | 2000 年 → 1000 年 | 10→9→8→7→6→5 → 4→3→2 |
| 1000 年 | 250 年 → 200 年 → 100 年 → 50 年 | 20→18→16→14→12 → 10→…→6 → 5→4→3 → 2 |
| 100 年 | 20 年 → 10 年 → 5 年 | 10/8 → 5 → 2 |
| 10 年 | 2 年 → 1 年 | 10/8 → 5/2 |
| **1 年** | **4 个月 → 2 个月 → 1 个月** | 10/8 → 5 → 2 |
| **1 个月** | **7 天** | 12/8/6/4 |
| **7 天（周）** | **1 天** | 8/6/4 |
| **1 天** | **4 小时 → 2 小时 → 1 小时** | 14/7/5 → 2 → 1 |

注意：`Doc/Readme_Viewer_CN.md` 的刻度表（5000年/2500年等）**与当前 STEP_LIST 不一致**，文档已过时。

### 档位选择

- `select_step_scale(index)`：clamp 到 [0,47]，新档粗算 tick 须在限制区间内（`:1463-1473`）。
- `auto_scale(since, until)`：`step_rough = |until-since| / 10`，找第一个粗算 tick < step_rough 的档的前一档（`:1448-1461`）。
- `set_axis_scale_step_limit` / `set_axis_time_range_limit`：外部可限制缩放/滚动范围（`:835-841`；`candlestick.py:157-159` 有用例）。

## 5. 缩放与平移

- **Ctrl+滚轮缩放**（`wheelEvent`，`:945-976`）：仅响应纵向滚轮；向上档位 +1（STEP_LIST 降序→更小刻度=放大），向下 -1。**锚定**：先记录鼠标像素对应 tick，换档后设 `scroll = a_to_b(该值) - 鼠标像素`、清空 offset——**鼠标指向的时间点保持不动**。档位到头静默不生效。
- **普通滚轮**：每格滚动 `pixel_per_scale / 4` 像素（1/4 主格）；向下=时间向未来（`:974`）。
- **左键拖拽**（`:978-1008`）：按下记点；移动时 `offset = 按下点 - 当前点`——内容跟随光标；松开时 `scroll += offset` 提交。拖拽中只 repaint 不更新 scroll，且实时提示被抑制。
- **键盘方向键**：设计为 100ms 定时器持续滚动（Up/Down ±pps/4，Left/Right ±整页）——**坏损**：`self.timer`（应为 `self.__timer`）与 `self.keys_pressed`（应为 `self.__timer_feature`）均不存在，且 `offset_scroll` 不 repaint（`main.py:367-392`；`viewer_ex.py:863-865`）。
- `jump_to_tick` 空 stub（`:860-861`）。

## 6. 横/纵切换与 Thread 宽度分配

- `set_axis_layout(HORIZON/VERTICAL)`（`:821-825`）；`set_axis_offset(0.0~1.0)` 设轴比例位置（越界不生效但仍 repaint，`:809-813`）。
- `update_thread_layout()`（`:1138-1211`）：`axis_mid = 横向总长 × offset`；刻度文字预留横向 30px、纵向 80px；轴区 = `axis_mid ± 15+10(+文字宽)`；横向模式 left/right 镜像翻转；**左右侧剩余宽度均分给该侧 Thread**（Thread 宽度不可单独设置）。
- 横向模式残留 TODO："Can we just rotate the QPaint axis?"（`:1178,1201`）。

## 7. 绘制流程（paintEvent，`:1220-1254`）

1. `check_update_paint()`（`:1054-1073`）：惰性更新流水线——layout_updated/scale_updated → `update_thread_layout()`；scale_updated/scroll_updated → `update_thread_scale()`；清 flag。
2. 背景：粉红整底。
3. 刻度（横向 `:1260-1312` / 纵向 `:1334-1386`，对称）：
   - 轴线横贯/纵贯；主刻度线 ±15px、副刻度线 ±5px；
   - 从 `estimate_closest_scale(tick_since)` 起画，`assert paint_tick <= tick_since`——**边缘 tick 可能触发 AssertionError 崩溃**（`:1275,1346`）；
   - 主刻度画线 + 居中文字；
   - **副刻度省略规则**：反复 `next_sub_scale`，当副刻度 ≥ 下一主刻度，或距下一主刻度不足半个副步长时停止——避免月末/年末副刻度与主刻度重叠（`:1298-1309,1373-1384`）。
4. `paint_threads`：逐个 Thread 先填背景色再画 item。
5. 实时提示（见 §8）。

## 8. 实时提示（悬停）

- 鼠标移动（非拖拽）时更新坐标/tick/悬停 item 并 repaint；提示关闭或拖拽中不画（`:1419,1513-1520`）。
- 绘制：**过鼠标的横竖十字线**（横贯/纵贯整个画布）+ 蓝色 `(36,169,225)` 提示框（`:1422-1444`）；提示框位于光标正上方一行高，右缘超出画布则整体左移到光标左侧。
- 文本 `format_real_time_tip`（`:1498-1509`）：`年/月/日`（月日补零）；悬停 item 且其 tip 非空时追加 ` | <item tip>`。
- item tip 规则（`HistoryIndexBar.get_tip_text`，`:207-234`）：取第一个 `abstract` tag strip；**单点**：`标题 : [年份]`；**持续**：`标题(当前第N年/总持续M年) : [起年 - 止年]`——N 随鼠标位置实时变化。
- `enable_real_time_tips(bool)` 全局开关（`:914-915`）。

## 9. 事件绘制细节（HistoryIndexBar，`:193-334`）

- **单点事件**（since==until）：带 10px 箭头的五边形指向时间轴；背景浅灰 `(243,244,246)`；6pt 字体；纵向像素范围以时间点为中心**向两侧各扩半个轨宽**（正方形点击区，`:242-253`）。
- **持续事件**：纯色矩形；背景青绿 `(185,227,217)`；8pt 字体。
- 文字：黑色，bar 矩形内 `AlignHCenter | AlignVCenter | TextWordWrap` 绘制第一个 abstract tag；**无省略号截断**，长标题显示不全（`:319-333`）。
- 命中测试：`axis_item_from_point` 遍历**全部** axis_items（非仅可见项）返回第一个包含点的 item（`:150-156`）。
- 注释掉的备选：菱形绘制（`:309-312`）、rotate(-90) 竖排文字（`:321-326`）。
