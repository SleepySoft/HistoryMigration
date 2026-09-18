# WHY · 渲染与布局的设计哲学

> 出处：`History/viewer_ex.py:344-353`（HistoryIndexTrack 类 docstring）、`viewer_utility.py:124-139`（AxisMetrics docstring）、`History/doc/Readme_Viewer_CN.md`、`README.md` Dev Note。

## 多 Thread 对照是软件的初心

`Readme_Viewer_CN.md:40`：

> 软件设计的最初的目的就是将不同历史在同一个时间线上进行对比。

Thread 宽度自动均分、轨宽可调，都是为了让对比布局**无需手工管理**（文档 `:50`）。这一初心直接来自原始需求 6（多 filter 同视图对照，见 [01-origin-and-requirements.md](01-origin-and-requirements.md)）。

## 布局稳定性优先于省计算

`HistoryIndexTrack` 的类 docstring（`viewer_ex.py:344-353`）是整套渲染设计的纲领，作者自称这是「历史 UI 最复杂的部分」。两条核心约束：

1. **持续事件（story）的像素宽度随时间轴缩放变化，单点事件（event）宽度恒定**（等于轨宽）——所以单点事件只放第一条 track，混排会破坏稳定性；
2. **动态布局（只布局可见 item）会导致来回滚动时布局不稳定**（同一 item 出现在不同列）——所以布局变更时对**所有 item 全量重排**，并用分步 flag（build_track → layout_track → layout_items → arrange_items）最小化重复计算。

长按续时长降序参与轨道分配也是同一哲学的体现（`:457-461` 注释保留了被否决的替代方案「开始时间越早优先级越高」及取舍说明：长优先布局更稳定，避免滚动时布局抖动）。

**一句话：用户看到的是「同一事件永远在同一列」，为此宁可多算。**

## 刻度体系：标准层级 + 配置化

- 刻度按「年→月→周→日→时」的人类习惯单位递进，是 README 显式 DONE 需求（`:173-175`：「main scale 为年时 sub scale 应为月（12 格），同理月之下为周，周之下为日，日之下为时」）。
- 48 档 `STEP_LIST` 是**配置而非算法**（`Readme_Viewer_CN.md:34`：「实际上这是可以调整的，在程序中通过 STEP_LIST 列表指定」）；跨数量级的「接缝」档被逐一注释掉，是手工调参的痕迹。
- `MAIN_SCALE_MIN_PIXEL=50`（一个主刻度最少 50px）是密度驱动的换档阈值。
- 刻度推进走精确日历运算（`offset_ad_second`，处理无 0 年）而非固定秒数——因为闰年、大小月使每个月/年的秒数不同。

## 刻度吸附与映射精度

作者在意 tick↔像素的映射精度：绘制刻度时记录「像素→精确 tick」表 `__optimise_pixel`，鼠标悬停取时间时优先命中精确刻度而非浮点反算（`viewer_ex.py:932-941`）；注释掉的旧代码里保留了「tick→pixel→tick 往返校验」的调试输出（`:1394-1398`）。

## transverse/longitudinal 几何抽象

`AxisMetrics`（`viewer_utility.py:122-258`）把矩形表示为「横向限度 + 纵向范围 + 时间刻度范围」，docstring 含 ASCII 图示：纵向→横向相当于「逆时针翻转」。设计意图是让横纵布局**共用同一套布局代码**，避免 `if layout == ...` 分支漫天。

但这个抽象**并不彻底**：横向实现仍残留镜像翻转特判和两条 TODO——「Can we just rotate the QPaint axis?」（`viewer_ex.py:1178,1201`）。作者后来在 UniversalHistory README 中承认（被划线的句子）：「借助 AI 我也轻松地实现了绘制时的坐标变换，从而解决了我一直耿耿于怀的横纵向坐标轴统一绘制的问题」——**统一逻辑坐标系 + QTransform 正是这个未遂意图的完成态**。

## 对迁移的启示

- 「布局稳定性优先」是渲染层的**第一遗产**，UniversalHistory 移植了 Track 分配策略（`migration_analysis.md` 决策表：「其 Track 分配策略已考虑稳定不闪烁，是核心资产」）。
- 「单点事件固定首轨」被用户反馈**有意放宽**：新布局让单点事件以虚拟像素区间参与统一分配（`migration_analysis.md` §8.5），但保留「长优先、稳定不闪烁」的内核。
- 「横纵共用代码」的意图在 UniversalHistory 中由统一逻辑坐标 + QTransform 真正实现。
- 密度驱动的刻度换档思想演化为 `TickStepper`（`UniversalHistory/docs/zoom_design.md`），48 档手工配置被标准层级金字塔取代。
