# Archon — AI 编码工作流引擎

## 是什么
开源的 AI 编码 workflow engine，用 YAML 定义开发流程（规划→实现→验证→审查→PR），让 AI 编码变得可重复、可控、可并行。作者 Cole Medin 的比喻：Dockerfile 标准化了 infra，GitHub Actions 标准化了 CI/CD，Archon 标准化 AI coding workflow。

- **Repo**: https://github.com/coleam00/Archon
- **官网**: https://archon.diy
- **协议**: MIT

## 解决什么问题
让 AI agent「修个 bug」，每次行为都不一样——可能跳过规划、忘记跑测试、PR 描述乱写。Archon 把流程固定为 YAML，AI 只在需要智能的环节介入，结构由你掌控。

## 核心设计
- **YAML 工作流**: 定义 DAG 节点，每个节点可以是 AI prompt 或 bash 脚本
- **Git Worktree 隔离**: 每个任务独立分支，并行跑多个任务互不冲突
- **循环节点**: 实现→测试→失败→再实现，`until` 条件满足才退出
- **人工审批门**: `interactive: true` 暂停等待人类确认
- **多平台接入**: CLI / Web UI / Telegram / Slack / Discord / GitHub Webhook

## 工作流示例
```yaml
# .archon/workflows/build-feature.yaml
nodes:
  - id: plan
    prompt: "Explore the codebase and create an implementation plan"
  - id: implement
    depends_on: [plan]
    loop:
      prompt: "Read the plan. Implement the next task. Run validation."
      until: ALL_TASKS_COMPLETE
      fresh_context: true
  - id: run-tests
    depends_on: [implement]
    bash: "bun run validate"  # 确定性节点，无 AI
  - id: review
    depends_on: [run-tests]
    prompt: "Review all changes against the plan. Fix any issues."
  - id: approve
    depends_on: [review]
    loop:
      prompt: "Present the changes for review. Address any feedback."
      until: APPROVED
      interactive: true
  - id: create-pr
    depends_on: [approve]
    prompt: "Push changes and create a pull request"
```

## 内置工作流（17 个）
| 工作流 | 用途 |
|---|---|
| archon-fix-github-issue | Issue → 调查 → 实现 → 验证 → PR → 审查 → 自修 |
| archon-idea-to-pr | 想法 → 规划 → 实现 → 5 并行 reviewer → PR |
| archon-smart-pr-review | 分类 PR 复杂度 → 针对性多 agent 审查 |
| archon-resolve-conflicts | 检测 merge conflict → 分析双方 → 解决 → 验证 |
| archon-architect | 架构扫描、复杂度治理、代码健康度提升 |
| archon-piv-loop | Plan-Implement-Validate 循环 + 人工审查 |

## 技术栈
- **运行时**: Bun
- **AI 引擎**: Claude Code（主要）/ Codex
- **存储**: SQLite / PostgreSQL（7 张表）
- **前端**: Web Dashboard（对话、工作流可视化、监控）

## 架构
```
Platform Adapters (Web/CLI/Telegram/Slack/Discord/GitHub)
    ↓
Orchestrator (消息路由 + 上下文管理)
    ↓
┌─────────────┬──────────────┬──────────────────┐
│ Command     │ Workflow     │ AI Assistant     │
│ Handler     │ Executor     │ (Claude/Codex)   │
└─────────────┴──────────────┴──────────────────┘
    ↓
SQLite/PostgreSQL (Codebases/Conversations/Sessions/Workflow Runs...)
```

## 与 OpenClaw 对比
| 维度 | Archon | OpenClaw |
|---|---|---|
| 定位 | AI 编码工作流编排 | 通用 AI agent 运行时 |
| 流程定义 | YAML workflow（类 GitHub Actions） | Skills + ACP（agent 自主决策） |
| 侧重 | 开发流程标准化、可重复 | 多渠道多能力、灵活组合 |
| AI 引擎 | Claude Code / Codex | 任意模型 + ACP harness |

## 可借鉴的点
- YAML workflow 定义开发流程的思路，适合团队标准化
- Git Worktree 隔离实现并行任务，很工程化
- 循环节点 + 退出条件的设计，避免 AI 无限循环
- 人工审批门的设计，关键节点保留人类控制

## 收录时间
2026-04-14
