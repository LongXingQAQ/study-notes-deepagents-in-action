# Deep Agents 学习记录

我通过 Datawhale《Deep Agents 实战》学习如何从概念走向可运行、可调试、有证据的 Agent 应用。

这个仓库保存学习总结、实验修改和执行证据。实际开发环境位于 WSL2 Ubuntu，研究应用与课程 Notebook 使用各自的虚拟环境。

## 学习记录

| 任务 | 笔记 | 当前状态 |
| --- | --- | --- |
| Task 0：入群准备与第一次成功 | [学习总结](notes/task0.md) | 技术实操与单变量实验已核对，文章待本人审查和公开发布 |

## 实验产物与文中图片

- `evidence/task0/agentseek-records.txt`：本次对话中实际命令输出的转录。
- `evidence/task0/baseline/`：本人运行 offline 最小工具实验生成的原始产物。
- `evidence/task0/changed/`：修改后的 Notebook、提取输出和保存结果核验记录；源码只修改 `TEXT`。
- `evidence/task0/trace-analysis.md`：已有 Trace 的分析记录，来源与核验范围在文件中说明。
- `evidence/task0/trace-tree.txt`：本人查询已有运行得到的完整原始 Trace 树；正文展示结构摘要。
- `evidence/task0-images/`：学习总结直接展示的 `info`、任务列表、`doctor`、研究结果、工具返回与断言通过截图。

这个仓库中的 Notebook 归档用于阅读执行证据。复现实验时，请在原课程仓库按其 Notebook 说明安装依赖，并选择课程内核；归档 Notebook 保留教学材料中的相对阅读链接，不应假定仅克隆这个学习仓库即可独立执行。

## 课程来源

- [Datawhale：Deep Agents 实战](https://github.com/datawhalechina/deepagents-in-action)
- [准备篇（上）](https://datawhalechina.github.io/deepagents-in-action/chapters/pre01-agentseek-create/)
- [准备篇（下）](https://datawhalechina.github.io/deepagents-in-action/chapters/pre02-agentseek-skills/)
- [Notebook 实验与运行说明](https://github.com/datawhalechina/deepagents-in-action/blob/main/notebooks/README.md)

复制保存的课程 Notebook 和阅读产物来自上述课程，保留原作者与来源信息，适用原仓库相应许可。本人总结为学习记录，不代表课程或模型供应商的结论。
