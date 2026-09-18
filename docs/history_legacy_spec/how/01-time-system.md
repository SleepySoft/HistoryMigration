# HOW · 时间系统

> 模块：`History/Utility/HistoryTime.py`（795 行，模块级函数，无类）。设计动机见 [../why/02-time-philosophy.md](../why/02-time-philosophy.md)。

## 1. TICK 定义

- `TICK = int`，单位**秒**。**公元元年（1 AD）1 月 1 日 0 点整 = TICK 0**；公元后为正至无穷，公元前为负至无穷（`HistoryTime.py:36`；README:51-53 确认「公元元年1月1日秒数为0，秒数为负则为公元前」）。
- **没有 0 年**：`year = -1` 表示公元前 1 年（1 BC）；`is_leap_year` 等处 `assert year != 0`（`:447`）。
- 公元前月日为「镜像参考」：公元前一日 = 公元前 1 年 12 月 31 日，公元前二日 = 12 月 30 日，依此从年末倒推（头注释 `:5`）。

## 2. 常量

- 单位（`:37-44`）：`TICK_SEC=1`、`TICK_MIN=60`、`TICK_HOUR=3600`、`TICK_DAY=86400`、`TICK_MONTH_AVG=2592000`（30 天平均月）、`TICK_YEAR=31536000`（365 天）、`TICK_LEAP_YEAR=31622400`（366 天）。
  - 缺陷：`TICK_WEEK = TICK(TICK_YEAR / 52)`——`int(31536000/52)=606461`，注释写的 608123.0769 是错的；该常量无人使用。
- 月份天数表（`:46-50`）：`MONTH_DAYS`/`MONTH_DAYS_LEAP_YEAR` 下标 1-12（0 位占位），平年 2 月 28、闰年 29；`MONTH_DAYS_SUM*` 为年内累计天数前缀和（平年末位 365、闰年 366）；派生 `MONTH_SEC*`（`:55-56`）。
- 周期常量（`:58-60`）：`DAYS_PER_4_YEARS=1461`、`DAYS_PER_100_YEARS=36524`、`DAYS_PER_400_YEARS=146097`，按 `365n + n//4 - n//100 + n//400` 推导（与 `Doc/datetime_algorithm.pdf` 一致）。
- `EFFECTIVE_TIME_DIGIT=10`（`:62`）定义了但从未使用。

## 3. 闰年规则

- `is_leap_year(year)`（`:440-449`）：格里高利规则 `(y%4==0 and y%100!=0) or y%400==0`，**对 year 取绝对值**——公元前年份镜像使用同一规则；断言 year≠0。
- `leap_year_count_since_ad(year)`（`:452-462`）：`|y|//4 - (|y|//4)//25 + (|y|//4)//100`。**全仓库无调用方**，遗留死代码。
- `year_ticks(leap)`/`year_days(leap)`（`:396-411`）：返回 366/365 天的秒数或天数。**注意 docstring 写反了**（"365 days if leap year" 应为 366），代码本身正确。
- `month_ticks(month, leap)`/`month_days(month, leap)`（`:414-434`）：返回**从年初到该月起点的累计**（查前缀和表），名字有歧义；断言 `0<=month<=13`（允许端点）。

## 4. tick ↔ 年月日转换

### 4.1 正向（日期→tick）

- `years_to_days(year)`（`:470-482`）：CE 计算 1..year-1 的天数 `365*(y-1)+(y-1)//4-(y-1)//100+(y-1)//400`；BCE 用 `|y|` 计算后取负（BCE 1 年 = -365 天）。
- `months_to_days(month, leap)`（`:485-494`）：年内前缀和。
- `date_to_days(y, m, d)`（`:497-512`）：CE 为 `年天数+月天数+d`；**BCE 为 `年天数+月天数+(d-1)`**——实现「公元前无 0 日」的连续映射（docstring 样例：-0001/12/31 → -1 天）。**不校验日溢出**（docstring 声明 "day can be larger than the day in month"）。
- `days_to_tick(days)`（`:579-586`）：`(days-1)*86400`，断言 days>0。
- `years_to_tick`（`:600-609`）对负年取绝对值再补符号；`date_to_seconds`（`:612-616`）、`time_data_to_tick`（`:619-627`）、总入口 `date_time_data_to_tick`（`:630-634`）——**无参数合法性校验**，时分秒超界直接线性相加。

