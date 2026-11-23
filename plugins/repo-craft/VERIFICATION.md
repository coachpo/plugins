# 验证记录

本文区分当前源码的验证结果和旧版本的试跑记录。模板或技能正文改动后，历史结果只适用于注明的版本和场景。

## 方法

- **隔离样例**：在暂存目录建 Git 项目，用 Claude Code 2.1.322 的无界面会话（`claude -p`，模型 `claude-sonnet-5-5`，`--permission-mode acceptEdits`）以 `--plugin-dir` 加载插件源码，用 `--add-dir` 允许读取插件目录。精简标准和外部链接试跑另用 macOS `sandbox-exec` 对子进程禁止写入源码和安装缓存。初始化事件确认插件路径。需要接续对话的旧场景用 `--resume`，检验再次维护时另开会话。
- **产物核对**：用 `git status`、`git diff`、`cmp` 和临时脚本检查产物：按条款比对模板、按索引格式的核对清单检查 `CONSTRAINTS.md`、解析相对链接和锚点。
- **宿主发现**：Claude Code 用 `claude plugin validate`；Codex 用 Codex CLI 0.167.0 在空白的临时 `CODEX_HOME` 下添加本地市场并 `codex plugin list`。
- **局限**：机器的用户级指令和设置照常加载，没有隔离；样例都是只有几份文件的小项目；执行者是同一个模型家族；这是按源码路径执行，不是安装后的自然语言路由。

## 已验证

### 打包与宿主发现

- 0.0.3 在仓库根运行 `claude plugin validate plugins/repo-craft` 和 `claude plugin validate .`，均输出 `✔ Validation passed`。同次运行 `python3 ~/.claude/plugins/marketplaces/claude-plugins-official/plugins/skill-creator/skills/skill-creator/scripts/quick_validate.py plugins/repo-craft/skills/maintain-docs-and-rules`，输出 `Skill is valid!`。解析两份清单确认版本均为 `0.0.3`。
- 两份插件清单均可解析，共有字段逐项相同。Claude Code 要求 512 至 2048 像素的正方形 PNG 图标，`assets/icon.png` 是由 `assets/repo-craft.svg` 渲染的 1024 × 1024 版本；`.claude-plugin/plugin.json` 的 `icon` 只供官方目录展示，Claude Code 加载时不读。Codex 的 `logo` 和 `composerIcon` 使用 SVG（正方形 128 × 128）。
- Codex CLI 0.167.0 添加本地市场后，`codex plugin list` 列出 `repo-craft@coachpo`。
- Skill Creator 的 `quick_validate.py` 对 `constraint-manager`、`maintain-docs-and-rules`、`investigate-repo` 通过。`polish-prompt` 因 `disable-model-invocation` 不在该校验器接受的通用字段内而失败：这个字段是 Claude Code 仅显式调用所需的，保留不改。

### Claude Code 安装后的加载与调用控制（0.3.1 的安装缓存，技能正文当时与源码逐文件一致）

- 初始化事件列出各技能和子代理 `repo-craft:repo-researcher`，插件路径指向安装缓存。
- `polish-prompt` 不接受模型调用：技能清单里没有它，模型直接调用被宿主以 `disable-model-invocation` 拒绝。用户以 `/repo-craft:polish-prompt` 主动调用时，一轮内返回只含提示词的草稿代码块、两个宿主各一行强度建议和待决问题，没有执行初稿中的任务。之后技能正文加入了 GOAL 模式；改动后按源码路径重跑的结果见下文“`polish-prompt` 的 GOAL 模式”，安装后的检查没有重跑。
- `repo-craft:repo-researcher` 可被委派：只用 `Read`、`Grep`、`Glob`，返回带路径和行号的结论、冲突和未搜索范围；主 agent 重读了其中的关键证据。

### `maintain-docs-and-rules`

#### 外部链接（0.0.3）

