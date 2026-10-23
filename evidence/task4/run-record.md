# Task 4 运行记录

整理日期：2026-10-24，时区 Asia/Shanghai。

## 来源和范围

- Notebook 由本人在 Ubuntu 中运行，`notebook/` 保存本次执行后的产物。
- 主项目 `agent.py` 的修改和新增无模型测试由学习助手按本人请求完成；本人随后启动应用进行真实模型审核。
- `code/agent-before.py` 来自修改前备份，`code/agent-after.py` 与 `code/test-course-after.py` 来自修改后实际文件，补丁由两份源码生成。
- 两张图片复制自本人提供的原图，位于 `../task4-images/`。
- `trace-tree.txt`、`tool-runs.json` 是只读查询已有 Trace 得到的 CLI 输出。`reviewer-feedback.md` 从 `task` 工具返回的消息正文提取，未重新生成审核结果。
- `reviewed-report.md`、`source.md` 是实际文件副本。被审核报告与 Task3 的 `normal-report.md` 字节一致。
- 归档过程没有再次调用模型，未保存 `.env` 或密钥。

## 环境

| 项目 | 记录 |
| --- | --- |
| 主项目 | `/home/lx/projects/content_builder_deepagent` |
| 模板 | `deepagents/content-builder` |
| 主项目 DeepAgents | `0.7.8`，来自已有配置与 Trace 元数据 |
| 模型配置 | `AGENTSEEK_MODEL=qwen3.8-flash`，provider 为 `openai` |
| 模型 API 地址 | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| 内容模式 | `text`，搜索和图片开关均为 `false` |
| Notebook 项目 | `/home/lx/projects/deepagents-course-materials` |
| Notebook 环境 | Python `3.12.3`、Linux x86_64、DeepAgents `0.7.22`，来自实际输出 |
| Notebook 模式 | `offline`，脚本模型，机制验证 |

## Notebook

```bash
cd "$HOME/projects/deepagents-course-materials"
uv run --project notebooks --locked python -m course_notebooks.run ch05-subagents --mode offline
```

终端输出 `ch05-subagents: passed (offline)`。报告中的源提交为 `f93fe0a8be843eae77efaf85448035ddc54ff7ea`，执行时间 `2026-10-23T16:39:45.787273+00:00`，即北京时间 2026-10-24 00:39:45。

实际 Notebook 验证了：`task` 指定 `researcher`；子 Agent 内部工具消息没有合并到主消息；`StateBackend` 的文件更新回到主状态；后续显式传回消息和文件状态，再通过 `read_file` 读取正文。

这是脚本模型安排响应的实验，不验证真实模型的自主规划和审核判断。

## 真实模型审核请求

以下为本次对话中提供并用于运行的请求，按行排版：

```text
请通过 task，把 slug 为 task3-plan 的报告交给 report_reviewer 审核。

请在委派任务中说明以下要求：
1. 必须先用 read_report 读取实际报告。
2. 检查是否包含学习安排、报告要求、检查要求三个部分。
3. 检查安排是否为周六整理资料、周一完成报告。
4. 检查是否注明 sources/task2-notes.md 和自编练习资料性质。
5. 逐项给出依据和审核结论。

主 Agent 根据子 Agent 返回的意见汇总回复。
不修改文件，不生成新报告，不联网。
```

实际 `task` 参数中的 `subagent_type` 为 `report_reviewer`，`description` 包含 slug、读取要求、核验标准和返回格式，完整内容保留在 `tool-runs.json`。

## Trace 与结果

- LangSmith 项目：`deepagents-course`。
- Trace ID 与根 Run ID：`01a169b4-0d6b-76a0-9aeb-828767ed9a7b`。
- thread ID：`a5c8afe5-c739-4310-9180-10d41c108169`。
- 根名称：`LangGraph`；显示耗时：53.30 秒。
- 开始、结束时间：北京时间 2026-10-24 00:59:30 至 01:00:23。
- `llm` 节点 5 个：主流程 2 个，`report_reviewer` 内部 3 个。
- `tool` 节点 3 个：`task` 委派包装层 1 个，子流程业务工具 2 个（`read_report`、`ls`）。
- 审核员实际先读取 slug 为 `task3-plan` 的报告，再列举 `blogs/task3-plan` 目录；顺序按工具 `start_time` 核对。
- `read_report` 返回的正文与归档的 `reviewed-report.md` 完全一致。
- 子 Agent 返回四项检查均符合、总体 PASS，并附三条非阻塞建议；主 Agent 在页面汇总结果。
- 没有保存或编辑工具调用。被审核文件与此前 Task3 归档副本一致。
- 三个工具 Run 的错误字段均为 null，不能由此推断审核能力已覆盖所有场景。
- 根状态、Token 和费用字段没有可用数据，本记录没有补填。树中的 `ChatOpenAI` 仅表示适配器，模型名称按本地配置记录。

## 查询方式

```bash
langsmith --format pretty trace get 01a169b4-0d6b-76a0-9aeb-828767ed9a7b \
  --project deepagents-course --include-metadata

langsmith --format json run list \
  --trace-ids 01a169b4-0d6b-76a0-9aeb-828767ed9a7b \
  --project deepagents-course --run-type tool \
  --include-io --include-metadata --limit 100
```

输出分别保存到 `trace-tree.txt`、`tool-runs.json`。工具结果共 3 条，没有达到查询上限。完整树用于定位子流程，实际工具顺序通过时间字段核对。

## 页面标签问题

截图显示 `Sub-agent: researcher`，原因是 `frontend/src/ToolCallCard.tsx` 的 `task` 卡片标题固定使用了该文字。相关原代码见 `frontend-label-excerpt.txt`。实际委派对象由 `task` 参数及 `report_reviewer` 子流程共同确认，本轮没有修改前端。

## 本次改动与边界

新增本地审核子 Agent，配置职责、提示词、自定义工具及注册，主 Agent 增加审核委派提示。本轮没有更换模型、改写报告或修改原有搜索开关；未进行修改前的 live 对照，不声称提升了效果或减少了 Token。

`tools=[read_report]` 并不移除框架提供的所有内置工具，真实子流程中还出现了 `ls`。本轮“只读审核”是提示词职责约束，加上本次没有写工具调用、报告内容未变的观察结果，不能泛化为已经建立权限隔离。
