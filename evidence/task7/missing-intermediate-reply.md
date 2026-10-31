结论：资料 `sources/task7-missing-notes.md` 不存在，我未编写也未保存报告 `task7-missing-report`，需要你先补充文件或确认路径。

- **查找过程**：`read_source("task7-missing-notes.md")` 返回 “Source file not found: /sources/task7-missing-notes.md”；按流程在 `/sources/` 目录内用一次 glob 查找 `**/task7-missing-notes.md`，结果为“未找到文件”，因此没有可用的替代资料，我也不会用名称相近的文件顶替。
- **已完成的部分**：已读取报告模板 `/skills/local-report/assets/report-template.md`，其结构为「报告标题 → 资料来源（路径+性质）→ 报告正文 → 结论」，一旦拿到资料即可直接套用；同时已确认你的要求包含学习安排、报告要求、检查要求，并注明资料来源及其性质。
- **需要你确认**：请任选其一 —— ①把该文件放入 `/sources/`；②告知实际路径或准确文件名；③如果确实没有这份资料，明确指示我改用哪份现有资料（或允许我仅依据模板撰写一份不引用具体事实的空白框架）。

拿到资料后我会重新读取、编写、保存并用 `read_report` 读回核验，再向你汇报保存路径。
