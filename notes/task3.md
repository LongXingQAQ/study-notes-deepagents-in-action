# Task 3：计划写出来以后，事情真的做完了吗？

学习日期：2026-10-20。

## 1. 这次验证什么

这次学习第 4 章，用已有的 `deepagents/content-builder` 项目练习任务规划。模型仍是阿里云百炼北京地域的 `qwen3.8-flash`，使用本地固定资料，关闭联网搜索和图片生成。

读完教程后，我知道 `write_todos` 能管理任务清单，但还不确定自己是否真正理解。于是先做了一次正常的读取、整理、保存、核验任务，再故意要求读取一个不存在的资料文件，观察 Agent 怎样修改计划、怎样说明未完成的工作。两轮都是 live 实验。

## 2. 我改了什么，哪些配置是原有的

项目的 `agent.py` 已经有下面的导入：

```python
from langchain.agents.middleware import TodoListMiddleware
```

`build_graph()` 中的 `create_deep_agent()` 也已经显式传入了 `TodoListMiddleware()`。下面是配置节选，其他参数省略：

```python
middleware=[
    TodoListMiddleware(),
    MemoryMiddleware(...),
],
```

这是模板已有的配置，不是我这次新增的 Python 代码。它提供 `write_todos` 工具、独立的 `todos` 状态和规划指导提示词；具体怎样拆任务、什么时候更新状态，仍取决于模型的行为。

本次主要改的是发给 Agent 的任务提示词：从普通的文件读写要求，改成明确要求先规划、按实际结果更新清单、读回核验后才能标记完成。第二轮进一步引入资料缺失条件，让 Agent 自己拆解任务。模型、工具实现和中间件配置保持原样。

两轮任务要求有所不同，不能把它们当作严格的单变量 A/B 测试，也不能据此声称 Todo 提高了多少完成率。这次记录的是规划机制和两种具体场景下的实际行为。

## 3. 正常资料：观察四次清单更新

固定输入是 [task2-notes.md 的副本](../evidence/task3/source.md)，内容沿用 Task 2 的自编练习资料：周六整理资料、周一完成报告，先写初稿再修改，每次保存或修改后重新读取确认。

我要求 Agent 建立四项任务：读取资料、整理三个部分、保存报告、读回核验。初始第一项为 `in_progress`，其余为 `pending`。报告保存的 slug 为 `task3-plan`，内容要保留学习安排、报告要求、检查要求，以及资料来源。

实际运行中调用了四次 `write_todos`。根据截图和工具输入输出，清单变化如下：

| 更新次数 | 读取资料 | 整理内容 | 保存报告 | 读回核验 |
| --- | --- | --- | --- | --- |
| 第 1 次 | `in_progress` | `pending` | `pending` | `pending` |
| 第 2 次 | `completed` | `in_progress` | `pending` | `pending` |
| 第 3 次 | `completed` | `completed` | `completed` | `in_progress` |
| 第 4 次 | `completed` | `completed` | `completed` | `completed` |

![前两次 Todo 更新：读取完成后，开始整理内容](../evidence/task3-images/normal-todos-01-02.png)

![后两次 Todo 更新：保存完成后核验，读回后全部完成](../evidence/task3-images/normal-todos-03-04.png)

我注意到，保存任务直接从 `pending` 变成了 `completed`，没有单独记录 `in_progress`。当时我猜是不是保存很快，所以系统自动省略了。后来才分清：清单是模型主动写入的状态，不是工具耗时的自动记录。

查询工具 Run 的时间后，可以确认顺序是：`save_report` 返回之后，模型才在第 3 次更新中标记保存完成；`read_report` 返回之后，才在第 4 次更新中标记核验完成。保存这一步的进度记录不够细，但完成标记有实际执行依据。

实际文件保存在 `/home/lx/projects/content_builder_deepagent/blogs/task3-plan/post.md`。[报告副本](../evidence/task3/normal-report.md) 包含三个要求的部分、周一的安排、来源路径和自编练习资料说明。

## 4. 缺少资料：它承认受阻，却改了交付内容

第二轮要求结合学习要求和可用时间制定报告。我先确认 `sources/task3-time-budget.md` 不存在，再发送请求。关键约束如下：

```text
请使用 write_todos 自行拆解任务，并根据实际执行结果更新状态。
报告必须依据 sources/task2-notes.md 和 sources/task3-time-budget.md，
不能自行编造可用时间。
只有成功读取两份资料后，才能生成报告，用 save_report 保存，
slug 为 task3-missing，再读回核验。
如果资料不足，请说明缺少什么，不能声称整个任务已完成。
```

同时限制只在 `sources/` 中查找，不修改原资料、不联网、不生成图片、不委派子 Agent。

![实验开始前，sources 中没有时间预算文件](../evidence/task3-images/missing-source-directory.png)

这轮 Agent 自己拆出了六项任务：定位目录、读取学习要求、读取时间预算、制定报告、保存、读回核验。随后通过 `glob`、`ls` 查看文件，读取已有资料，并使用两次 `grep` 查找“可用时间”和文件名关键词，结果均无匹配。

![目录查找、资料读取和缺失结论](../evidence/task3-images/missing-overview.png)

![两次关键词检索没有匹配，随后读取 planning.md](../evidence/task3-images/missing-search.png)

我最初看界面时，以为它把目录里的文件全读了一遍。后来核对原始工具参数，实际读取的是 `task2-notes.md`、`planning.md`、`files.md`、`approval.md`，没有读取 `memory.md`。因此不能写成“读遍全部文件”。关键词没有匹配，也只能说明没有找到这些词；文件缺失还要结合目录清单确认。