在隔离样例根目录运行 `claude -p --model claude-sonnet-5-5 --permission-mode acceptEdits --plugin-dir <插件源码目录> --add-dir <插件源码目录> --no-session-persistence --strict-mcp-config --verbose --output-format stream-json -- '<技能调用和请求>'`，显式调用 `/repo-craft:maintain-docs-and-rules`。从初始化事件核对 Claude Code 2.1.322、模型和 `repo-craft@inline` 的源码路径，从最终回复与实际文件分别核对交接和改动。

维护与只读审查从同一初始提交的副本开始。README 有六条外部链接，artifact 旁记录了与设计的已知偏差，另外三条违规链接各自只占一句。初始化样例仅含同一份 README。旧模板样例用 `git show` 取出 `c38fd18` 的两份模板，配齐占位文档和空 ADR 目录。提示只指定任务和范围，未列出要移除或保留的链接。

共 9 次独立会话，退出码均为 0。初稿跑了初始化、维护、只读审查和旧模板升级各 1 次。前三份报告能区分四个违规链接，但缩写了地址。交接格式明确要求完整原链接或路径后，重跑这四项，并增加同提示再次维护，共 5 次。定稿试跑放行了 `cmp`、`diff`，初始化还放行了复制贡献模板和创建 ADR 目录的命令。

| 场景 | 定稿试跑结果 |
| --- | --- |
| 初始化 | 移除 artifact、Agent 线程、设计分享和仓库外本机路径，保留偏差原句、官方文档和法规链接。交接列出四个完整原地址，说明偏差所在小节及三处整句删除，并列出未核实的设计依据。`CONTRIBUTING.md` 与新模板逐字节一致。 |
| 维护 | `git diff` 只有 README 改动。交接将完整 artifact 链接对应到 README 的“当前限制”，其余三个完整链接或路径各标“整句删除”。保留的两条链接均已按类型核对，设计依据列为疑点。 |
| 同提示再次维护 | 在样例内提交首次维护结果，用相同提示“维护本项目的整套文档，直接更新仓库并交接结果。”另开会话。`git status --porcelain=v1 -uall` 无输出，`git diff --exit-code` 退出 0。 |
| 只读审查 | 报告逐项列出四个完整原地址为拟删项、两条公开资料为保留项。工具记录只有只读操作，`git status --porcelain=v1 -uall` 无输出。 |
| 0.0.2 模板再次维护 | 贡献指南的“文档”小节新增一条，`git diff --stat` 为 `1 file changed, 1 insertion(+)`。两份文件各自与新版模板 `cmp` 均退出 0，新条款仅出现一次，交接没有冲突询问。 |

定稿的只读审查和旧模板升级都误报 `docs/adr/` 原本缺失。实际样例已预建该空目录。逐项核对工具调用，初始化、维护和审查均未登录或打开会话链接。

措辞修订后，允许类别从“项目自有的公开主机和仓库”改为“项目自有的主机和代码仓库（含私有仓库）”，并把第三方平台的控制台或仪表盘列为受限材料。按同一方法另跑维护 1 次，退出码 0。样例的 `package.json` 用 `repository` 指向 `github.com/acme/widget`，并配齐固定文档、与新模板一致的 `CONTRIBUTING.md` 和空 ADR 目录。README 链接了项目仓库及其中一个 issue、Stripe 官方文档、`localhost` 地址、Stripe 控制台和一个 AI 对话。结果是：仓库和 issue 链接、官方文档、本地地址保留，控制台和对话链接整句删除；交接逐项列出完整原链接，`git diff --stat` 只有 README 和架构文档。这次没有误报 `docs/adr/` 缺失。

#### 精简标准（0.0.2 的历史试跑）

