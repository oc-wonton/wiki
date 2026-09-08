# Loop Engineering 阅读清单

更新时间：2026-09-08。

## 必读

1. [Loop engineering: Getting started with loops](https://claude.com/blog/getting-started-with-loops) — Claude Code 团队对 Loop Engineering 最直接的定义，区分 turn-based、goal-based、time-based 和 proactive loops，并强调停止条件、验证、模型选择与 token 边界。
2. [Building Effective AI Agents](https://www.anthropic.com/engineering/building-effective-agents) — 解释 workflow 与 agent 的区别，以及 prompt chaining、routing、parallelization、orchestrator-workers、evaluator-optimizer 等基础模式。Loop 设计的架构底座。
3. [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — 长时 Coding Loop 的具体做法：initializer、feature list、progress file、Git 状态、一次只做一个 feature、每轮恢复基本功能和留下干净交接。
4. [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — 讲如何判断 Loop 是否真的变好：task/trial/grader/transcript/outcome，能力集与回归集，代码、模型和人工 grader 的组合。

## 组织与工程实践

5. [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) — 把 Loop 放入完整 SDLC：规划、实现、测试、Review、部署和维护，以及人工审批和治理边界。
6. [QQ 飞车 Agentic 研发转型过程中的 Loop Engineering](https://mp.weixin.qq.com/s/FTUowOWA8kbzGJ21DA4PoQ) — 中文一线案例：Hook Linter 自愈、CI/MR 闭环、事前/事中/事后、Graph Engineering，以及 4 万行/27 轮 Review 的无界 Loop 失败案例。

## 思想实验与反面教材

7. [Ralph Wiggum as a “software engineer”](https://ghuntley.com/ralph/) — 最极端的循环式 Coding Agent 实践：单进程、每轮只做一件事、spec/plan 反复注入、测试与类型系统形成 backpressure。适合理解 Loop 的最小原型，但其“最终一致性”、超大并发和无限运行理念风险很高，不能直接照搬到生产。

## 建议阅读顺序

- 只读三篇：1 → 3 → 4。
- 想做 Coding Loop：1 → 2 → 3 → 4 → 7。
- 想做组织流程：1 → 5 → 6 → 4。

## 共同结论

好的 Loop 必须有：明确触发条件、可验证目标、环境反馈、有限修复轮次、成本/时间/权限边界、失败升级点和可持久化的学习结果。只有重复调用 Agent、没有验收和停止条件的，不是 Loop Engineering，只是自动化重试。
