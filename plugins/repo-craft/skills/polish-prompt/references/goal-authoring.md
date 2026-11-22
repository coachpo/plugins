# GOAL 写法

写 `/goal` 条件时使用。`polish-prompt` 的 GOAL 模式和 `review-fix-goal` 都按本文件写；各技能的调用边界、参数来源和其余输出以各自的 `SKILL.md` 为准。

两个宿主都用 `/goal <条件>` 设定目标：条件文字既是第一轮的指令，也是完成判据，上限 4000 字符（[Claude Code](https://code.claude.com/docs/en/goal#write-an-effective-condition)、[Codex](https://learn.chatgpt.com/docs/developer-commands)）。判定方式不同：Claude Code 由独立的评估模型只读对话来判断，它不运行命令、不读文件；Codex 由执行模型对照证据审计后自行标记完成（[Codex Goals](https://developers.openai.com/cookbook/examples/codex/using_goals_in_codex)）。按更严格的 Claude Code 写，一条命令同时适用两个宿主：

- 条件写明预期结果（一个可度量的终态）、范围、授权边界和不得改变的东西、验证方法、阻塞依赖，以及**对话中出现什么证据算达成**，例如“对话中贴出的 `npm test` 输出显示退出码为 0”。要求执行者把命令和关键输出贴进对话，不只报告结论。
- 迭代型目标写明轮数上限，以及受阻、需要用户决定或达到上限时停下并输出交接报告；把“已输出交接报告”列为结束情形之一，并要求报告区分已完成与受阻，受阻不算完成。
- 超过 4000 字符时先删背景，不删授权边界、完成证据和停止条件；仍然超出，就在 `问题：` 行请用户决定怎样拆成几个目标。

设定目标即开始执行，所以有待决问题时除了标明是草稿，还要说明答复前不要运行这条 `/goal`。

代码块里只有一条以 `/goal ` 开头的命令，条件写成一段，不硬换行（两个宿主的官方示例都是单行，文档没有说明多行粘贴的行为）。