在隔离项目根目录用 `claude -p --model claude-sonnet-5-5 --permission-mode acceptEdits --plugin-dir <插件源码目录> --add-dir <插件源码目录> -- '<技能调用和请求>'` 显式调用 `/repo-craft:maintain-docs-and-rules`。14 次独立会话均由 Claude Code 2.1.322 加载 `repo-craft@inline` 源码，退出码均为 0。维护和审查使用同一初始提交的不同副本，样例一开始已有全部固定文档和旧 ADR。

| 场景 | 最近一次实测结果 |
| --- | --- |
| 维护整套文档 | 交接列了 5 项删除及源码或 schema 路径，包括混合段落的实现语句和旧 ADR 的实现段。混合段保留调用方约束及理由，术语归专题并从架构链接，外部运行限制与恢复步骤归运维，有依据的暂缓事项归新 ADR。进度和临时 TODO 已标明性质并删除。旧 ADR 的背景、结论及放弃方案与取舍逐字保留。`AGENTS.md` 无改动。 |
| 同提示再次维护 | 在样例内提交首次维护结果，新会话沿用“维护本项目的整套文档，直接更新仓库并交接结果。”这一提示。`git status --porcelain=v1 -uall` 无输出，独立比对得到 `12 files compared with baseline, 0 changed`。 |
| 空项目初始化 | 产物与两份新模板逐字节一致。另按小节核对条款及顺序，跨文件重复为 0。四份固定文档各含一行占位注释，`docs/adr/` 为空，没有专题占位。 |
| 旧模板维护 | 用 `git show` 取 `27f033c` 的两份模板建样例。`git diff --stat` 只列出根规则入口名单替换一行、贡献指南新增“文档”小节六行。产物与新模板逐字节一致，无跨文件重复或冲突询问。 |
| 只读审查 | 报告确认固定位置齐全，建议按需建术语专题，给出 5 项拟删实施细节及来源（含旧 ADR），列明混合段应保留的约束和拟迁入位置。独立比对得到 `10 files compared with baseline, 0 changed`。审查还把 README 直链术语表列为必需，技能允许从 README 经架构入口访问。 |

初稿各跑表中场景 1 次。只做文档时仍修改了根规则，交接也漏列了混合段删去的实现语句。明确写入范围后，维护、同提示再次维护和审查各跑 1 次，根规则保持原样，但有理由的暂缓事项留在运维文档，审查还将单个术语的归属当作可选。

明确两类内容的唯一归属后，表中 5 个场景各跑 1 次。维护会话没有逐项检查全部链接，另用临时脚本遍历产物，得到 `17 relative links, 10 anchors, 0 unresolved`。这轮审查将有术语才需要的专题误称为固定位置。把固定与按需文件分表后，再单独跑只读审查 1 次，得到表中审查结果。

#### 精简标准引入前的样例

