# AGENTS.md

本文件为在本仓库工作的编码 Agent 提供协作规则。开始任何修改前，先确认当前工作区及其 submodule 已初始化。

## 仓库范围

- 根仓库只保存 submodule 指针、根级文档和迁移分析；不要在根仓库直接实现业务代码。
- `History/` 是旧版参考实现和 `.his` 解析器来源，除非需要修复迁移必需的解析问题，否则不要重构它。
- `UniversalHistory/` 是主要开发目标。新桌面功能默认使用 Python 3.10+ 和 PyQt6。
- 如果修改 submodule 内的文件，注意这些变更属于 submodule 自己的 Git 历史；不要在未经说明的情况下混合提交根指针和业务变更。

## 项目分层

| 层 | 位置 | 规则 |
| --- | --- | --- |
| 时间 | `universal_history/chrono/` | 以 `JDNTimestamp` 的定点整数为准；不引入浮点时间运算。 |
| 数据 | `universal_history/models/` | 使用 `Event`、`EventIndex` 和 `Workspace`；UI 通过信号响应数据变更。 |
| 格式 | `universal_history/adapters/` | 外部格式只通过 Adapter 进入模型；不要让模型或 UI 直接解析文件。 |
| 渲染 | `universal_history/render/` | 只负责几何、布局和绘制；不直接读写业务数据。 |
| UI | `universal_history/ui/` 和 `main_window.py` | 组装用户流程；复杂计算放到模型、Adapter 或渲染层。 |

## 常用命令

从仓库根目录初始化：

```bash
git submodule update --init --recursive
```

在 `UniversalHistory/` 中创建环境：

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m pip install requests
```

启动应用：

```bash
python -m universal_history
```

运行全部测试：

```bash
python -m unittest discover -s tests -v
```

运行分组测试：

```bash
python -m unittest discover -s tests/render_tests -v
python -m unittest discover -s tests/ui_tests -v
```

无显示环境可尝试：

```bash
QT_QPA_PLATFORM=offscreen python -m unittest discover -s tests -v
```

仓库未配置统一 formatter。不要为了格式化而重排无关代码。

## 编码约定

- 保持现有代码风格：类型提示、`pathlib.Path`、`typing.Optional` 与 Python 3.10+ 兼容写法。
- 优先使用 `from __future__ import annotations`，使模型和接口定义保持清晰。
- PyQt6 只出现在 UI 与渲染实现中；数据模型不要依赖 `QWidget` 或文件系统路径以外的 UI 对象。
- `Workspace` 是内存模型；持久化由 Adapter 完成。不要让 `Workspace` 直接实现 `.his` 序列化。
- 修改 `.his` 解析或写出时，先用 `History/depot/example/example.his` 验证向后兼容。
- 时间边界使用天文纪年：`0` 年表示 1 BC，`-1` 年表示 2 BC。迁移旧 `History` 年份时调用 `history_year_to_jdn_year()`。
- UI 用户可见文本目前多为英文；不要在同一功能中突然混入中文，除非项目已有对应上下文。

## 测试要求

- 修改 `chrono/`、`models/`、`adapters/` 时，至少运行对应的 unittest 模块或分组。
- 修改布局、缩放、横纵切换或主窗口交互时，运行 `tests/render_tests` 和 `tests/ui_tests`。
- 修复缺陷时优先补充一个能失败的回归测试。
- 如果当前环境缺少 PyQt6、`lunar_python` 或 `requests`，先说明测试因依赖缺失未执行；不要把未运行的结果描述为通过。

## 文档要求

- 更新架构、阶段状态、运行方式或已知限制时，同步更新根目录 `README.md` 和 `docs/PROJECT_STATUS.md`。
- 如果某个决定来自历史分析，优先链接 `migration_analysis.md`，不要复制大段内容造成文档漂移。
- 文档中的命令必须与 `pyproject.toml`、`requirements.txt` 和现有入口一致。

## 修改检查清单

1. 确认变更落在正确层，没有让 Adapter、模型和 UI 互相反向依赖。
2. 保持 `.his` 文件兼容，避免静默改写用户数据。
3. 运行相关测试；涉及 UI 时至少确认包可导入且主窗口能创建。
4. 更新受影响的 README、状态文档或设计文档。
5. 不要自动 commit；除非用户明确要求。
