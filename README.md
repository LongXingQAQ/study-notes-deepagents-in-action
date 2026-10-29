# Deep Agents 学习记录

我通过 Datawhale《Deep Agents 实战》学习如何从概念走向可运行、可调试、有证据的 Agent 应用。

这个仓库保存学习总结、实验修改和执行证据。实际开发环境位于 WSL2 Ubuntu，研究应用与课程 Notebook 使用各自的虚拟环境。

## 学习记录

| 任务 | 笔记 | 当前状态 |
| --- | --- | --- |
| Task 0：入群准备与第一次成功 | [学习总结](notes/task0.md) | 技术实操与单变量实验已核对，文章待本人审查和公开发布 |
| Task 1：我的第一个工具 Agent | [学习笔记](notes/task1.md) | 自定义工具 live 调用与 LangSmith Trace 已保存，笔记待本人审查 |
| Task 2：让成果留在文件里 | [学习笔记](notes/task2.md) | content-builder 保存、读回与修改实验已核对，文件和两轮 Trace 已保存，笔记待本人审查 |
| Task 3：让项目按计划完成 | [学习笔记](notes/task3.md) | 正常规划与缺资料实验已归档；保存边界偏差如实记录，笔记待本人审查 |
| Task 4：把一项工作交给子 Agent | [学习笔记](notes/task4.md) | Notebook 机制验证与本地报告审核委派已核对，代码、输出和 Trace 已归档，笔记待本人审查 |
| Task 5：把经验做成可复用 Skill | [学习笔记](notes/task5.md) | 本地报告 Skill、模板与 live 实验已归档；工具异常处理修正和查找范围偏差如实记录，笔记待本人审查 |
| Task 6：记住用户偏好，执行前等待审批 | [学习笔记](notes/task6.md) | 文件偏好写入、新会话加载及批准/拒绝实验已核对，源码、日志和 Trace 已归档，笔记待本人审查 |

## 实验产物与文中图片

- `evidence/task0/agentseek-records.txt`：本次对话中实际命令输出的转录。
- `evidence/task0/baseline/`：本人运行 offline 最小工具实验生成的原始产物。
- `evidence/task0/changed/`：修改后的 Notebook、提取输出和保存结果核验记录；源码只修改 `TEXT`。
- `evidence/task0/trace-analysis.md`：已有 Trace 的分析记录，来源与核验范围在文件中说明。
- `evidence/task0/trace-tree.txt`：本人查询已有运行得到的完整原始 Trace 树；正文展示结构摘要。
- `evidence/task0-images/`：学习总结直接展示的 `info`、任务列表、`doctor`、研究结果、工具返回与断言通过截图。
- `evidence/task1/run-record.md`：报告长度检查工具的 live 调用与 Trace 摘要，模型名称由本人确认。
- `evidence/task1/tool-result.json`：从同一张截图整理出的工具结果数据，不是程序原始导出。
- `evidence/task1/trace-tree.txt`：本人查询此次运行得到的 LangSmith CLI 树形输出。
- `evidence/task1-images/`：工具定义、导入、注册、实际调用和 LangSmith Trace 的五张截图。
- `evidence/task2/run-record.md`：content-builder 两轮 live 文件实验的提示词、参数与结果整理，以及前端依赖处理记录。
- `evidence/task2/source-after.md`、`report-after.md`：从 WSL 项目复制的修改后资料与报告，完成日期均为周一。
- `evidence/task2/trace-01.txt`、`trace-02.txt`：保存并读回、修改并读回两轮运行的 LangSmith CLI 原始树形输出。
- `evidence/task2-images/`：练习资料、工具参数与返回结果、报告修改前后等十张截图；正文直接展示关键图片。
- `evidence/task3/`：固定资料、正常报告、缺失说明、两轮原始 Trace 树及工具输入输出、八次 Todo 更新和运行记录。
- `evidence/task3-images/`：四次正常清单更新、资料目录、缺失查找与最终回复等六张截图。
- `evidence/task4/notebook/`：本人执行第 5 章 offline 实验生成的 Notebook、Markdown、HTML 与运行报告。
- `evidence/task4/code/`：审核子 Agent 接入前后源码、差异补丁与无模型测试代码归档。
- `evidence/task4/`：被审核报告、子 Agent 审核意见、原始 Trace 树、工具输入输出、运行记录和检查结果转录。
- `evidence/task4-images/`：Notebook 通过和真实审核结果的两张截图；前端固定显示名称的问题在笔记中说明。
- `evidence/task5/`：实际 Skill 与报告模板、工具修正源码、报告三个版本，以及正常流程和缺失资料修正前后的 Trace 与工具输入输出。
- `evidence/task5-images/`：Skill 与模板读取、报告流程、资料缺失结果和 Trace 列表等八张原始截图。
- `evidence/task6/`：偏好文件、新会话模型请求、审批脚本与配置、实际发布日志、终端转录，以及记忆和审批各阶段的 Trace 与工具记录。
- `evidence/task6-images/`：首次参数错误、保存偏好、实际文件和新会话回答四张截图。

这个仓库中的 Notebook 归档用于阅读执行证据。复现实验时，请在原课程仓库按其 Notebook 说明安装依赖，并选择课程内核；归档 Notebook 保留教学材料中的相对阅读链接，不应假定仅克隆这个学习仓库即可独立执行。

## 课程来源

- [Datawhale：Deep Agents 实战](https://github.com/datawhalechina/deepagents-in-action)
- [准备篇（上）](https://datawhalechina.github.io/deepagents-in-action/chapters/pre01-agentseek-create/)
- [准备篇（下）](https://datawhalechina.github.io/deepagents-in-action/chapters/pre02-agentseek-skills/)
- [Notebook 实验与运行说明](https://github.com/datawhalechina/deepagents-in-action/blob/main/notebooks/README.md)

复制保存的课程 Notebook 和阅读产物来自上述课程，保留原作者与来源信息，适用原仓库相应许可。本人总结为学习记录，不代表课程或模型供应商的结论。
