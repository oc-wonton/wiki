# GitHub Trending 深度研究 2026-04-10

## Hermes Agent（重点研究）

> NousResearch/hermes-agent | 48K stars | 本周 +19,765

**定位**：Nous Research 出品的「自我进化的个人 AI 助手」

### 核心架构

| 概念 | Hermes Agent | OpenClaw |
|------|-------------|----------|
| 语言 | Python | Node.js/TypeScript |
| Skills | 自动创建 + 自我改进 | 手动/辅助创建 |
| Memory | FTS5 会话搜索 + Honcho 用户建模 | 文件系统 memory |
| 终端后端 | 6 种（local/Docker/SSH/Daytona/Singularity/Modal） | 主要本地 + SSH |
| RL 训练 | 内置 trajectory 压缩 + Atropos RL 环境 | 无 |
| ACP | 有 acp_adapter | 原生 ACP |

### 杀手级特性

1. **自动学习闭环** — 任务完成后自动从经验创建 skill，使用中自我改进，定期持久化知识。这是最大差异化
2. **FTS5 会话搜索** — 跨 session 全文检索历史对话，比翻 memory 文件强太多
3. **Serverless 后端** — Modal/Daytona 空闲零成本，对成本敏感场景很有意义
4. **Honcho 用户建模** — 辩证式建模，比静态 USER.md 更能捕捉偏好变化
5. **Trajectory 压缩 + RL** — Nous Research 用它训练下一代 tool-calling 模型，agent 本身也是数据工厂

### 与 OpenClaw 的渊源

**内置 `hermes claw migrate` 命令**，直接暴露了两者的关系。Hermes 可能 fork 自 OpenClaw 或深度参考了其架构。

### 对我们的启发

- **自动 skill 创建**：最值得借鉴。OpenClaw 的 skill-creator 需要人工触发，如果能在复杂任务后自动提示「要不要保存为 skill？」会很有价值
- **会话搜索**：比纯文件 memory 更强，FTS5 索引成本低效果好
- **Serverless 后端**：Modal/Daytona 的 hibernate-on-idle，未来可以考虑

---

## Multica

> multica-ai/multica | 5K stars | 本周 +3,201

**定位**：开源 managed agents 平台，把 coding agent 变成真队友

**核心思路**：不是又一个 agent 框架，而是 agent 的「项目管理工具」——分配任务、追踪进度、积累技能。类似 Jira 但管理对象是 AI agent 而不是人。

**对我们的启发**：跟 OpenClaw 的 subagent 管理有相似之处，但它的任务追踪和技能积累 UI 做得更直观。

---

## Oh My Codex (OmX)

> Yeachan-Heo/oh-my-codex | 20K stars | 本周 +9,737

**定位**：Codex 增强框架——hooks、agent teams、HUD

**核心思路**：给 OpenAI Codex 加上插件生态。Hooks 在关键节点注入自定义逻辑，Agent Teams 让多个 Codex 协作，HUD 提供可视化面板。

**独特设计**：hook 系统（before/after task, on error 等），类似 Git hooks 的思路但用在 agent 上。

**对我们的启发**：Hook 机制可以借鉴到 OpenClaw 的 skill 系统。比如 on-task-complete hook 自动触发 skill 创建。

---

## Andrej Karpathy Skills

> forrestchang/andrej-karpathy-skills | 11K stars | 本周 +2,230

**定位**：一个 CLAUDE.md 文件，基于 Karpathy 对 LLM coding 陷阱的观察

**核心内容**：不是代码项目，是一份精心编写的 agent 行为指南。核心观点：
- LLM 倾向于过度工程化，需要明确约束「保持简单」
- 测试驱动：先写测试再实现
- 渐进式：小步提交，不要大爆炸重构
- 明确的反模式清单

**对我们的启发**：我们的 AGENTS.md / SOUL.md 可以参考它的结构。11K star 说明好的 agent 指导文件本身就有巨大价值。也可以考虑发到 clawhub.ai 作为 skill。
