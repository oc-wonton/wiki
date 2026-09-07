# 阅读：The Claude Code Guide for Startups

原文：<https://claude.com/blog/claude-code-guide-for-startups>

Anthropic 访谈十余家高速成长的初创公司，总结出五条 AI-native 开发原则。文章不是 Claude Code 功能教程，而是一份组织与工程流程指南。

## 五条原则

### 1. Everyone ships

Agentic coding 降低从想法到可运行原型的门槛，使最理解业务问题的人可以制作第一版，但不取消专业分工。市场、法律、临床等角色负责把领域知识变成原型；工程师仍负责产品化、集成和质量。

组织需要让这种参与制度化：连接员工日常工具；建立 prototype showcase 和进入正式 roadmap 的渠道；用根级规则、目录级约束和按需 Skill 共享团队标准。

### 2. Automate the tedium

Agent 承担机械性的 80%，人集中处理需要判断的 20%。成熟团队把 Agent 嵌入整个 SDLC，而不是只用于生成代码：新员工环境搭建、并行 ticket/PR、代码审查、测试、CI、bug triage，以及数据分析和文档处理。

代表案例：ClickHouse 用专门 Agent 修 flaky tests、找测试缺口并处理运维；Commure 用多个 subagent 并行推进约 13 个 ticket；多个团队使用自定义 reviewer、analytics agent 和面向业务数据的自动分析流程。

### 3. Trust, but verify

不能可靠监控和验证的流程，不应自动化。高监管团队没有让 Agent 直接合并主干，而是把人工审计、确定性检查、版本化规则、golden eval set 和回归检测放进闭环。

Cainex 的关键经验是“修原则，不修样例”：领域专家纠正模型后，Claude 修改产生错误的指导原则，再用 golden set 和随机样本回测；避免逐例打补丁导致过拟合。每个关键 Agent 用例都应维护多套 eval，并持续更新。

### 4. Build for rebuilding

模型能力持续变化，系统不应被当前模型的限制永久塑形。团队应把重建视为竞争力：用 worktree 并行运行 v1/v2，用 eval 对比，只有新版本胜出才合并；重建完成的定义包括删除旧路径，而不只是上线新路径。

对非平凡重写，先在只读计划阶段探索和设计，尽早发现架构偏移。

### 5. Prototype, dogfood, productionize

先用 Coding Agent 构建内部 Agent，在真实团队中 dogfood，再将验证过的流程产品化。构建 Agent 的过程让团队更了解模型能力与 harness 设计，也帮助区分产品问题究竟来自模型还是 harness。

## 判断

文章价值最大之处不是“让所有人写代码”，而是把 AI-native 组织拆成三类机制：

1. **降低原型门槛**，但保留产品化责任；
2. **自动化机械流程**，但建立可验证闭环；
3. **降低重建成本**，让架构能随模型能力演进。

“团队像十倍规模一样交付”带有营销色彩，案例多为成功样本，也没有量化失败率、返工率和安全成本。文章中的并行 Agent、自动 Review 和自改进闭环只有在任务边界清晰、测试与 eval 足够强时才可靠。若没有这些约束，速度提升可能只是把验证成本推迟到后面。

## 对当前体系的启示

- OpenClaw 负责协调、信息和可观察性；OMP 负责实现，保持职责分离。
- AI-Native SDLC Skill 的 `intent → spec → plan → build handoff` 正好对应“先约束再自动化”。
- 根级规则只放不可违反的架构、安全和流程边界；可复用操作放 Skill，避免规则文件无限膨胀。
- 并行 Agent 应以单 ticket、独立 worktree、明确验收标准为边界，不能只追求并发数量。
- 每个重复 Agent 流程都需要 golden set、随机样本回归、人工升级点和最终验收人。
- 重建必须同时包含旧路径清理、回滚方案和 v1/v2 对比证据。
- 内部 Agent 先 dogfood，稳定后再变成用户功能；不要把演示成功直接当作生产可用。
