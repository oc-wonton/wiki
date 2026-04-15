# Skill vs 文档 — 知识在循环中活起来

## 来源
- 作者: touchskyer（摸天人说）
- 日期: 2026-04-14
- 原文: 微信公众号《文档写完就落灰了，Skill 越用越活》

## 核心观点

### 文档 vs Skill 的本质区别
- 文档是 write-once-read-maybe-update-never
- Skill 每次执行都在接受验证 — 跑不通就知道过时了
- **Skill 是 intent，不是 procedure** — 写目标不写步骤，agent 自己决定 how
- Skill 是 human-agent contract 的持久化形式
- 三个属性: Explicit（白箱可审计）、Portable（跨 agent）、Auditable（git log）

### 人的角色变了: author → curator
- 不是人写 Skill 给 agent 用，而是 agent 写、人审批
- Friction 从「花 30 分钟写」降到「花 30 秒看一眼」
- 第二大脑这次能成，因为写的人变了

### Skill Marketplace 必然失败（三重否定）
1. **Supply**: reproduction cost ≈ 0，markdown 没法收费
2. **Demand**: agent 能自动 fork + 适配，不需要逛商店
3. **Coupling**: 环境/版本/上下文三层耦合，不可能开箱即用
- 实证: GPT Store 就是教科书级失败

### Fork 即流动
- Fork 继承的不是代码，是 **decision trace**（为什么做这个选择）
- Skill 三层模型:
  - **Commodity**: "summarize this" — 会被模型吸收
  - **Configurable**: template + 参数 — 社区开源解决
  - **Bespoke**: 深度个人化 — fork + 持续迭代，decision trace 是核心价值

### Knowledge Loop
```
Human → Skill File → Agent → 执行结果 → Human (curate) → 更好的 Skill
```
每次循环知识都在进化。文档做不到，因为文档不参与执行。

### 安全
- 三类威胁: tool poisoning / rug pull / description shadowing
- NL API surface 比 REST API 更难 static analysis
- 个人开发者的安全模型: **local-first**（你控制文件 > 你信任平台 > 你信任作者）

## 对旺财的启示
1. wiki/cards 就是 Skill library，但要更 intent 化，加 decision trace
2. 做完任务主动提炼 pattern 成 Skill，不等彦祖提醒
3. 执行反馈闭环 — Skill 执行失败了自动更新

## 金句
> 冷启动从来不是问题。迭代才是。
> 别预测未来，去用。从今天开始，让你的 agent 帮你写第一个 Skill。

## 收录时间
2026-04-14
