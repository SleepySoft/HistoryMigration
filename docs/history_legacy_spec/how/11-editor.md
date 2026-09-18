# HOW · 编辑器

> 模块：`History/editor.py`（857 行）：`HistoryRecordEditor`（单条记录编辑）、`HistoryRecordBrowser`（depot/文件浏览）、`HistoryEditorDialog`（组合对话框）。辅助：`Utility/DateTimePicker.py`、`Utility/ui_utility.py`。

## 1. 数据持有语义（重要设计约束）

- 类 docstring（`editor.py:16-21`，Update 20250205）：**编辑器只保存「正在编辑的记录」和「它所属的 source」**，完整记录列表必须从 `History` 取——单一数据源，避免多份列表不同步。成员只有 `__source` 和 `__current_record`。
- 通过 **Agent 回调**（`on_apply`/`on_cancel`）把落盘委托给外部，编辑器本身不写文件。

## 2. 界面结构（init_ui，editor.py:93-177）

1. 顶行：source 路径显示（拉伸 1）+ `Open File` + `New File`。
2. 次行：记录下拉框（拉伸 1）+ `New Event` + `Del Event`。
3. 中部 QTabWidget：
   - **Tab「Event Editor」**：
     - 第 0 行：`Event ID` 标签 + uuid（只读 QLabel）+ `Save and New` 按钮；
     - 五要素行，每行 = **QRadioButton（行标签兼倾向选择）+ 输入框 + Auto Detect 按钮 + Lock 复选框**：
       - Time 行另有 `Calendar` 按钮；
       - Event 行只有 radio，无输入框（输入框是下方三个大框），**默认勾选**；
     - `Event Tags` 行**已被注释隐藏**，但控件仍存在并参与数据流（缺陷，见 §6）；
     - GroupBox 内：`Event Title`（QLineEdit）、`Event Brief`（QTextEdit 拉伸 2）、`Event Description`（QTextEdit 拉伸 5），均纯文本。
   - **Tab「Label Tag Editor」**：只有一个 `EasyQTableWidget` **空表**——无行列设置、无任何读写代码。纯占位（对应 README:133 的 Front/Back 卡片计划）。
4. 底部 `Apply` + `Cancel`。
5. 构造末尾自动 `create_new_file()`——一打开就处于「无 source + 一条全新记录」状态。

## 3. 记录下拉框（editor.py:202-244, 395-421）

- 数据源：当前 source 的记录经 `History.sort_records()` 按 since 升序。
- 显示文本：`'[' + format_tick(record.since()) + '] ' + record.uuid()`——`format_tick` 默认只输出年份，形如 `[1949] xxxx-uuid`。
- 当前记录不在列表中（新建未保存）时**追加到尾部**。
- 选中逻辑 workaround：index==0 时手动调用 `on_combo_records()`（注释解释 Qt 信号在添加首项时数据未准备好）。
- 找不到记录仅 print 日志，无 UI 提示。
- **切换记录时不提示保存未提交的修改**，未 Apply 的编辑直接丢失。

## 4. 时间输入（ShadowLineEdit + Calendar）

- 时间框是 `ShadowLineEdit`（`ui_utility.py:446-474`）：`textChanged` 即解析当前文本，把标准化结果用 **QToolTip 悬浮显示**在输入框上方（空间不足显示在下方 10px，SansSerif 10）；点击输入框也重新弹出。
- 悬示格式：`format_tick(t, True)` → `年/月/日`（公元前显示为正数年份，**无 BC 标记**）。
- **Calendar 按钮**（`editor.py:314-322`）：当前文本第一个 tick 转 datetime（BCE/超范围 → None）弹 `DateTimePicker`；确认后回填 `[yyyy-mm-dd HH:MM:SS]`（带方括号，正好符合解析器 `[]` 约定）。
- `DateTimePicker`（`DateTimePicker.py:8-44`）：`QDateTimeEdit`，格式 `yyyy-MM-dd HH:mm:ss`，calendarPopup；传入 None 默认**当前系统时间**。`__main__` 演示代码解包数量错误（缺陷）。

## 5. 倾向（focus）与必填校验（ui_to_record，editor.py:453-510）

- 五个互斥 QRadioButton：Time / Location / Participant / Organization / Event，默认 Event；对应 focus_label `'time'/'location'/'people'/'organization'/'event'`。
- **Time 永远必填**：为空弹框 "Time field is required." 并中止（`:473-475`）。
- focus 必填规则（`:477-496`）：focus=location/people/organization → 对应字段非空；focus=event → title/brief/event **至少一个**非空；失败弹框提示。
- **缺陷**：focus=time 分支漏设 `input_valid`（`:477-478`）——**选 Time 倾向永远无法 Apply**。因默认 focus 是 event，不易触发。
- focus_label 写入记录并决定 .his 里记录的结束标记。

## 6. 字段映射细节（ui_to_record 续）

- time/location/people/organization/tags 按英文逗号 split 后 `set_label_tags`（每个 tag strip）；time 标签立即解析为 ticks 取 min/max。
- title/brief/event 直接写入同名 label。
- **隐藏的 `__line_default_tags` 仍参与写入**：空串 split 产生空 tag（`:458,502`）。
- **Apply 用全新 HistoryRecord 重建记录**（只复制 uuid）——旧记录上 UI 未暴露的 label（author、自定义 tags 等）**在 Apply 后全部丢失**。迁移时必须决定保留还是修复此行为。

## 7. 锁定（Lock）语义（clear_ui，editor.py:425-451）

