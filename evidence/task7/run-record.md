# Task7 运行与归档记录

本轮继续使用 `/home/lx/projects/content_builder_deepagent`。真实模型实验由本人在网页发起；学习助手只查询已有 Trace、读取实际文件、执行无模型检查及整理归档，没有为了记录重新运行付费模型。时间表使用北京时间。

## 环境与固定输入

- 主项目：`deepagents/content-builder`，实际源码见 [agent.py](code/after/agent.py) 和 [lesson_tools.py](code/after/lesson_tools.py)。
- 模型：阿里云百炼 `qwen3.8-flash`，provider 为 `openai`，兼容 API 为北京地域 `https://dashscope.aliyuncs.com/compatible-mode/v1`。
- 七轮根 Trace 均记录 DeepAgents `0.7.8`。没有归档 `.env` 或实际密钥。
- 实验使用本地资料；未联网搜索、生成图片或委派子 Agent。
- 资料：[source.md](source.md)，内容为周六整理、周一完成；先写初稿再按反馈修改；每次保存或修改后读回。原文明确为自编练习资料。
- 缺失输入：`sources/task7-missing-notes.md`，最终归档时确认不存在；目标报告 `blogs/task7-missing-report/post.md` 同样不存在。

正常案例提示词：

```text
请根据 sources/task2-notes.md，编写一份简短的学习安排报告，包含学习安排、报告要求和检查要求，注明资料来源及其性质。

请将报告保存为本地文件，slug 使用 task7-normal-report。保存后重新读取实际文件，确认内容完整、来源标注正确，再告诉我结果。
```

缺失案例提示词：

```text
请根据 sources/task7-missing-notes.md，编写一份简短的学习安排报告，包含学习安排、报告要求和检查要求，注明资料来源及其性质。请将报告保存为本地文件，slug 使用 task7-missing-report。
```

不同轮次可能有换行差异；各轮实际请求以对应 root JSON 的 `inputs` 为准。

## 修改来源与边界

1. 收紧 Skill 的缺失处理，明确一次 `glob` 的路径、原文件名及停止条件。
2. 在主 Agent 系统提示词中增加本地报告任务先读取 Skill 的要求。
3. 将 `read_source` 的非法文件名校验从 `ValueError` 改为带明确说明的 `ToolException`；保留原有 `handle_tool_error=True`。
4. 给 `read_report` 的文件不存在情况增加 `ToolException` 处理和 `handle_tool_error=True`。非法 slug 校验仍在读取之前执行。

以上是逐步发生的累计修正。[changes.diff](code/changes.diff) 比较此前留存基线与最终实际文件，不是每一轮运行时的源码快照。基线 `code/before/agent.py` 和 `lesson_tools.py` 来自本仓库 Task6 归档，`code/before/SKILL.md` 来自 Task5 归档。最终源码和 Skill 直接从当前 WSL 项目按字节复制，未整理格式或修改内容。

首次模型请求摘录用于核对新增系统提示词是否真实传入。前三次缺失案例和第一次正常案例没有该条系统提示；参数错误案例及最终两轮包含。不能把中间一次缺失行为改善归因于尚未传入的提示词。

## 七轮记录

| 标签 | 开始时间（北京时间） | 秒 | LLM / tool | 根 Trace ID | 结果 |
| --- | --- | --- | --- | --- | --- |
| missing-baseline | 10-31 22:58:00 | 35.375 | 8 / 7 | `01a19277-b2b9-7901-9098-ea1b8a65ccf2` | 两次根目录 glob 和一次根目录 grep，不符合缺失查找标准 |
| missing-skill-only | 10-31 23:12:27 | 18.126 | 5 / 4 | `01a19284-ee9a-7773-ab94-d736112698cc` | 查找后才读 Skill，glob 未指定 path |
| missing-intermediate | 10-31 23:23:16 | 35.100 | 8 / 7 | `01a1928e-d585-7c31-936f-5fd489ee3b7c` | glob 已限定 sources，但新增系统提示未真实传入；未生成报告 |
| normal-readback-failure | 10-31 23:28:10 | 9.391 | 3 / 3 | `01a19293-4ff6-7ff0-8762-a59ef4f8ea8d` | 为检查是否覆盖而提前读取报告，FileNotFoundError 中断 |
| normal-argument-failure | 10-31 23:45:11 | 6.976 | 3 / 3 | `01a192a2-e551-7613-ad76-8485a9641495` | 传入 sources/task2-notes.md，ValueError 中断 |
| normal-final | 11-01 00:02:49 | 33.422 | 10 / 9 | `01a192b3-0b22-71e0-a207-80727414a337` | 保存、读回和磁盘一致，正常案例通过 |
| missing-final | 11-01 00:06:25 | 27.908 | 6 / 5 | `01a192b6-552f-7443-9f46-c4939ddbe8ee` | sources 内查找一次后停止、询问，无正式报告，缺失案例通过 |

`tool` 数量按 LangSmith 实际 tool Run 统计，包含 Todo 更新、文件工具等，不等于业务步骤数量。耗时只记录事实，不用于评价某一处修改的因果效果。

### 原始记录链接

