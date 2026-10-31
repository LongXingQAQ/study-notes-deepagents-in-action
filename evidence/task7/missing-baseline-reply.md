无法生成报告：**必需资料 `sources/task7-missing-notes.md` 不存在**，按规范在资料缺失时不编写、不保存正式报告。

排查过程：
- `read_source("task7-missing-notes.md")` → `Source file not found: /sources/task7-missing-notes.md`
- 列出 `/sources/`：`approval.md`、`files.md`、`memory.md`、`planning.md`、`task2-notes.md`
- 全盘按 `*task7*` 查找、并全文检索 "task7" → 均无匹配

唯一名称相似的候选（性质：自编练习资料，非真实调研结果）：
- `/sources/task2-notes.md` — 内容恰好包含“学习安排 / 报告要求 / 检查要求”三项，可能是你要的那份资料，但主题标注为 **Task2**，不能与 Task7 混用。

请确认以下任一方式后我再继续：
1. 用 `task2-notes.md` 的内容编写，slug 仍用 `task7-missing-report`（或改用 `task2-report`）；
2. 补充或修正 `task7-missing-notes.md` 的真实路径并放入 `/sources/`；
3. 直接把该资料内容粘贴给我。

对应待办（报告编写、保存、读回核验）保持未完成状态，等你确认后执行。