- 五要素行和（隐藏的）tags 行各有 Lock 复选框。
- 语义：**`clear_ui()` 时不清空被锁定行**——便于连续录入同一地点/人物的多条记录（README Dev Note 确认为有意设计）。
- clear_ui 无论锁定都清空：uuid、source、title/brief/event（后两者还重置字体为微软雅黑 10pt 黑字白底）。
- **回填不还原 focus radio**：`record_to_ui` 不读 `record.get_focus_label()`，重新编辑旧记录时倾向保持 UI 当前选中项。
- 锁定对回填不生效（回填无条件 setText）；只在「新建记录」路径上有保值效果。

## 8. 新建/打开/删除/保存

- **New Event** → `create_new_record()`（`:528-537`）：新建记录（自动 uuid）→ clear_ui（尊重 Lock）→ 刷新下拉框（新记录追加尾部并选中）。开头 `if self.__current_record is not None: pass  # TODO`——**未保存修改不提示直接丢弃**。
- **New File** → `create_new_file()`（`:539-541`）：source 置空 + 新建记录（**记录被创建两次**，第二次 uuid 生效）；source 标签显示 "No Source - Will ask for a source when saving record."。
- **Open File**（`:339-348`）：文件对话框（depot 根，`*.his`）；选中后 `edit_source(fname, 'xxx')`——`'xxx'` 是故意不存在的 uuid 使当前记录落空 → 自动选中第一条；`__current_depot` 置空。
- **edit_source(source, uuid)**（`:251-279`）：source 未加载先 `load_source`；uuid 找到则选中，找不到/为空则新建记录。
- **Del Event**（`:353-361`）：`history.remove_record(uuid)`（**跨所有 source** 按 uuid 删）→ 置 None → 刷新 → 通知 agent `on_apply()`——**删除后立即落盘，无确认对话框**。
- **Save and New** = Apply 后立即 New（只放在 Event ID 行右侧，位置不直观）。
- **Cancel** → 仅通知 `on_cancel()`（对话框直接 close）；**无未保存提示**。
- **快捷键 Ctrl+S = Apply**（`:296-298`）。
- **Auto Detect**（`:308,324-334`）：四个按钮全部弹 "Not implemented"，文案 `'In feature, we will use NLP or LLM to recognize the %s information from main text'`（"In feature" 为笔误）——NLP/LLM 抽取五要素的占位。

## 9. Apply 保存逻辑（editor.py:363-389）

1. `__source` 无效（空或 `INVALID_SOURCE`）→ 弹 `getSaveFileName`（depot 根，`*.his`）；取消则整个 Apply 中止——**「新建到哪个文件」在保存时才问**；
2. 新建 HistoryRecord，`copy_uuid_from`（保持 uuid 实现更新而非新增），`set_source`；
3. `ui_to_record()` 校验填充，失败中止（仅打印日志）；
4. `history.upsert_records(source, new_record)`：先按 uuid 全局删旧再追加尾部；
5. 通知所有 agent `on_apply()` → **实际写盘在 agent 处**；
6. 刷新下拉框。

## 10. HistoryRecordBrowser（editor.py:582-741）

- depot 下拉框 + 文件 QListWidget（最小宽 200）+ `Rename` 按钮。
- depot 枚举：depot root 下所有子目录名；**只在构造时枚举一次**，运行期新增 depot 不出现；填充时防递归信号，填充后手动选中第 0 项并手动触发回调。
- 文件列表：`os.walk` 递归枚举 depot 下所有 `.his`；重建列表时若当前文件仍在则恢复选中。
- 切换 depot → 通知 agent `on_select_depot`；选中文件（相同则忽略）→ 通知 agent `on_select_record(完整路径)`。
- **Rename**（`:672-690`）：无选中直接返回；弹输入框；不以 `.his` 结尾自动补后缀；`os.rename` 同目录改名；成功/失败均弹提示框；成功后刷新。

## 11. HistoryEditorDialog（editor.py:746-821）

- 横向 QSplitter：左 Browser（拉伸 3）、右 Editor（拉伸 7）；标题 "History Record Editor"；屏幕 80% 居中；加最小/最大化按钮。
- 外部不传 agent 时对话框自身同时充当 Editor 和 Browser 的 agent。
- **on_apply（真正落盘）**：`history.save_source(source)` 整文件覆写（utf-8）→ 弹消息框 "Save to {source} successful./fail: ..." → 刷新 Browser。**每次 Apply/Del 都弹此框**——连续录入体验差。
- `on_cancel` → close；`on_select_depot` 同步 depot 给 editor；`on_select_record` → `editor.edit_source(路径, 'xxx')`。
- `show_browser(False)` 供双击事件时隐藏浏览器。

## 12. ui_utility.py 辅助组件

- `restore_text_editor`（`:41-50`）：清空+聚焦+微软雅黑 10pt+黑字白底（clear_ui 时对 brief/event 调用）。
- `resize_widget_to_screen_percentage`（`:54-79`）：百分比 10-100 否则 raise；按可用屏幕缩放居中。
- `InfoDialog`（`:86-103`）：只读信息框（原用于 Help，现已不被调用）。
- `CommonMainWindow`（`:110-268`）：通用主窗脚手架——**HistoryUi 并未继承它**，遗留代码。
- `WrapperQDialog`（`:275-323`）：把任意 QWidget 包成模态对话框；OK 置 `is_ok=True` 后关闭，Cancel 直接关闭（is_ok 区分确认/取消）。
- `EasyQTableWidget` / `EasyQListSuite`（`:330-439`）：表格/列表辅助；后者当前无人使用。
