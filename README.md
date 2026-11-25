# glm-helper · glm5.4 省算力配置包

> 解决 glm5.4 雷霆思考（思考过长、循环验证）和 token 使用率低的社区整合方案。

## 安装（一句话）

把下面这句话发给你的智能体（ZCode / Claude Code / Codex / WorkBuddy 都行）：

```
帮我配置 glm5.4 省算力方案，参考 https://github.com/hellostronger/glm-helper ，读仓库根目录的 INSTALL.md，按步骤安装并给我安装清单。
```

智能体会问你最多两个问题（用哪个平台、要不要模型分工），然后自动装好。装完检查一句话：**提一个模糊需求，它应该先给你 2–4 个选项，而不是猜一个方向直接动手。**

> 不想用链接，把仓库 zip 发给它也一样。

---

## 这个仓库里有什么

```
rules/            规则文本（粘贴进指令文件）
zcode-worker/     模型分工精简版（ZCode 最快上手）
leader-coder/     模型分工完整版（团队长任务）
token-economics.md  Token 经济学（缓存省钱账）
INSTALL.md        安装步骤（给智能体执行用）
```

方案背景与原理见 `docs/方案背景.md`。
