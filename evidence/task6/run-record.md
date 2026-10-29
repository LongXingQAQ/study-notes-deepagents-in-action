# Task 6 运行记录

学习日期：2026-10-29；时区 Asia/Shanghai。

## 来源与范围

- 本人在已有 content-builder 主项目中运行真实模型，保存格式偏好、换新会话提问，并分别执行批准和拒绝两个 SDK 实验。
- 本轮没有修改 Python 实现，偏好文件新增一项 format；审批使用模板原有脚本，通过命令参数切换决定。
- `code/` 保存本次实际使用的 agent.py、lesson_tools.py 和 approval_demo.py。preferences.md、publications.jsonl、approval-report.md 为实际项目文件副本。
- 四张图片来自本人提供的原始截图，复制前后 SHA256 一致。
- `*-root.json`、`*-tools.json` 是已有 Trace 的 CLI JSON 查询结果；查询输出保留在文件中。工具列表并不包含被中间件拦截、尚未执行的工具请求，这些请求要结合模型输出和审批终端记录判断。
- `*-trace.txt` 保存已有 Trace 的 CLI 树形输出，排列不作为调用时间顺序的唯一依据。
- memory-save-reply.md、memory-read-reply.md 是从对应根 Run 最后一个 AI 消息提取的正文。
- memory-first-model-request.json 从新会话 LLM Run 中按开始时间找到第一次请求，提取其 inputs。它是序列化后的实际请求，不是人工补写的提示词。
- verification.json 是程序根据导出记录和文件检查生成的摘要，不是原始模型输出。
- approval-output.txt 来自用户粘贴的终端输出转录，不是重新执行产生的日志。
- 归档没有再次调用模型，没有修改项目，也没有归档 .env 或密钥。

## 模型与项目

- 应用：`/home/lx/projects/content_builder_deepagent`，模板 deepagents/content-builder。
- LangSmith 项目：`deepagents-course`。
- 模型：阿里云百炼 qwen3.8-flash；通过 openai 兼容适配器调用。新会话 LLM 元数据也记录此模型名称。
- 本轮使用主项目既有 DeepAgents 0.7.8 环境；没有运行或冒充第8—9章 offline Notebook 成果。

## 记忆实验

### 请求

首次：

```text
请记住我的回答格式偏好：先用一句话给出结论，再列出三条要点。以后默认采用这个格式。只保存这一项偏好。
```

重试：

```text
请调用 save_preference 保存我的回答格式偏好，参数严格使用：key 为 format；value 为“先用一句话给出结论，再列出三条要点”。这是我明确要求记住的偏好，只保存这一项。
```

以上按对话中发送的文字整理，原请求保留在根 Run 的 inputs 中。新会话实际请求是“请解释一下 Skill 和工具有什么区别”。

### 结果与 Trace

| 阶段 | 根 Trace / Run ID | 北京时间 | 耗时 |
| --- | --- | --- | --- |
| 首次错误 | `01a18845-c598-7a72-9979-93660910ea2f` | 2026-10-29 23:27:16 | 2.328秒 |
| 明确参数后保存 | `01a18848-db27-7903-9a56-e327d3563196` | 2026-10-29 23:30:38 | 5.299秒 |
| 新会话读取 | `01a1884c-fc6f-7321-8fe1-fb2a2a566aee` | 2026-10-29 23:35:09 | 20.506秒 |

首次工具参数 key=answer_format，引发 ValueError；该 Trace 没有成功工具返回。随后同一 thread 重试，实际 key=format，工具 status=success，返回内容与磁盘上的 preferences.md 一致。

保存 thread：`e4fcad40-f553-4c14-b51a-7abf74d49fe5`；新问答 thread：`12d67ff5-7edb-40d6-bd99-0927c3cc4e1c`。

新会话第一次模型请求包含“先用一句话给出结论，再列出三条要点”，没有旧的保存偏好请求。请求中只有当前问答对应的用户消息，文件记忆与旧消息历史分开核对。回答主体体现此格式，另有补充示例，不声称完全没有额外段落。

新会话调用 ls(/skills/) 和 read_file(/skills/blog-post/SKILL.md, limit=60)，并没有显式调用读取偏好文件的工具。不能把读 Skill 的记录误作读取记忆的证据；记忆加载以实际模型请求为准。

对应文件：