### 4.2 反向（tick→日期）

- `tick_to_days(tick)`（`:640-647`）：**只接受非负 tick**（`assert tick>=0`），返回 `(tick//86400+1, 余秒)`，天数从 1 起。
- `days_to_years(days)`（`:518-542`）：400/100/4/1 四级分解；负数取绝对值分解后补符号；含 `remainder == DAYS_PER_4_YEARS-1` 特判（4 年周期最后一天属第 4 年）；返回的年、余日都从 1 起。
- `days_to_months(days, leap)`（`:545-557`）：前缀和表线性扫描。
- `days_to_date`（`:560-571`）：**BCE 分支将年内余日镜像** `remainder = year_days(leap) - (remainder-1)`——公元前日期从年末往年初倒数（对应「公元前一日=12月31日」规则）。
- `tick_to_years(tick)`（`:665-682`）：对 `abs(tick)` 分解；**BCE 特判**（`:679-682`）：0 秒划归 CE，BCE 需偏移 1 秒——余秒恰为 0 时年份 +1 修正，余秒取全年秒数−余秒。
- `tick_to_date`（`:685-694`）、`tick_to_time_data`（`:697-711`，取绝对值，BCE 时间分量不做镜像）、`tick_to_date_time_data`（`:714-728`）六元组输出。

### 4.3 datetime 桥接

- `now_tick()`（`:126-131`）。
- `tick_to_datetime`（`:212-225`）：超出 datetime 范围（year<1 或 >9999）时捕获异常 print 并返回 None——**BCE 的 tick 永远无法转成 datetime**。这直接决定编辑器 Calendar 按钮对公元前时间回退为当前系统时间的行为（见 [11-editor.md](11-editor.md)）。
- `datetime_to_tick` / `datetime_to_date_time_data`（`:228-241`）。

## 5. 时间偏移（刻度步进核心）

- `offset_ad_second(tick, offset)`（`:733-771`）：offset 为六元组（年，月，日，时，分，秒）。步骤：
  1. 日/时/分/秒按固定秒数加到 tick；
  2. 重新分解出年月日，归一化地加月偏移（月>0 与月<=0 两条分支，`:750-756` 注释给出 -11/-12/-13 月换算示例）；
  3. **加年偏移时处理「无 0 年」**（`:758-766`）：CE 跨零到 BCE 额外 -1（year=1 加 -1 → -1），反之 +1；
  4. 日数钳到当月天数（`day=min(day, mdays)`）；
  5. 时分秒从余秒重新拼回。
  - **实际使用方**：时间轴主/副刻度步进（`viewer_ex.py:601,604`）。
- `offset_date_time(origin, offset)`（`:774-795`）：六元组逐位相加后秒→分→时→日进位（注意负值的地板除语义），再过 `date_to_days→days_to_date` 归一化。**无「无 0 年」修正**，跨零年会断言失败。全仓库无调用方，遗留。

## 6. 格式化输出

- `format_tick(tick, show_date=False, show_time=False)`（`:184-202`）：默认**只输出年份绝对值**——BCE 不输出「公元前/BC」字样（`:194-197`）；show_date 追加 `/m/d`；show_time 追加 `HH:MM:SS`（**月日与时间之间无空格**，`:199-201`）。调用方：编辑器记录下拉框前缀、时间悬浮提示。
- `format_datetime(dt, show_date=True, show_time=True)`（`:205-209`）：输出 `[YYYY-mm-dd HH:MM:SS]` 带方括号——正好符合解析器的 `[]` 保护壳约定，供日期拾取器回填。缺陷：`:206` 三元表达式两分支相同，show_date 参数无效；show_date=False 且 show_time=False 时不加方括号。

## 7. 遗留

- 旧浮点年份时代的实现（`time_text_to_history_times`、`standardize`、`parse_single_time_str`、`standard_time_to_str`、`tick_to_cn_date_text`）整段注释保留（`:319-386`）；其中 `tick_to_cn_date_text` 的公元分支误写 `str(-year)`。
- `recycled/history_time.py` 是被替换的旧版本存档。
