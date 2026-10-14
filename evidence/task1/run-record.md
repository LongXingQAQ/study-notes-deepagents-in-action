# Task 1：自定义工具的 live 调用记录

学习日期：2026-10-13，与本人学习笔记一致。

来源：本人提供的[应用运行截图](../task1-images/live-tool-call.png)，以及本人查询并保存的 [LangSmith CLI 树形输出](trace-tree.txt)。下方参数和返回值为前端截图转录；Trace 文件是 CLI 实际输出。整理记录时没有重新运行模型。

模型名称由本人确认：阿里云百炼北京地域 `qwen3.8-flash`，通过 OpenAI 兼容接口接入；截图本身未展示模型名称。

## 用户请求

```text
请由主 Agent 直接调用 check_report_length 工具。
text 参数严格使用“你好 Agent”（不包含引号），max_chars 使用 5。
不要联网搜索，也不要委派子 Agent。
根据工具返回结果，告诉我实际字符数和是否超过上限。
```

## 工具调用

工具名称：`check_report_length`。

页面状态：`DONE`。

参数：

```json
{
  "text": "你好 Agent",
  "max_chars": 5
}
```

返回结果：

```json
{
  "character_count": 8,
  "max_chars": 5,
  "within_limit": false
}
```

同一组数据已保存为 [tool-result.json](tool-result.json)。该 JSON 是截图中工具结果的数据转录，不是程序直接导出的原始响应。

## 最终回复中的结果

- 实际字符数：8，两个汉字、一个空格、五个英文字母。
- 上限：5。
- 是否超过上限：是，超出 3 个字符。

## LangSmith Trace

- 项目：`deepagents-course`。
- 根名称：`LangGraph`。
- Trace ID / 根 Run ID：`01a13ac7-ac0a-74b1-a9a8-28f3f04d6cab`。
- 根耗时：`5.63s`。
- 模型 Run：2 个 `ChatOpenAI`，显示耗时分别为 `3.40s`、`2.13s`。
- 工具 Run：1 个 `check_report_length`，显示耗时为 `0ms`。
- chain Run：18 个；包括包装层，其耗时与内部节点重叠，不能相加。
- 没有 `tavily_search` 或 `task` 工具 Run。

查询命令：

```bash
langsmith trace get 01a13ac7-ac0a-74b1-a9a8-28f3f04d6cab \
  --project deepagents-course \
  --include-metadata \
  | tee /mnt/d/deepagents-in-action/evidence/task1/trace-tree.txt
```

[完整 CLI 输出](trace-tree.txt) / [终端截图](../task1-images/langsmith-trace.png)。

## 验证范围

截图展示了新工具的调用参数、完成状态、返回结果，以及模型对结果的解释。与预期 `8 > 5` 一致。

Trace 树展示调用层级和显示耗时，前端截图展示参数、工具返回、`DONE` 状态和最终回复。两类证据结合使用，树形输出本身没有完整输入输出和错误字段。

Trace 列表中的 `STATUS` 与 `TOKENS` 都是 `N/A`，没有把它们解读为失败或零用量。`0ms` 是工具耗时的显示值，不代表工具没有执行。
