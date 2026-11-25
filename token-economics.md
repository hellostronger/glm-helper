# Token 经济学 · 缓存才是省钱的主战场（L4）

> 来源：群友定价账（2026-11-25）。核心洞察：**贵的是输出，最贵的是"没命中缓存的输入"**。省钱不是少说话，是让上下文稳定、让调用变少、让重活烧在便宜的模型上。

## 一、定价事实（社区口径，以官方计费页为准）

| 套餐 | 等价关系 | 换算 |
|---|---|---|
| GLM V3 plan | 30w 缓存 token ≈ 2w 输出 token | 缓存输入 ≈ 输出单价的 **1/15** |
| GLM V2 plan | 2w 输入 token ≈ 2w 输出 token | 输入与输出同价，缓存差价更敏感 |

由此推出三个结论：

1. **输出占比是真正的健康指标**：总消耗里真正产出内容的占比，目标抬到 **3–5%**。
2. **缓存命中是王道**：主会话上下文每多一次"冷读"（缓存失效后重付全量），等于白烧 15 倍输出的钱。
3. **真实成本 = 上下文长度 × 调用次数**：30w 上下文执行 3 次调用 ≈ 执行一次压缩 + 少说 10 次模型调用的账。

## 二、三板斧（通过约束 AGENTS.md 实现，可直接粘贴）

```markdown
# Token Economics (append to Rule 3)

- **Subagents for everything**: all execution (reads, writes, runs) goes to subagents on the
  fast model. The main session's context must stay stable and cache-hit; a subagent burns its
  own tokens and returns only a summary. The main model's context is the expensive one to grow.
- **One call, batch work**: single model calls that batch reads / writes / actions whenever a
  batch stays verifiable. Real cost = context size × call count — batching cuts the multiplier.
- **Context lock 256k**: keep the main session under a 256k context budget. When nearing it,
  compress once deliberately — a cold re-read after overflow costs 15× an output token.
- **Health metric**: output tokens should be ≥ 3–5% of total billed tokens in a session. If it
  drops below, you are paying for cold reads and repeated calls, not for work.
```

### 1. 万事子 agent

- 主会话只保留：对齐结论、台账、任务书、子 agent 的摘要终报。
- 读文件、改文件、跑命令、走查——全部派出去；token 烧在子 agent（快模型、便宜）上，主上下文不膨胀、缓存持续命中。
- 与 L3 分工规则天然契合：规则三的 worker/coder 就是执行体，本节补充的是**为什么**（成本结构）。

### 2. 一次调用批量读/写/做

- 每次模型调用都会把上下文重付一遍；能一次读完的不要读三次，能一次写完的不要写三次。
- 判断边界：批量以"一次可验证"为限（对应 L2 的动作标注——每一批仍要能标注"服务台账第 N 条 → 预期产出"）。

### 3. 上下文锁 256k

- 给主会话设硬预算：接近上限时**主动做一次压缩**，而不是溢出后反复冷读。
- 30w 上下文连跑 3 次调用的账，等价于一次压缩 + 10 次模型调用——压缩是主动支付的小账，冷读是被动支付的大账。

## 三、与 L1–L3 的关系

| 层 | 管什么 | 本节补充 |
|---|---|---|
| L1/L2 规则熔断 | 减少**无效思考**的 token | 思考量 |
| L3 模型分工 | 让重活烧在**便宜的模型**上 | 单价 |
| L4 token 经济学 | 让**结构**少付钱：缓存命中、调用次数、上下文规模 | 乘数 |

三者相乘才是总账：`总成本 ≈ 上下文 × 调用次数 × 单价`。L1/L2 砍无效内容，L3 换单价，L4 砍乘数。

## 四、注意事项

- 定价等价关系来自群友实测口径，随套餐调整会变，装之前对一遍官方计费页。
- "万事子 agent"以子 agent 跑在快模型为前提；子 agent 若也跑贵模型则本节不成立（这就是 L3 存在的原因）。
- 输出占比 3–5% 是健康线不是硬指标：短会话天然偏低，长会话才是它要防的形态。
