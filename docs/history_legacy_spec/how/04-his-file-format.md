# HOW · .his 文件格式

> 模块：`History/core.py` 的 `TokenParser` / `LabelTagParser` / `HistoryRecordLoader.from_text`。WHY（为什么纯文本）见 [../why/03-data-philosophy.md](../why/03-data-philosophy.md)。

## 1. 词法（core.py:175-177）

```python
LABEL_TAG_TOKENS = [':', ',', ';', '#', '"""', '\n', ' ']
LABEL_TAG_WRAPPERS = [('"""', '"""'), ('#', '\n')]
LABEL_TAG_ESCAPES_SYMBOLS = ['\\']
```

`TokenParser`（`:18-170`）细节：token 与前导文本交替产出（遇 token 先回吐已积累文本，下次调用才返回 token 本身）；空格被当作 space token 跳过不作为产出（只有 `' '` 是，`\n` 不是）；`config` 会把 wrapper 起止符号并入 tokens；`offset_until` 找不到目标时指针推到末尾返回 False。测试 `test_token_parser_case_normal`（`:1200-1224`）固化了词法行为，可直接用作迁移验收用例；`test_token_parser_case_escape_symbol` 是空 pass——转义路径无测试覆盖。

## 2. 语法规则

- **记录开始**：每条记录以 `[START]: focus_label` 开头（`core.py:849-857`）。`[START]` 是普通 label（方括号无特殊词法含义，只是约定）；其第一个 tag 即 focus label。
- **LabelTag 行**：`label: tag1, tag2, tag3`。`:` 分隔 label 与 tags；`,` 分隔多个 tag；`;` 或换行 `\n` 结束当前 label 行（`:233-234`）。
- **记录结束（隐式）**：当某行的 label 等于该记录的 focus label 时，该记录在该行关闭（`:865-868`）。dump 时 focus label 被强制移到最后（`:574-577`），缺失或为空补写 `focus_label: end`。规范形态：

  ```
  [START]: event
  uuid: xxx
  ... 其他 label ...
  event: 内容或 end
  ```

- **多行文本 wrapper**：`"""..."""` 包裹的文本原样保留，内部不解析 token、不分割。实例见 `example.his:12-30`。
- **注释**：`#` 到行尾 `\n`（wrapper `('#','\n')`），解析时丢弃其间所有 token（`:228-229`）。
- **转义**：wrapper close 符号前若紧邻 `\`，则 close 不生效、继续留在 wrapper 内（`:98-101, 166-170`）；写出时 tag 内含 `"""` 替换为 `\"""`（`check_wrap_tag`，`:283`）。**反斜杠自身不转义**（`:284` 被注释）——含 `\` 的 tag 写出后读回语义可能变化。
- **写出侧自动包裹**（`check_wrap_tag`，`:280-288`）：tag 含任一 token 字符（`:` `,` `;` `#` `"""` 换行 **空格**）时整体用 `"""` 包裹——注意空格也触发包裹。
- **tag 去重**：同一 label 行内重复 tag 丢弃（`append_tag`，`:251-252`）。

## 3. 实例对照（History/depot/example/example.his，6 条记录）

1. `[START]: event`（行 1-30）：完整记录——uuid、`time: BC3000`、`location: Shanghai, Beijing`（多 tag）、people、organization、`tags: example, readme`、`author: Sleepy`、title/brief/event 三个 `"""` 多行块。
2. `[START]:people`（行 34-47）：focus 为 people；`people: Mike` 出现在 event 块**之后**——因为 focus 是 people，记录直到 people 行才关闭。
3. `[START]: event`（行 51-62）：`tags: tag2, tag3, tag4, tag5, even`、`author: SleepySoft`。
4. `[START]: location`（行 66-78）：focus 为 location，`location: Guangxi` 收尾。
5. `[START]: event`（行 82-93）。
6. `[START]: event`（行 97-107）。

真实数据（`depot/China_CN/三国时期事件.his`）展示更多方言形态：`title:`/`brief:` 单行不包裹也合法；`event: end` 空事件收尾；`time: 184年2月-184年8月` 中文自然语言时间段；记录之间无空行直接相连。

## 4. 解析容错行为（LabelTagParser.parse，core.py:201-244；from_text，:836-871）

状态机：`next_step ∈ {'label','tag'}`、`expect`（期望 token 列表）、`until`（注释跳过模式）：

1. `until != ''`（注释中）：丢弃一切直到 `\n`。
2. 当前 token 不在期望列表 → 置 `ret=False` 但**继续解析**（容错）。
3. `#` → 进入注释模式；`:` `,` `"""` → 直接丢弃；`\n`/`;` → 下一词按 label 处理；label 状态的词 → `switch_label` 新建条目；tag 状态的词 → `append_tag` 追加（去重）。
4. **返回值 `ret` 被 `from_text` 忽略**，error_list 也被丢弃（`:838,841,855`）——**解析错误完全静默**。

`from_text` 装配规则：

- 遇 `[START]` 关闭并收纳上一条未完成记录、重置 focus（tags 为空仅记 error 不中断）；
- 普通 label 时若 record 为 None 才惰性创建（source 经 `normalize_source` 归一化）；
- label==focus 时立即关闭记录；文件末尾残留记录兜底收纳；
- 连续两个 `[START]` 之间的空 section 不产生记录；
- **focus 行之后、下一个 `[START]` 之前的游离 label 会开启一条 focus='' 的无头记录**；
- **返回值 `{原始source: [records]}`——dict 键用原始 source，记录内部 source 是归一化相对路径，两者不一致**。

## 5. 兼容要求（迁移红线）

- 修改 `.his` 解析或写出时，先用 `History/depot/example/example.his` 验证向后兼容（项目 AGENTS.md 约定）。
- 写出不得静默改写用户数据：`"""` 包裹、转义、focus 压轴、`event: end` 补齐等规则需保持一致。
