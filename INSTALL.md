# INSTALL.md · 安装步骤（给智能体执行）

> 触发方式：用户说「帮我配置 glm5.4 省算力方案，参考 …/glm-helper」并把本仓库给你。你按本文件执行。
> 铁律：**覆盖任何已有文件前先备份 `.bak` 并告知位置；全程不 git commit / 不 git push；用户没回答的问题不许擅自替换。**

## 第 0 步：确认两件事（最多问两个问题，都能答就不用问）

1. **平台**：ZCode / Claude Code / Codex / WorkBuddy / 其他（用户没说就问）。
2. **要不要模型分工**：不要 → 只装规则文本（路径 ①）；要 → ZCode 走路径 ②，Claude Code 走路径 ③，Codex/WorkBuddy 无子智能体机制、走路径 ①（规则三自带兜底，分工自动退化为单模型）。

## 路径 ① 只装规则（所有平台通用，5 分钟）

1. 把 [`rules/discuss-before-diverging.md`](rules/discuss-before-diverging.md) **整篇**追加到指令文件**顶部、项目专属规则之前**。
2. 用户重度单模型使用时，再加整篇 [`rules/progress-over-polish.md`](rules/progress-over-polish.md)，紧随其后。
3. 算力敏感时，把 [`token-economics.md`](token-economics.md) 第二节的英文片段追加在规则之后。

指令文件位置：

| 平台 | 全局 | 项目级 |
|---|---|---|
| ZCode | `~/.zcode/AGENTS.md` | `<项目>/AGENTS.md` |
| Claude Code | `~/.claude/CLAUDE.md` | `<项目>/CLAUDE.md`（或加一行 `@<本文件绝对路径>` import，免复制） |
| Codex | 项目根 `AGENTS.md`（或全局指令） | 同左 |
| WorkBuddy | 自定义指令 / 系统提示词设置 | 同左 |
| Cursor | `~/.cursor/rules/discuss-before-diverging.mdc`（顶部加 `alwaysApply: true`） | — |
| Antigravity | `~/.gemini/config/GEMINI.md` 最顶部（防超 24KB 尾部截断） | — |

## 路径 ② ZCode 精简版分工（10 分钟）

1. 规则部分执行路径 ①（`zcode-worker/AGENTS.md` 本身自包含规则一+二+三，可整篇替代）。
2. `cp zcode-worker/worker.md ~/.zcode/agents/worker.md`（项目级 `.zcode/agents/` 已有同名定义则提醒用户二选一，项目级会盖过用户级）。
3. **必改**：`worker.md` 的 `model` 字段——模型 ID 与套餐绑定，列出用户可用的模型 ID 让用户选，原样填入。
4. 提醒用户新建会话或 `/reload`；输入框打 `@` 应能看到 `worker`。

## 路径 ③ Claude Code（含分工）

1. 规则部分执行路径 ①（推荐 `@路径` import：`@<仓库绝对路径>/rules/discuss-before-diverging.md`）。
2. 分工：`cp leader-coder/coder.md ~/.claude/agents/coder.md`（精简版可换 `zcode-worker/worker.md`），然后改 frontmatter：`model` 换 `haiku` 或删掉；**删掉** ZCode 专有的 `thoughtLevel`、`injectAgentsMd`。
3. Leader 技能（可选，团队长任务）：`cp leader-coder/leader-SKILL.md ~/.claude/skills/leader/SKILL.md`，新会话后 `/leader` 触发。
4. 提醒用户新开会话生效。

## 路径 ⑤ 完整版 Leader/Coder（团队长任务，用户点名才装）

按 [`leader-coder/install.md`](leader-coder/install.md) 执行。关键：`coder.md` 里的 `<占位符>`（数据根、隔离环境变量、测试命令、模型名）逐条问用户后替换，**没回答的不许擅自改**。

## 装完：输出安装清单

- 每个文件装到了哪里、改了什么、备份在哪；
- 哪些改动需要新建会话才生效（ZCode 全部；Claude Code 的子智能体与技能）；
- 哪些在等用户确认（如模型 ID）。

## 生效验证（给用户的最后一句话）

提一个有多种理解的需求（如「优化一下登录」）——模型应该**先给 2–4 个选项**让你选，而不是猜一个方向直接动手。达到即安装成功。

## 降级与兜底

- Codex / WorkBuddy 无子智能体机制：只装规则文本即可，L1/L2 收益完整，分工部分按规则三兜底由主模型自己干。
- 用户只想试效果：只执行路径 ① 第 1 步。
