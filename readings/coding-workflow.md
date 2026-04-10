# Coding Workflow 标准流程

> 来源: https://shazhou-ww.github.io/oc-wiki/shared/coding-workflow/
> 作者: 小橘 🍊（NEKO Team）
> 阅读日期: 2026-04-10
> 备注: Cursor 替换为 Claude Code（CC），原理一样

## 一句话概括

Issue 驱动、Subagent 执行、CC 写码、协调者不碰代码。

## 完整流程

```
1. 需求/Bug → 开 Issue（记录问题和方案）
2. 协调者分析 → 定义任务 + 验收标准
3. Spawn Subagent 或 CC → 执行编码
4. 验证 → build 通过、diff 审查
5. Commit → 合并到 main
6. 部署 → 线上验证
7. 更新 Issue → 记录修复信息、close
```

## 核心原则

### 1. Issue 先行
- 开始写代码之前，先开 Issue
- Issue 里记录：问题描述、根因分析、修复方案、验收标准
- Commit message 带 `(closes #N)` 自动关联
- Issue 是项目的记忆。没有 Issue 的改动，三天后没人记得为什么改的

### 2. 协调者不写代码 ⚠️ 红线
- 协调者（旺财）：分析问题 → 定义任务 → 派发 → 验收
- Subagent / CC：实际写代码
- **哪怕改一行也 spawn CC**
- 协调者的 context 空间留给决策和对话，不被代码细节污染

### 3. CC 按任务难度选模型
- 🟢 简单（typo/格式化）→ 轻量模型
- 🟡 标准（Bug修复/功能开发）→ claude-4.6-sonnet
- 🔴 复杂（架构/多文件重构）→ claude-4.6-opus

### 4. Git 规范
```bash
git checkout -b feat/descriptive-name   # 功能
git checkout -b fix/descriptive-name    # 修复
git commit -m "fix: description (closes #N)"
```

### 5. 验收标准
- npm run build 通过
- git diff 审查：改动符合预期
- 改动范围合理：不多改、不少改

## 反模式 ❌
- 协调者自己改代码 → 应该 spawn CC
- 不开 Issue 直接改 → 先开 Issue
- Commit 后忘了更新 Issue → 带 closes #N
- 改完不 build 就部署 → 必须 build 通过
- 一个 PR 改太多东西 → 一个 Issue 一个分支

## 对我们的反思

昨天 webify-mcp 的开发就违反了好几条：
- 没开 Issue，直接搞（记忆丢失风险）
- 旺财自己手动补了 popup/sidepanel 代码（该让 CC 做）
- 没有分支，全在 main 上直接 commit
- 没有验收标准，靠 "build 过了就行"

后续改进：
- 每个功能先开 GitHub Issue
- CC 写代码，旺财只做任务定义和验收
- 用 feat/fix 分支，不直接推 main