- 空项目初始化：根 `AGENTS.md` 和 `CONTRIBUTING.md` 与模板逐字节一致，其余四个固定文档是一行占位注释，`docs/adr/` 为空目录；产物提交后再次维护没有改动（0.3.6 时的模板）。
- 有清单和一个包的项目：初始化后的两份文件通过按条款比对，写了“尚无生产部署”的工作假设，再次维护没有改动（0.3.4 时的模板和技能正文）。
- 按旧版模板初始化的项目再次维护：模板条款补入，属于另一份模板的条款移到对方文件，不产生冲突询问；项目专有规则和冲突项保留原条款并逐项询问，答复“用模板覆盖”后只改冲突那一行。最终合并规则措辞下，该样例共 6 次中有 1 次把一条等义规则整条保留、同时又补入了对方文件，导致两份文件各留一份。
- 模板里的局部规则入口条款：Claude Code 上 Sonnet 做改函数任务时有入口读贡献指南 2 次、无入口 0 次；Opus、Codex 以及 Sonnet 做补 README 任务时，没有入口也会读；Haiku 两次都没有照入口去读。
- 遮蔽条件：根目录有 `CLAUDE.local.md` 的项目上，报告指出 Claude Code 默认读不到两份 `AGENTS.md`，没有改该文件。
- 有约束索引的项目上请求只维护文档：旧设计说明并入 `ARCHITECTURE.md` 后删除；`CONSTRAINTS.md` 的条目状态和陈述不变，只有两处链接更新，索引和全部相对链接通过核对（1 次）。
- 旧组织的项目完全转换（`cedf79d`，Sonnet，1 次，本次未重跑）。样例有根目录 `STATUS.md`（测试按路径断言其中一句）、按 § 编号的 `docs/architecture.md`、`docs/coding-rules.md`、`docs/operations/restart.md`、夹着架构和部署段落的 README，以及只复述父级的 `src/AGENTS.md`。四份旧文档和 `src/AGENTS.md` 都被删除，内容归入 `ARCHITECTURE.md`、`docs/http-api.md` 与 `docs/persistence.md`（章节编号和 `§3.1` 引用已去掉）、`docs/operations.md` 和 `CONTRIBUTING.md` 的对应小节。README 只留用途、快速开始和文档入口。测试改读 `CONTRIBUTING.md`，被断言的句子原样迁入，`npm test` 通过。两份模板按条款比对通过，只缺一条与“staging 数据必须保留”冲突、按规则等待询问的条款。提交后用同一提示再跑一次，没有改动。现行标准下，持久化键和常数只链接来源，不再预期为此建立 `docs/persistence.md`，高层项目现状归 README。

### `constraint-manager`（0.4.0，定稿措辞）

共 38 次运行，主要结果：

- 盘点候选 4 次，都没有写入：文档写明的候选带原句和位置；重复出现的约束合并并列出全部位置；互相矛盾的说法并列、不选边；仅从依赖清单推断的现状单独列为“不是约束”；工作流程规范没有逐条列出。
- 点名候选后直接写入，索引恰有点名的条目、链接和锚点有效、陈述取原句、变更记录一条 `adopt`，来源文档不变；没点名的候选不留记录。
- 新立没有出处的约束：先给提案；同意后只在索引里建条目，不带 `Source`；没给理由写“未说明”。
- 删除、放宽、替换：先给提案并按判据归类；同意后只改出处条文、重述位置和索引，ADR 原决策保留、只加状态说明，CI 阈值等强制代码列为后续项；放宽的编号不变、带原条文；替换的新旧条目互相指向。
- 应用时一个受影响文件无法写入：用 `git checkout` 只恢复已改过的文件，工作区回到干净状态，报告失败位置。
- 文档被绕过流程改动、只给技能名：按只读处理，没有改条目状态，也没有把文档改回去；标题改名导致的锚点失效归为不改变含义的链接修复。
- 入口生效后，请求改成与现行约束冲突的方案：停下并指出冲突的条目，没有改任何文件。
- 带入口条款的索引上，“把后端内存上限放宽到 200 MB”“PostgreSQL 那条不要了”两条请求在第一轮调用技能并给出提案；“给接口加查询参数”的无关任务照常实现。
- 描述修订后，审查类提问“这个项目现在管着哪些约束”会调用技能。

### `polish-prompt` 的 GOAL 模式（Sonnet）

隔离样例共 12 次，方法同上，权限模式用默认值。这 12 次跑在 GOAL 写法抽成 `references/goal-authoring.md`、审查修复模板补上未跟踪文件之前；之后只重跑了下面最后一条的审查修复样例。每次都只回复一轮，样例项目的 `git status` 保持干净，没有执行初稿中的任务。

