# Hermes Agent vs OpenClaw 对比分析

> 研究日期：2026-04-10
> Hermes Agent 版本：v0.8.0 | 仓库：https://github.com/nousresearch/hermes-agent

---

## Hermes Agent 是什么？

Nous Research 出品的开源 AI Agent，定位「自我进化的个人 AI 助手」。核心卖点：

- **学习闭环**：从经验中自动创建 Skills，使用中自我改进，定期提醒自己持久化知识
- **多平台入口**：CLI (TUI) + Gateway（Telegram、Discord、Slack、WhatsApp、Signal）
- **多模型支持**：OpenRouter、OpenAI、Anthropic、GLM、Kimi、MiniMax 等，`hermes model` 一键切换
- **多终端后端**：Local、Docker、SSH、Daytona、Singularity、Modal（serverless）
- **90+ 内置工具**：浏览器、文件操作、代码执行、Home Assistant、图片生成、TTS 等

Python 项目，MIT 协议。

## 架构和核心概念

| 概念 | Hermes Agent | OpenClaw |
|------|-------------|----------|
| 语言 | Python | Node.js/TypeScript |
| CLI | prompt_toolkit TUI (`cli.py`) | `openclaw` CLI |
| Gateway | 内置多平台 gateway | 内置多平台 gateway |
| 配置 | `~/.hermes/config.yaml` + `.env` | `~/.openclaw/` |
| Skills | agentskills.io 标准，自动创建 + 自我改进 | AgentSkills 标准，手动/辅助创建 |
| Memory | 内置 memory tool + FTS5 会话搜索 + Honcho 用户建模 | 文件系统 memory（MEMORY.md、daily notes） |
| Subagent | 内置 delegate tool | sessions_spawn + subagents |
| Cron | 内置 cron scheduler | 内置 cron |
| ACP | 有 acp_adapter，支持 ACP 协议 | 原生 ACP 支持（acpx） |
| 终端后端 | 6 种（local/Docker/SSH/Daytona/Singularity/Modal） | 主要本地 + SSH |
| 模型切换 | `hermes model` 运行时切换 | 配置 + `/model` 命令 |
| RL 训练 | 内置 trajectory 压缩 + Atropos RL 环境 | 无 |

## 相似点

1. **同一赛道**：都是「个人 AI 助手」，强调多平台接入 + 持久记忆 + 工具调用
2. **Gateway 模式一致**：CLI + 消息平台（Telegram/Discord/WhatsApp 等）共用一个 gateway 进程
3. **Skills 系统兼容**：都支持 agentskills.io 标准
4. **Subagent 机制**：都能 spawn 子代理并行工作
5. **Cron 调度**：内置定时任务
6. **SOUL.md / Memory**：都有持久化人格和记忆的概念
7. **OpenClaw 迁移**：Hermes 内置了 `hermes claw migrate`，说明它直接从 OpenClaw 演化/fork 而来

## 关键不同

| 维度 | Hermes Agent | OpenClaw |
|------|-------------|----------|
| **出身** | Nous Research（AI 研究公司） | 社区/独立项目 |
| **自我学习** | 核心特色：自动从经验创建 skill、自我改进、FTS5 会话检索 | 靠人工维护 wiki/memory |
| **用户建模** | 集成 Honcho 做辩证用户建模 | 简单的 USER.md |
| **研究导向** | trajectory 压缩、RL 训练管线、batch generation | 无 |
| **终端后端** | 6 种，含 serverless（Modal/Daytona，空闲零成本） | 主要本地 |
| **工具数量** | 90+（浏览器、HA、图片生成、TTS 等） | 依赖 skills + MCP |
| **TUI** | 完整 prompt_toolkit TUI（多行编辑、自动补全、流式输出） | 无独立 TUI |
| **语言** | Python → 对 ML/研究生态友好 | Node.js → 对 web/infra 生态友好 |

## 各自优劣

### Hermes Agent 优势
- **学习闭环**是杀手级特性：自动创建 skill + 自我改进 + 会话搜索，减少重复劳动
- **研究就绪**：trajectory 压缩 + RL 训练，适合训练下一代 tool-calling 模型
- **Serverless 后端**：Modal/Daytona 让 agent 空闲时几乎零成本
- **工具生态丰富**：90+ 工具开箱即用
- **Nous Research 背书**：有研究团队持续迭代

### Hermes Agent 劣势
- Python 项目，依赖较重（看 pyproject.toml 依赖列表）
- 比较新（v0.8.0），稳定性待验证
- 功能堆叠多，复杂度高

### OpenClaw 优势
- Node.js 生态，部署轻量
- ACP 协议原生支持成熟
- 架构简洁，易于理解和定制
- 社区驱动，灵活度高

### OpenClaw 劣势
- 缺少自动学习闭环（技能创建、自我改进）
- 终端后端较少
- 无研究/训练管线
- 用户建模较简单

## 值得学习和借鉴的

1. **自动 Skill 创建**：任务完成后自动提取可复用 skill，这是最值得借鉴的。OpenClaw 的 skill-creator 需要人工触发，如果能在复杂任务后自动提示「要不要保存为 skill？」会很有价值
2. **FTS5 会话搜索**：跨会话全文检索，让 agent 能搜索过去的对话。比纯文件 memory 更强
3. **Serverless 终端后端**：Modal/Daytona 的 hibernate-on-idle 模式，对 VPS 成本敏感的场景很有意义
4. **Honcho 用户建模**：辩证式用户建模比静态 USER.md 更能捕捉用户偏好变化
5. **Trajectory 压缩**：即使不做 RL 训练，压缩 trajectory 也能优化 context 使用
6. **`hermes claw migrate`**：说明两个项目有共同基因，Hermes 可能 fork 自 OpenClaw 或深度参考了其架构。这个迁移工具本身也说明了生态互通的思路

## 结论

Hermes Agent 本质上是 OpenClaw 的「研究增强版」——相同的个人助手赛道，但加了自动学习、RL 训练、更多终端后端。它的 `hermes claw migrate` 命令直接暴露了两者的渊源关系。

对 OpenClaw 来说，最值得吸收的是**自动 skill 创建**和**会话搜索**——这两个特性能显著提升 agent 的长期实用性。