发现缺失后，它在任务文字里加了“【受阻】”，将“读取时间预算”和“制定学习安排报告”保留为 `pending`。这里的“受阻”是原因说明，实际状态没有变成新的 `blocked` 类型。

同时，它把原来的保存和核验任务改成了“保存资料缺失说明报告”和“读回说明报告”，并继续执行。最后的清单是：

| 最终任务 | 状态 |
| --- | --- |
| 定位资料目录 | `completed` |
| 读取学习要求 | `completed` |
| 读取时间预算，注明文件不存在 | `pending` |
| 根据两份资料制定学习安排报告 | `pending` |
| 保存资料缺失说明 | `completed` |
| 读回并核验缺失说明 | `completed` |

![最终回复承认资料不足、任务未完成，并说明保存了缺失说明](../evidence/task3-images/missing-final-answer.png)

它没有编造可用时间，也没有把全部任务标记完成，这两点符合资料不足时如实说明的要求。但实际生成了 `blogs/task3-missing/post.md`，内容是 [资料缺失说明](../evidence/task3/missing-report.md)。这与本轮“两份资料齐全后再生成并保存报告”的预期不一致：它主动把交付物换成了说明文件，不能把这轮写成完全遵循要求。

这次让我看到，清单记录得合理，不代表后续每个动作都符合原要求。`TodoListMiddleware` 不会自动禁止保存操作，也不会替我检查模型是否悄悄调整了交付内容。这个偏差本轮先如实保留，没有为了得到更漂亮的结果重跑。

## 5. Trace、输出与打卡材料

已有记录保存在 LangSmith 的 `deepagents-course` 项目中。两轮 Trace 的 `thread_id` 相同，因此不能把第二轮描述成完全独立、没有前文影响的实验。

| 场景 | Trace ID | 耗时 | 模型调用 | 工具调用 |
| --- | --- | --- | --- | --- |
| 正常资料 | `01a15917-47a3-7ba2-a16b-165e4be04050` | 22.30 秒 | 8 | 7 |
| 缺失资料 | `01a15943-b13d-7112-b90e-84af16c958ed` | 91.98 秒 | 15 | 14 |

正常轮工具为 `write_todos` 4 次，以及读取、保存、读回各 1 次。缺失轮为 `write_todos` 4 次、`glob` 1 次、`ls` 1 次、`read_source` 4 次、`grep` 2 次、保存与读回各 1 次。

调用顺序按工具记录的 `start_time` 排序核对，没有直接采用 CLI 树的显示顺序。四次清单更新的请求与实际写入状态已提取到 [todo-updates.json](../evidence/task3/todo-updates.json)。

| 材料 | 文件 |
| --- | --- |
| 固定资料 | [source.md](../evidence/task3/source.md) |
| 正常轮实际报告 | [normal-report.md](../evidence/task3/normal-report.md) |
| 缺失轮实际说明文件 | [missing-report.md](../evidence/task3/missing-report.md) |
| 正常轮原始 Trace 树、工具输入输出 | [Trace](../evidence/task3/normal-trace.txt)、[工具记录](../evidence/task3/normal-tools.json) |
| 缺失轮原始 Trace 树、工具输入输出 | [Trace](../evidence/task3/missing-trace.txt)、[工具记录](../evidence/task3/missing-tools.json) |
| 提示词、配置与核对说明 | [运行记录](../evidence/task3/run-record.md) |

模型名称按本地配置记录为 `qwen3.8-flash`；树中的 `ChatOpenAI` 是适配器名称。根 Trace 的状态字段为空、Token 和费用字段未提供，不据此填写用量，也不把根运行结束当成业务任务完成。

对照打卡要求：本次验证了显式 Todo 配置下的计划更新与多步骤执行；改动在任务提示词层面；正常轮完成了报告，缺失轮识别了不足但出现额外保存行为；关键代码、截图、输出、状态和 Trace 均已归档。下一步修正集中在缺资料时的保存边界，不同时更换模型或工具。

## 6. 我的学习心得

这次最开始按提示词跑完，我还是怀疑自己到底有没有学会。因为清单怎么列、用什么工具、最后检查什么，都是学习助手事先写好的。应用跑通和我理解了整个第四章，确实不是同一回事。

我真正开始有感觉，是查看四次 `write_todos` 的时候。我发现保存任务没有出现进行中状态，以为是执行得太快。看完工具记录才明白，状态取决于模型有没有更新，不能把清单当成完整的执行录像。

缺资料实验更有意思。Agent 能自己查目录、搜索关键词、说明缺什么，还把两项任务留在待办状态。我一开始觉得这样就处理好了，后来才注意到，它又保存了一份缺失说明，实际改变了我要求的交付方式。以后检查结果时，除了看有没有完成，还要看它到底完成了什么。

我现在对 Todo 的作用有了更具体的理解，但还不能说已经熟练掌握整章中间件机制。这次先保留真实过程和没有解决的问题，再逐步补代码练习。

## 7. 下一步修正

后续只针对“缺资料时仍保存说明”这一点，明确区分正式报告与缺失反馈，观察是否仍会发生额外写入。如果业务要求必须阻止保存，再考虑在保存工具或执行逻辑中检查前置条件，不能只依赖完成状态。中间件 Hook 和状态持久化的细节，留待后续结合具体代码继续学习。

## 参考来源

- [课程第 4 章：任务规划与分解](https://datawhalechina.github.io/deepagents-in-action/chapters/ch04-task-planning/)
- [AgentSeek：content-builder 模板](https://github.com/agentseek-ai/agentseek-templates/tree/main/templates/deepagents/content-builder)
- 本地项目 `src/content_builder_deepagent/agent.py`，以及本次真实运行的截图、工具记录和实际文件。
