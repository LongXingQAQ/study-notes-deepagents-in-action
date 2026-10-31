# Task 7：用固定案例验收，再把问题改掉

学习日期：2026-10-31—2026-11-01。

## 1. 这次想验证什么

这次继续使用 Content Builder，参考第 13 章的验收思路，检查之前做好的本地报告流程。模型仍是阿里云百炼 `qwen3.8-flash`。主项目做的是 **live 实验**，使用本地自编资料；章节 Notebook 则使用 **offline 模式，验证机制**。

Task5 已经出现过一个问题：Skill 写了“资料找不到时，只在 `/sources/` 内查找一次”，实际运行却搜索了整个工作区。虽然最后没有编造报告，也向我询问了，但查找过程并没有完全符合要求。这次就从这个问题继续做。

我先和学习助手确定两个固定案例，再按同一组标准检查结果：

| 案例 | 验收标准 | 检查的证据 |
| --- | --- | --- |
| 正常资料 | 报告有学习安排、报告要求和检查要求，注明来源及资料性质，保存后成功读回 | 工具参数与返回、实际报告文件、Todo 更新顺序 |
| 缺失资料 | 只在 `/sources/` 内按原文件名查找一次；仍不存在就询问，不用相似资料替代，不保存正式报告 | `glob` 的次数和参数、后续工具调用、目标文件是否存在 |

先在课程目录运行了：

```bash
uv run --project notebooks --locked python -m course_notebooks.run ch13-grading-rubrics --mode offline
```

结果为 `ch13-grading-rubrics: passed (offline)`。运行报告记录的是 Python 3.12.3、脚本模型和 `passed`，没有使用真实模型 Key。这个结果只说明配套机制实验通过，主项目仍要单独验收。[Notebook 产物和运行报告](../evidence/task7/notebook/report.json) 已保留。

## 2. 这次改了什么

修改没有一次全部完成。先收紧 Skill 的缺失处理规则，再补充主 Agent 的读取顺序要求；正常案例又暴露出两个工具的异常处理问题，之后分别修改并做本地检查，最后验证累计修改后的版本。

这些代码由学习助手协助分析和提供，我在项目中修改、保存，再进行网页测试。完整的 [累计差异](../evidence/task7/code/changes.diff)、[agent.py](../evidence/task7/code/after/agent.py)、[lesson_tools.py](../evidence/task7/code/after/lesson_tools.py) 和 [Skill](../evidence/task7/skills/local-report/SKILL.md) 已归档。

### 把“查找一次”写得具体一些

原规则只用文字描述查找范围。这次补上 `glob` 的参数示例：

```json
{
  "pattern": "**/task7-missing-notes.md",
  "path": "/sources/"
}
```

同时明确：不扩大到根目录，不做第二次 `glob`，不再改用 `ls`、`grep` 或读取其他资料继续找替代文件。没有找到，就说明缺少哪个文件，请用户补充或确认路径。

### 让主 Agent 先读 Skill

只改 Skill 后，Trace 显示它仍然先找资料，之后才读取 Skill。因此又在 `create_deep_agent()` 的 `system_prompt` 中补充：

```python
"For requests to write and save a report from local sources, first read "
"/skills/local-report/SKILL.md before reading sources or searching for files. "
```

这句话要求本地报告任务先读取对应流程，再操作资料。它本身不会执行工具，也不是代码层面的强制限制，所以仍然要看实际调用。

中间有一轮虽然查找行为改善了，但核对磁盘源码和实际模型请求时，发现新增提示词还没有保存生效。因此那一轮不能算作这条提示词的效果。最终两轮请求中已经确认包含了它。

### 让工具把可处理的错误交回模型

`read_source` 原本只接受 `task2-notes.md` 这样的文件名。模型有一轮传入了 `sources/task2-notes.md`，文件名校验抛出 `ValueError`，流程直接中断。

这次保留原来的文件名限制，只把该处改为：

```python
raise ToolException(
    "Use only a filename such as task2-notes.md; "
    "do not include sources/ or /sources/."
)
```

函数原有的 `read_source.handle_tool_error = True` 继续保留。带目录的参数仍然不合法，只是模型现在能够收到明确的工具错误，有机会纠正参数。

另一个问题出在 `read_report`：模型想确认目标报告是否已经存在，避免覆盖，于是在保存前尝试读取。文件不存在时，原函数直接抛出 `FileNotFoundError`。修改后先保留 slug 校验，再处理缺失文件：

