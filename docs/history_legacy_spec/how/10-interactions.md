# HOW · 交互总表

> 右键菜单、键盘行为在主窗口 `History/main.py`（**不在 viewer_ex.py**）；鼠标交互在 `viewer_ex.py`。

## 1. 右键菜单（main.py:396-513，挂在 TimeAxis 的 CustomContextMenu 上）

通过 `thread_from_point(pos)` 判断光标下是否有 Thread（`viewer_ex.py:923-930`）。**Record 上没有独立右键菜单**——命中测试只到 Thread 级，点在事件上弹出的仍是所在 Thread 的菜单。

### 空白处（无 Thread）

- **Add Thread**：新建 `HistoryIndexTrack`，背景色按 16 色调色板轮换（计数先 +1 再取模，第一个 Thread 用第 2 个颜色）；空 indexes；min track width=50。加在哪一侧由 `align_from_point` 决定（`viewer_ex.py:917-921`）：纵向模式点在轴中线左侧 → 左 Thread，右侧 → 右；横向模式点在轴线（从底部量 axis_mid）以下 → 左，以上 → 右。

### Thread 上（main.py:413-429）

1. **Load File**：从 depot 根目录选 `.his`（对话框过滤器文字误写为 "Filter Files (*.his)"），`History.load_source()` 载入核心，对返回 records 建**索引快照**灌入该 Thread（`:474-480`）。
2. **New Record**：取该 Thread 第一个 display source（无则空串）弹编辑器新建记录（`:482-490`）。
3. （分隔线）
4. ~~Load Index / Use Filter~~：**菜单项已被注释移除**（"Incomplete function, removed"），处理代码残留为死代码（`:419-421, 466-472, 497-510`）。
5. **Set Track Width**：`QInputDialog.getInt`（默认 50，**此处无范围钳制**；对比 ThreadEditor 中钳制 10-100，`:62-63`）——调的是**轨宽**而非 Thread 宽（`:459-462`）。
6. **Add Thread On Left / On Right**：在**当前 Thread** 的左/右侧插入新 Thread（`:437-450`；插入逻辑 `viewer_ex.py:882-894`）。
7. （分隔线）
8. **Remove This Thread**（`:454-455`）。

## 2. Record 级交互（viewer_ex.py）

- **双击事件条**（`:993-999,1014-1043`）：打开 `HistoryEditorDialog`（模态），`edit_source(index.source(), index.uuid())`，`show_browser(False)` 隐藏左侧浏览器；双击空白无反应。编辑器 Apply/Cancel 通过 agent 回调：Apply → 转发保存、关闭、repaint；Cancel → 关闭（`:1524-1540`）。
- **悬停**：实时十字线 + 蓝色提示框（见 [08-timeline-rendering.md](08-timeline-rendering.md) §8）。

## 3. 鼠标平移缩放（viewer_ex.py）

| 操作 | 行为 |
| --- | --- |
| 左键拖拽 | 内容跟随光标；松开提交 scroll；拖拽中抑制实时提示 |
| 滚轮 | 每格滚 1/4 主格像素；向下=向未来 |
| Ctrl+滚轮 | 缩放，锚定鼠标指向的时间点不动 |

## 4. 主窗口菜单与快捷键（main.py:227-297）

| 菜单 | 条目 | 快捷键 | 行为 |
| --- | --- | --- | --- |
| File | Load Files | Ctrl+F | 多选 `.his` 载入内存（depot 内路径转相对）；只载内存不显示 |
| File | Load Depot | Ctrl+D | **空实现 `pass`**（`:311-312`） |
| File | Load All Records | Ctrl+A | 枚举所有 depot 全量加载；**同步阻塞 UI 线程**（README:92 承认长时间不响应） |
| File | Exit | Ctrl+Q | 走 closeEvent 确认 |
| View | Historical Record Editor | Ctrl+R | 以共享 History 打开编辑器模态窗；**与时间轴零联动** |
| View | History Filter Editor | Ctrl+R（**快捷键冲突**） | FilterEditor 包在 WrapperQDialog 中打开 |
| View | Axis Appearance Setting | Ctrl+T | 横/纵 + 轴位置；**构造即 TypeError 崩溃**（见 [12-main-window.md](12-main-window.md)） |
| Help | Help | Ctrl+H | **无效果**（弹 readme 代码被注释） |
| Help | About | Ctrl+B | **无效果** |
| 编辑器 | Apply | Ctrl+S | 保存（要求修饰键精确等于 Control，`editor.py:296-298`） |
| 主窗口 | Ctrl+G | — | 仅打印日志（预留） |
| 主窗口 | ↑↓←→ | — | 设计为平滑滚动，**坏损不可用** |

## 5. 其他全局行为

- 关闭窗口弹确认框「是否确认退出？」（Close/Cancel，默认 Cancel）——**全应用唯一的中文 UI 文本**（`main.py:515-528`）。
- 窗口初始大小 = 屏幕 80% 居中（`main.py:225`；`ui_utility.py:54-79` 要求百分比 10-100 否则 raise）。
- README:55 已知问题：双击弹出的编辑器窗口可能跑到后台（疑 PyQt5 bug）。