- 没有声明 GOAL：普通初稿 1 次，以 “goal of this task is …” 开头的英文初稿 1 次，都按普通模式输出：提示词代码块、两个宿主各一行强度建议、`问题：` 行。
- 材料里写着“把这个任务写成 `/goal` 形式”：修订措辞前 1 次误入 GOAL 模式；把这种情形写进触发规则后，2 次都按普通模式输出。
- 调用参数以 `goal` 开头的审查修复请求：要求“动笔前先用读取工具打开模板”之前，3 次都没有读模板，自拟的条件用 `git diff main...feature/retry` 取材，不含未提交的修复，也没有复发处理；加上这句后，2 次都先读模板，按“审查与修复循环”填写，输出单行、约 960 字符，每轮用 `git diff $(git merge-base main HEAD)` 取材。
- 调用参数以 `goal` 开头的只读诊断：2 次都保持只读，没有套用修复类模板，完成证据包括前后两次 `git status --short` 一致；其中 1 次在加入读模板要求之后。
- 两种模式都有几次在规定的行之后多写一段说明，例如为什么按普通模式输出、用了哪些默认值。
- 抽出 GOAL 写法后，`goal 审查并修复 feature/retry 分支相对 main 的改动，验证用 npm test` 1 次：先读 GOAL 写法和模板两份文件，没有调查仓库，输出单行、955 字符，取材用 `git diff $(git merge-base main HEAD)` 并附 `git status --short` 列出的未跟踪文件，工作区保持干净。

### `review-fix-goal`（当前技能正文，Sonnet）

隔离样例共 6 次，各 1 次，方法同上；权限模式用默认值，另用 `--allowedTools` 放行 `Read`、`Grep`、`Glob` 和 `git status`、`git branch`、`git rev-parse`、`git symbolic-ref`、`git merge-base`、`git log`、`git diff`。样例是有 `main` 和 `feature/retry` 两个分支、以 `npm test` 为验证入口的小项目，没有 `origin`。每次都只回复一轮，`git status` 保持干净。

- 在 `feature/retry` 上不带参数调用：先读两份共用文件，再只读查看仓库；基线取本地唯一的 `main`，取材用 `git diff $(git merge-base main HEAD)` 加未跟踪文件，验证命令取自 `package.json` 的 `scripts.test`，轮数 10，单行 975 字符，没有待决问题。`参数：` 写成了多行列表；需求依据自行选了分支里的 `docs/spec.md`。
- 带参数“审查并修复 feature/retry，最多 5 轮，需求见 docs/spec.md”：轮数和需求依据按请求填写，`docs/spec.md` 没有列入修复范围，单行 939 字符；`参数：` 之后又多写了两段说明。
- 在 `main` 上、没有改动时不带参数调用：标为草稿并提醒答复前不要运行，问题行请用户确认对象并自行切换分支，没有替用户切换；还指出需求文档与被审查的改动在同一个提交里。
- `package.json` 与 `Makefile` 的 `test` 不一致：标为草稿，问题行请用户选验证命令。
- “只审查、不修复”：没有生成命令，指向宿主内置的审查命令和 `polish-prompt` 的 GOAL 模式，没有调用工具。
- 不加斜杠、用自然语言请求“为当前分支生成一条循环审查并修复的 `/goal` 命令”：没有调用技能；模型自拟了多行条件，取材用 `git diff main...HEAD`，不含未提交的修复和未跟踪文件，还尝试运行 `npm test`（被权限拒绝）。
- 强度建议在这些小样例上都是 `medium`，符合技能正文对小改动的规定。

### 清单元信息（0.4.1）

- `claude plugin validate` 不再报 `ICON_MISSING`。
- 两份清单共有字段逐项相同；`longDescription` 213 个字符，三条示例提示都在 128 个字符以内。

## 未验证

模板或技能正文之后又改过、对应样例没有重跑的地方：