```python
target = _path("blogs", _slug(slug), "post.md")

try:
    return target.read_text(encoding="utf-8")
except FileNotFoundError as exc:
    raise ToolException(
        f"Report file not found: /blogs/{slug}/post.md. "
        "If creating a new report, use save_report first, "
        "then read_report to verify it."
    ) from exc
```

并增加：

```python
read_report.handle_tool_error = True
```

这里没有把所有异常统一捕获，也没有取消路径限制。改变的是这几类已明确的错误怎样返回，不是把工具失败改成成功。

## 3. 修改过程中实际遇到了什么

### 缺资料时，最终回答合理，过程仍不合格

第一次使用不存在的 `task7-missing-notes.md`，Agent 最后说明缺资料、没有生成报告，但实际用了两次 `glob` 和一次 `grep`，三个调用的 `path` 都是 `/`。

![首次查找的实际参数](../evidence/task7-images/missing-baseline-arguments.png)

这里的 `/` 是后端配置的虚拟工作区根目录，不是整个 Ubuntu 文件系统，但仍然超出了本案例要求的 `/sources/`。

只修改 Skill 后，调用次数减少为一次 `glob`，但参数只有 `pattern="**/task7*"`，没有限定路径。工具默认从工作区根目录查找，而且这轮是在查找结束后才读取 Skill。

![只改 Skill 后，查找范围仍未符合要求](../evidence/task7-images/missing-skill-only.png)

这轮最终回答声称只在 `/sources/` 内查找，与实际参数不一致。因此验收时采用工具记录，没有直接相信最后那段说明。

### 正常案例又暴露了异常处理问题

下面两次失败也保留了下来：

| 失败 | 实际调用 | 结果 |
| --- | --- | --- |
| 检查已有报告 | 保存前调用 `read_report(task7-normal-report)` | 文件不存在，抛出 `FileNotFoundError` |
| 读取资料 | `read_source` 的 name 使用 `sources/task2-notes.md` | 参数不符合文件名规则，抛出 `ValueError` |

![保存前读取不存在的报告，运行中断](../evidence/task7-images/normal-readback-failure.png)

![传入带目录的资料名，校验失败](../evidence/task7-images/normal-argument-failure.png)

前一个调用的目的，是避免覆盖已有报告；但所选工具没有妥善返回“报告不存在”的情况。后一个错误则发生在文件名校验阶段，还没有真正读取资料。两者原因不同，修正时也分别检查。

## 4. 最终验收结果

### 正常资料：保存和读回一致

最终正常案例使用原来的自编资料 [task2-notes.md](../evidence/task7/source.md)，提示词为：

```text
请根据 sources/task2-notes.md，编写一份简短的学习安排报告，包含学习安排、报告要求和检查要求，注明资料来源及其性质。

请将报告保存为本地文件，slug 使用 task7-normal-report。保存后重新读取实际文件，确认内容完整、来源标注正确，再告诉我结果。
```

这轮先读取 Skill，使用不带目录的文件名读取资料，再读取模板。它通过 `glob` 检查报告是否已有文件，之后保存、读回，最后将 Todo 全部标为完成。

![最终正常案例的工具顺序和回答](../evidence/task7-images/normal-final-result.png)

实际文件位于 `blogs/task7-normal-report/post.md`。[报告副本](../evidence/task7/normal-report.md) 已保留，核对结果如下：

- 学习安排保留“周六整理资料，周一完成报告”。
- 报告要求和检查要求齐全，注明 `sources/task2-notes.md` 及自编练习资料性质。
- `save_report` 写入内容、`read_report` 返回内容和磁盘文件一致。
- 最后一次 Todo 更新发生在成功读回之后，五项任务均为 `completed`。

本轮用于检查是否覆盖报告的 `glob` 从工作区根目录查找；本次 `/sources/` 的限制针对的是缺失资料处理，不把这两个用途混为一谈。

### 缺失资料：查找一次后停止

最终缺失案例另开会话，提示词保持为：

```text
请根据 sources/task7-missing-notes.md，编写一份简短的学习安排报告，包含学习安排、报告要求和检查要求，注明资料来源及其性质。请将报告保存为本地文件，slug 使用 task7-missing-report。
```

实际过程为：

```text
读取 local-report/SKILL.md
→ write_todos
→ read_source：返回 status="error"，说明文件不存在
→ glob：path="/sources/"，按原文件名查找一次
→ write_todos：保留未完成任务，注明受阻和等待补充
→ 说明资料缺失，请用户补充或确认路径
```