- [首次错误](memory-before-root.json)、[错误工具调用](memory-before-tools.json)、[Trace](memory-before-trace.txt)。
- [保存结果](memory-save-root.json)、[工具输入输出](memory-save-tools.json)、[Trace](memory-save-trace.txt)、[最终回复](memory-save-reply.md)。
- [新会话结果](memory-read-root.json)、[工具输入输出](memory-read-tools.json)、[Trace](memory-read-trace.txt)、[模型请求](memory-first-model-request.json)、[回答](memory-read-reply.md)。

旧调用在后续界面出现取消说明，是前序未完成工具消息的显示结果，不能据绿色 DONE 推断旧调用写入成功。删除偏好仅在理解问答中讨论，没有实际执行。

## 审批实验

### 实现与终端证据

脚本先通过 save_report 准备固定报告，再用 build_graph(model, checkpointer=InMemorySaver(), approvals=True) 创建图。待审批工具是 publish_report(slug="approval-demo")，仅追加本地测试发布记录。

graph.invoke 返回后，脚本要求存在 __interrupt__，核对 action_requests 中的工具名、slug，且发布日志条数没有变化，然后在同一个 config 下使用 Command(resume={"decisions": [{"type": decision}]}) 恢复。

决定由命令行 --decision 提前给出，并非网页审批或交互式终端输入。脚本恢复后还检查没有再次中断，且日志变化符合所选决定。

[两次终端转录](approval-output.txt)：approve 为 0→1，reject 为 1→1。归档时实际日志仍然只有一条。公开日志与报告见 [publications.jsonl](publications.jsonl) 和 [approval-report.md](approval-report.md)。

### Trace

| 阶段 | 根 Trace / Run ID | 北京时间 | 耗时 | 实际工具执行 |
| --- | --- | --- | --- | --- |
| 批准前暂停 | `01a18852-4768-73e2-a918-dc45e949dc90` | 23:40:56 | 1.575秒 | 无 |
| 批准后恢复 | `01a18852-4d90-7a02-a833-fc602ee29e68` | 23:40:57 | 3.213秒 | publish_report 一次 |
| 拒绝前暂停 | `01a18854-19e0-71a0-9876-6d6dbbd4ad1c` | 23:42:55 | 1.402秒 | 无 |
| 拒绝后恢复 | `01a18854-1f5c-7842-8d58-b1f843c514ed` | 23:42:56 | 4.937秒 | 无 |

时间均为2026-10-29北京时间，原 JSON 使用 UTC。四段根 Run 错误字段为空，不把暂停时“没有根错误”解释为发布已完成。

批准实验两段使用 thread `19035cb4-de38-4439-ab0e-367cdb3b17c2`；拒绝实验两段使用 thread `1a18a17a-7ce6-407a-9bff-1a79fa34cca2`。两次实验各用一个新 thread，每次内部的暂停和恢复共用其 thread。

- [批准前 Trace](approve-pause-trace.txt)、[根记录](approve-pause-root.json)、[工具列表](approve-pause-tools.json)。
- [批准后 Trace](approve-resume-trace.txt)、[根记录](approve-resume-root.json)、[工具列表](approve-resume-tools.json)。
- [拒绝前 Trace](reject-pause-trace.txt)、[根记录](reject-pause-root.json)、[工具列表](reject-pause-tools.json)。
- [拒绝后 Trace](reject-resume-trace.txt)、[根记录](reject-resume-root.json)、[工具列表](reject-resume-tools.json)。

暂停的 CLI 根输出主要保留 messages，不将它冒充完整的 __interrupt__ 原始对象。审批请求结构的证据来自实际终端转录，以及脚本检查到 __interrupt__ 才继续的逻辑。准备报告的独立 save_report Trace 不计为发布执行。

本例内存 Checkpointer 只在脚本进程内使用，没有测试跨进程恢复、重复恢复幂等性或多用户隔离。文件偏好保存在磁盘，与进程内 Checkpoint 的生命周期不同。

## 查询方式

凭据由本地环境变量提供：

```bash
langsmith --format pretty trace get TRACE_ID --project deepagents-course --include-metadata
langsmith --format json run get TRACE_ID --project deepagents-course --include-io --include-metadata
langsmith --format json run list --trace-ids TRACE_ID --project deepagents-course --run-type tool --include-io --include-metadata --limit 100
```

将 TRACE_ID 替换为上表完整 ID。新会话首次模型请求通过同一 Trace 的 llm Run 查询，按 start_time 升序取第一次输入。Token 或费用未作总量推算。