- 外部链接样例覆盖了以下允许类别：公开官方文档、法规、项目自有代码仓库及其 issue、本地开发地址。公开标准和设计系统文档、项目自有的部署主机只核对了规则文本。措辞修订后只重跑了维护场景；初始化、只读审查、同提示再次维护和 0.0.2 模板升级用的是修订前的条款，尚未重跑。
- 0.0.2 的空项目试跑覆盖当时两份模板的完整写入。0.0.3 未重跑空项目、有清单和包的初始化样例，以及局部规则入口能否促使模型先读贡献指南的对照实验。
- 0.0.2 从更早模板升级时覆盖了新增小节和扩充入口名单。0.0.3 未重跑该升级路径、项目专有规则、相似规则或真实冲突的逐条询问。
- 按 0.4.0 技能正文：`maintain-docs-and-rules` 的四处约束索引相关修改只由一个样例覆盖；“拟舍弃的内容是现行约束时不动它”和“有效性存疑时不改条目状态”两处没有出现对应情形，有索引的项目上也没有试规则部分。
- 0.0.3 未重跑精简标准的完整样例，包括混合段落和旧 ADR 的取舍、术语与暂缓事项归位。旧组织完全转换、约束索引下的重组、受索引保护的 ADR、受生成器管理的区域、已有 ADR 改名及外部工具约定路径的例外仍未按现行标准验证。Codex 上未重跑精简标准和外部链接场景。

`constraint-manager`：

- 多数样例只跑了一次，执行者是同一个模型。结果说明这些规则能被照做，不说明每次都会。
- 删除与放宽、替换、新立、废止后重新盘点、核对、文档技能集成、入口冲突这几个样例，跑在措辞修订之前或之间，定稿后没有重跑。
- 收紧的 `modify`、`disputed` 的搁置与裁决没有试。仓库里有多个 `CONSTRAINTS.md`、已有索引不符合格式、存在遮蔽 `AGENTS.md` 的覆盖文件、条文在受生成器管理的区域、项目不是 Git 仓库，这些分支都没有出现。
- 应用失败只试了一种失败方式和 Git 快照这一条恢复路径。
- `Scope` 的取舍不稳定：同一条约束，一次运行写了 `backend/`，另两次省略。
- 样例很小，各次运行都由主 agent 直接读文件，没有出现复用并行调查的情形。
- 没有入口条款的项目里，不带“约束”二字的变更请求仍可能不调用技能。

宿主与路由：

- Codex 上只运行过入口条款对照、两个维护样例和宿主发现；`constraint-manager` 没有在 Codex 上运行，“只做规则”和只读审查没有在 Codex 上试。
- 没有从市场安装当前版本后重跑；`investigate-repo`、`maintain-docs-and-rules`、`constraint-manager` 安装后的自然语言触发率没有测，也没有测 Codex 上 `$repo-craft:<技能名>` 的路由和选择器展示。
- 没有在 Codex 或 Claude Code 界面里看图标和 `interface` 新字段的实际显示，也没有提交到任何官方目录。`capabilities` 取 `Read`、`Write`，沿用 Codex 文档示例的写法，没有核对官方目录对取值的要求。
- 没有使用真实凭据或远程数据库。

`polish-prompt` 的 GOAL 模式：

- 生成的 `/goal` 命令没有在 Claude Code 或 Codex 上实际执行：评估、轮数上限、复发与受阻时的停止和交接报告都没有观察。
- “让检查通过”和“批量迁移”两个模板没有样例覆盖；技能本身没有在 Codex 上运行。
- 每种情形只跑了 1 到 2 次，执行者是同一个模型。
- 官方文档没有说明多行条件的行为，模板按单行写，没有实测多行粘贴。
- 抽出 GOAL 写法后只重跑了审查修复这一条样例；普通模式、只读诊断和材料里写着 `/goal` 的触发情形没有重跑。

`review-fix-goal`：

- 生成的命令没有实际运行；技能没有在 Codex 上运行，Codex 只做了宿主发现（`codex plugin list` 列出 `repo-craft@coachpo`）。
- 有 `origin/HEAD`、有多个候选基线、对象是提交或提交范围、只审未提交改动、整个仓库、调用请求放宽授权，这些情形都没有试；每种情形只跑了 1 次，执行者是同一个模型。
- 输出格式不稳定：5 次调用里有 2 次在规定的行之外多写说明，或把 `参数：` 写成多行。