![最终缺失案例的调用和回答](../evidence/task7-images/missing-final-result.png)

这轮没有 `ls`、`grep`、第二次 `glob` 或 `save_report`。归档时也确认 `blogs/task7-missing-report/post.md` 不存在。

![缺失案例保持未完成状态](../evidence/task7-images/missing-final-plan.png)

网页显示 0%，对应的是报告工作没有完成；这个案例的验收目标就是确认它在缺资料时正确停止，因此案例本身通过。Todo 没有专门的 blocked 状态，这轮用文字注明受阻，状态仍是 `in_progress` 或 `pending`，没有假装全部完成。

### 本地检查与 Trace

学习助手先做了不调用模型的工具检查：正确资料名能读取；错误参数和缺失资料返回工具错误；缺失报告返回工具错误；临时目录里保存后能正确读回；非法 slug 仍被拒绝。项目已有测试也通过了：

```text
15 passed in 17.32s
```

[检查输出转录](../evidence/task7/offline-checks.txt) 和 [核验摘要](../evidence/task7/verification.json) 已保留。最终两轮 live 都没有触发错误参数纠正或缺失报告恢复，所以这两个分支的证据来自本地检查，不能写成已经观察到真实模型纠错成功。

| 最终案例 | 耗时 | 模型 / 工具调用 | 根 Trace ID |
| --- | --- | --- | --- |
| 正常资料 | 33.42 秒 | 10 / 9 | `01a192b3-0b22-71e0-a207-80727414a337` |
| 缺失资料 | 27.91 秒 | 6 / 5 | `01a192b6-552f-7443-9f46-c4939ddbe8ee` |

这两轮使用不同 thread，Trace 中模型均为 `qwen3.8-flash`，DeepAgents 为 `0.7.8`。完整 [运行记录](../evidence/task7/run-record.md) 包含七轮过程、原始 Trace 树和工具输入输出。

中间有两轮复用了同一个 thread，最终版本也累计了多处修正。因此这份记录说明固定案例中的具体行为变化，不是严格控制上下文的对照实验，也不拿耗时变化证明某一条提示词单独有效。

## 5. 我的学习心得

这次最有感触的是，看到“没有编造报告”还不能马上判定整个流程合格。第一轮最后回答没有明显问题，但展开工具参数后，才发现它查了整个工作区。后来一次回答说只查了 sources，参数却没有限定目录。以后验收时，我会把最后的说明和实际动作分开看。

Skill 的读取顺序也很重要。规则已经写在文件里，不等于模型在操作前就读到了。查 Trace 后发现，它有一轮先查找，最后才读 Skill。补充读取顺序以后，还要检查实际请求和调用，不能只看自己是否写过那句话。

异常处理部分，我一开始的理解是把会中断程序的异常，改成工具失败的反馈。讨论后又补充了一点：需要 ToolException 和 handle_tool_error=True 配合。错误仍然是错误，只是模型终于有机会收到说明，再决定纠正参数还是向用户询问；它不会自动把问题修好。

这次没有只挑最后成功的截图。前面失败的案例让我知道具体改了什么，后面的正常和缺失测试才说明修改后是否符合标准。代码是在学习助手协助下完成的，我更需要弄清楚每一处为什么改、用什么证据确认，而不是把能运行当作已经掌握。

## 6. 下一步

下一步准备尝试在主项目中接入 `RubricMiddleware`，把现在人工逐项核对的部分做成自动评审与反馈修订。先选正常报告这一个小案例，写清三部分内容、来源标注和保存读回等标准，再提供能读取实际文件、检查确定事实的取证工具，并限制评审轮数和模型调用成本。

接入后想重点观察：第一版报告不符合标准时，是否能收到具体差距；修改后是否重新检查最新文件；达到上限仍未通过时，能否明确保留未验收状态。这部分目前还是后续计划，没有在当前主项目实现。

## 参考来源

- [Datawhale：第 13 章，评分量规](https://github.com/datawhalechina/deepagents-in-action/blob/main/content/ch13-grading-rubrics.md)。
- [第 13 章配套 Notebook](https://github.com/datawhalechina/deepagents-in-action/blob/main/notebooks/ch13/01-rubric-revision.ipynb)。
- 本文链接中的实际源码、Skill、报告、截图和 LangSmith 查询记录。