| 运行 | 根 Run 与输入输出 | 工具输入输出 | Trace 树 |
| --- | --- | --- | --- |
| 首次缺失 | [root](missing-baseline-root.json) | [tools](missing-baseline-tools.json) | [trace](missing-baseline-trace.txt) |
| 只改 Skill | [root](missing-skill-only-root.json) | [tools](missing-skill-only-tools.json) | [trace](missing-skill-only-trace.txt) |
| 中间缺失 | [root](missing-intermediate-root.json) | [tools](missing-intermediate-tools.json) | [trace](missing-intermediate-trace.txt) |
| 报告读回异常 | [root](normal-readback-failure-root.json) | [tools](normal-readback-failure-tools.json) | [trace](normal-readback-failure-trace.txt) |
| 资料参数异常 | [root](normal-argument-failure-root.json) | [tools](normal-argument-failure-tools.json) | [trace](normal-argument-failure-trace.txt) |
| 最终正常 | [root](normal-final-root.json) | [tools](normal-final-tools.json) | [trace](normal-final-trace.txt) |
| 最终缺失 | [root](missing-final-root.json) | [tools](missing-final-tools.json) | [trace](missing-final-trace.txt) |

`*-root.json`、`*-tools.json` 和 `*-trace.txt` 保留 LangSmith CLI 的原始标准输出。`*-tool-sequence.json` 按开始时间提取工具名、参数、编号；`*-llm-metadata.json` 从 LLM 查询结果提取元数据；`*-first-model-request.json` 保存最早一次 LLM Run 的输入，外层添加来源说明。这几类提取文件和 [trace-summary.json](trace-summary.json) 都不是完整 API 原始响应。

### 会话条件

- `missing-intermediate` 与 `normal-readback-failure` 共用 thread：`206bbc6e-0876-4cf1-b3cd-957bc8637630`。这次正常请求继承了上一轮缺失任务的上下文，不能当成独立基线。
- 最终正常 thread：`cdd1ac3f-ef0b-4f6b-aded-dbe25bbb6544`。
- 最终缺失 thread：`fbdbf5ec-5b2e-41c2-a00d-b188665958dd`。
- 最终两个案例互相独立，但并不意味着整个修改过程都是同上下文、同版本的严格对照实验。

## 最终结果怎样核验

正常案例读取 Skill 在先，`read_source` 使用 `name="task2-notes.md"`。`save_report` 写入内容、后续 `read_report` 返回内容及 [实际报告副本](normal-report.md) 完全一致。三个指定部分、来源路径和练习资料性质均存在；最后 Todo 更新在读回之后，五项全部 completed。[最终回答提取](normal-final-reply.md) 保留了模型说明。

正常案例另有一次 `glob(path="/", pattern="**/task7-normal-report*")` 用于确认是否覆盖已有报告。本次 sources-only 约束适用于必需资料缺失后的查找，不把正常输出检查误记成缺失处理越界。

缺失案例的工具序列为：读取 Skill、write_todos、read_source、glob、write_todos。`read_source` 返回的 ToolMessage 状态是 `error`；`glob` 返回状态是 `success`、内容为 `No files found`。前者读取失败，后者查找成功执行但没有匹配，两者语义不同。根 Run 无 error 不代表每个业务步骤成功。

唯一一次 glob 的参数为：

```json
{"path": "/sources/", "pattern": "**/task7-missing-notes.md"}
```

没有 ls、grep、二次查找或 save_report。最终未生成目标报告，六项 Todo 均未 completed；两项 in_progress、四项 pending，文字注明受阻与等待补充。这是固定缺失案例的合格结果，不是报告任务已完成。[最终回答提取](missing-final-reply.md) 已保留。

归档核验见 [verification.json](verification.json)，共 17 项程序核对。它是根据原始记录与实际文件生成的摘要；文件内容中是否存在语义偏差仍需要人工阅读，不能仅凭字符串检查证明全部内容正确。

## 无模型证据

- [offline-checks.txt](offline-checks.txt)：实际工具检查和已有 pytest 结果的终端输出转录，明确不是原始日志文件。
- 工具检查覆盖正确资料名、带目录名、越界路径、缺失资料、缺失报告，以及临时目录中的保存读回。没有写入正式报告。
- 项目已有测试执行结果：`15 passed in 17.32s`。使用脚本模型和禁用 tracing 的测试配置，没有付费模型调用。
- 最终 live 两轮没有触发带目录参数纠正或缺失报告恢复，不能用最终成功记录声称模型实际完成了这些错误恢复。

## 配套 Notebook

本人运行产生的 [Notebook](notebook/ch13-grading-rubrics.ipynb)、[Markdown](notebook/ch13-grading-rubrics.md)、[HTML](notebook/ch13-grading-rubrics.html) 和 [report.json](notebook/report.json) 均直接复制。报告记录 offline、scripted、passed，执行时间为 10-31 22:51:29（北京时间），课程源码提交 `f93fe0a8be843eae77efaf85448035ddc54ff7ea`。

这些内容来自课程仓库，保留其作者和来源。归档用于阅读证据；复现应在原课程仓库使用其锁定环境，不能把 Notebook 的机制通过当作主项目真实模型已通过。

## 查询方式

在 Ubuntu 中加载主项目的环境配置后，读取已有记录，例如：

```bash
langsmith --format pretty trace get 01a192b3-0b22-71e0-a207-80727414a337 --project deepagents-course --include-metadata
langsmith --format json run get 01a192b3-0b22-71e0-a207-80727414a337 --project deepagents-course --include-io --include-metadata
langsmith --format json run list --trace-ids 01a192b3-0b22-71e0-a207-80727414a337 --project deepagents-course --run-type tool --include-io --include-metadata --limit 100
```

归档前检查了查询内容不包含当前环境中的实际密钥，没有保存环境密钥值。
