# Leader / Coder 双模型分工 · 安装说明

> 一键安装方式：把本目录（`leader-coder/`）连同路径告诉你的智能体，让它按下面步骤执行。

## 安装步骤（交给你的智能体执行）

1. 先读本目录的 `leader-SKILL.md`（Leader 技能，中文）与 `coder.md`（Coder 子智能体模板，中文），然后说明你运行在什么平台上（Claude Code、Codex、Cursor、ZCode 等），后续步骤按平台执行。
2. **安装 Leader 技能**：
   - ZCode：把 `leader-SKILL.md` 原样放到 `~/.agents/skills/leader/SKILL.md`。
   - 其他平台：换成所在平台的技能格式再放，并向用户说明改了什么。
3. **安装 Coder 子智能体**：
   - ZCode：把 `coder.md` 原样放到 `~/.zcode/agents/coder.md`。
   - 其他平台：把正文作为系统提示词，换成所在平台的子智能体格式再放，并说明改了什么。注意 ZCode 下若项目级 `.zcode/agents/coder.md` 已存在，它会盖过用户级这份，**两处只能留一份**。
4. **追加规则片段**：`leader-SKILL.md` 已内置「分叉前先对齐」一节，无需再贴；若你的项目也想让非 Leader 会话生效，可把 [`../rules/discuss-before-diverging.md`](../rules/discuss-before-diverging.md) 整篇追加到项目说明文件顶部、项目专属规则之前（Claude Code 是 `CLAUDE.md`，其余通常是 `AGENTS.md`）。目标文件里已有同名小节则跳过。
5. **覆盖任何已存在的目标文件之前**，先复制一份 `.bak` 备份，并告诉用户备份文件在哪里。
6. **替换 coder.md 的模板占位符**：`<...>` 标出的地方是按作者项目调过的（数据根、隔离环境变量、测试命令、模型名），需要按用户项目逐条替换。把这些需要替换的地方逐条列出来问用户，按用户回答修改；**用户没有回答的不要擅自改**。每条规矩背后的道理可以迁移，具体写法不能。
7. **全程不要 git commit、不要 git push。** 装完输出一张清单：每个文件装到了哪里、改了什么、哪些在等用户确认；并提醒：ZCode 下智能体定义要到**下一个会话**才生效。

## ZCode 下的落位

```
~/.zcode/agents/coder.md                              # 来自 coder.md
~/.agents/skills/leader/SKILL.md                      # 来自 leader-SKILL.md
<项目>/AGENTS.md                                      # 追加规则片段（可选，Leader 已内置）
```

## 三点要知道

1. 智能体定义在会话启动时加载。新建或修改的定义要到**下一个会话**才生效。
2. 项目级 `.zcode/agents/coder.md` 会盖过用户级那份。只留一份。
3. `coder.md` 是模板，安全规矩和测试配方必须按项目替换。读一遍安全规矩那一节再改。
4. Claude Code、Codex、Cursor：两个文件都是纯 Markdown，可以直接读。子智能体 frontmatter 的头部字段（`model`、`thoughtLevel`、`tools`）是 ZCode 的，需要换成你的工具的子智能体格式。
