# HOW · 主窗口

> 模块：`History/main.py`（593 行）：`HistoryUi`（主窗口）、`ThreadEditor`（单条 Thread 配置行）、`AppearanceEditor`（外观设置面板）。菜单与快捷键总表见 [10-interactions.md](10-interactions.md) §4。

## 1. HistoryUi 构造（main.py:192-225）

- 持有唯一的 `History` 实例（全应用共享）；`TimeAxis` 为中央控件并 `set_agent(self)` / `set_history_core(history)`。
- 标题 "SleepySoft/History - Sleepy"；状态栏 "Ready"；窗口为屏幕 80% 居中；全屏/固定尺寸代码被注释。
- `QTimer` + `__timer_feature` 集合用于方向键连续滚动（坏损，见 [10-interactions.md](10-interactions.md)）。
- `on_menu_selected`（`:358-363`，dock 显隐切换）是从 CommonMainWindow 抄来的死代码——本窗口没有任何 dock。

## 2. Axis Appearance Setting（main.py:85-187, 328-335）

- **AppearanceEditor**：`Appearance` 组 = Layout 单选（Horizon/Vertical，默认 Vertical）+ Position 滑条（0-100，默认 50，右侧 QLabel 实时同步）；窗口固定 600×400。
- `set_appearance_config()/get_appearance_config()` 读写 `{'layout', 'offset', 'thread'}`；offset = label 文本 / 100。
- 菜单路径：构造时读当前轴 layout/offset 预填，包进带 OK/Cancel 的 `WrapperQDialog`；`is_ok()` 时应用到 TimeAxis。
- **构造即崩溃**：`__group_thread_editor` 被置 None、其创建代码被注释（`:100,121-122`），但 `main_layout.addWidget(self.__group_thread_editor)` 仍执行（`:127`）——`addWidget(None)` 抛 TypeError。**Ctrl+T 菜单在实际运行中打不开对话框**，横纵切换/轴位置功能无法到达。
- 即使修复构造，`append_thread`/`layout_thread_editor` 还依赖为 None 的 `__layout_thread_editor`（`:181-187`）；`apply_appearance_config` 中重建 Thread 的大段代码被注释（`:544-559`），实际只应用 layout 和 offset。

## 3. ThreadEditor（单个 Thread 的配置行，main.py:24-80）

- `Thread Index` 输入框 + `Browse`（选 `.index` 文件）；`Thread Layout` Left/Right 单选（默认 Right）；`Track Width` 输入框（默认 '50'）；`Remove` 按钮回调 agent。
- `get_thread_config()`：宽度转 int **钳制 [10,100]**，解析失败回退 50（`:58-69`）——与右键菜单 Set Track Width 的无钳制不一致。
- 当前只被 AppearanceEditor 使用 → 同样不可到达。
- README:166-170 的 DONE 记录（"界面支持配置1-10个thread，包括enable，focus label，depot，filter"）描述的是历史曾实现、当前已残破的功能——**迁移时应视为「设计意图存在、参考实现不完整」**，由新实现补全而非照搬。

## 4. 退出确认（closeEvent，main.py:515-528）

点窗口 X 或 Exit 时弹 `QMessageBox.question`：标题「退出」、正文「是否确认退出？」（**全应用唯一的中文 UI 文本**，与全英文界面不一致），按钮 Close/Cancel，默认 Cancel；Close 才接受关闭。

## 5. 入口

`main.py:564-569`：QApplication + HistoryUi 显示。`viewer_ex.py:1570` 的独立 `main()` 加载硬编码 `depot/history.index`（该文件未必存在），独立运行 viewer_ex.py 基本不可用；`HistoryViewerDialog`（`viewer_ex.py:1547-1561`）仅被该废弃入口使用。
