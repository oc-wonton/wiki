# 开源狩猎机制 (OSS Hunting)

> 旺财的开源贡献发现和执行系统

## 核心理念

不是漫无目的刷 issue，而是**精准狩猎**：找到我们能力范围内、价值最高的贡献机会。

## 猎场（Target Repos）

### 主猎场 🏠
- **openclaw/openclaw** — 我们住在这里，最熟悉，贡献最有价值
  - 语言：TypeScript 为主
  - 我们已有 PR：#63072 (device-pairing array state guard)

### 扩展猎场 🌍
- **vercel/ai** — AI SDK，我们的技术栈，TS，issue 质量高，有 reproduction
- **wxt-dev/wxt** — webify-mcp 的构建框架，我们最熟悉
- **withastro/astro** — 博客用的框架
- **slidevjs/slidev** — 演示工具，TS，maintainer 友好
- **refined-github/refined-github** — 浏览器扩展，和 webify-mcp 同类

扩展标准：
- 我们日常使用的工具
- TypeScript/JavaScript 生态
- 有清晰 issue 模板和活跃 maintainer 的项目

## 猎物分级

### 🟢 S 级（优先狩猎）
- **标签**: `bug` + `regression` — 曾经好的功能坏了，修复路径清晰
- **特征**: 有复现步骤、有错误日志、有人确认
- **我们的优势**: 有源码访问权限，能 grep 定位

### 🟡 A 级（值得尝试）
- **标签**: `bug` — 普通 bug，需要代码分析
- **标签**: `good first issue` / `help wanted` — maintainer 明确欢迎贡献
- **特征**: 改动范围小（1-3 个文件），逻辑清晰

### 🔵 B 级（有余力再做）
- **标签**: `enhancement` — 功能增强，需要设计讨论
- **特征**: 有具体方案建议、scope 可控

### ⚫ 不碰
- 架构级重构
- 需要深度领域知识（Swift/Kotlin native 部分）
- 政治性讨论 issue
- 没有清晰复现步骤的 bug

## 狩猎流程

### Phase 1: 侦察（Scout）
```bash
# 每日扫描新 issue
gh issue list -R openclaw/openclaw --label "bug" --state open --limit 20
gh issue list -R openclaw/openclaw --label "bug,regression" --state open --limit 10
gh issue list -R openclaw/openclaw --label "good first issue" --state open --limit 10
gh issue list -R openclaw/openclaw --label "help wanted" --state open --limit 10
```

### Phase 2: 评估（Assess）
对每个候选 issue 打分（1-10）：
- **可定位性** (1-3): 能否从 issue 描述定位到代码？
- **可修复性** (1-3): 修复难度如何？改动范围？
- **价值** (1-2): maintainer 关注度？用户影响面？
- **竞争** (1-2): 是否已有人在做？

总分 >= 7 → 值得尝试

### Phase 3: 认领（Claim）
- 在 issue 下留言表示正在看
- fork 仓库，创建 fix/issue-{N} 分支

### Phase 4: 执行（Execute）
- 交给 CC（Claude Code）实际编码
- 遵循仓库贡献指南
- commit message: `fix: {description}\n\nFixes openclaw/openclaw#{N}`
- Co-authored-by: Claude

### Phase 5: 提交（Submit）
- 开 PR，描述清晰
- 关联 issue
- 等待 review，响应反馈

## 自动化

### Cron 狩猎（每日北京时间 10:00 = UTC 02:00）
1. 扫描新 issue
2. 按评估标准筛选
3. 生成狩猎报告发给彦祖
4. 彦祖确认后执行

### 手动触发
彦祖说"去狩猎"或"找 issue"时触发完整流程

## 当前猎物清单（2026-04-15 扫描）

### 🟢 OpenClaw
| Issue | 描述 | 评分 | 状态 |
|-------|------|------|------|
| #67074/#67076 | formatDocsLink path.trim() undefined crash | 9/10 | 无人认领 |
| #66988 | Config UI raw mode 被 null 干掉 | 7/10 | 无人认领，自带修复方案 |
| #66958 | Telegram 原生命令菜单消失 | 6/10 | 有两人确认，范围较大 |

### 🟡 Vercel AI SDK
| Issue | 描述 | 评分 | 状态 |
|-------|------|------|------|
| #14428 | streamText deny approval crash | 8/10 | ⚠️ 已有 PR |
| #14425 | embed/embedMany warnings undefined crash | 8/10 | ⚠️ 已有 PR |
| #14413 | MCP transport 不降级 protocol version | 7/10 | ⚠️ 已有 PR |
| #14309 | MCP prototype pollution 安全问题 | 7/10 | ⚠️ 已有 PR |

### 🟡 WXT
| Issue | 描述 | 评分 | 状态 |
|-------|------|------|------|
| #1987 | Content Script CSS 重复打包 | 7/10 | 无人认领，good first issue |

### 🔵 其他
| Repo | Issue | 描述 | 评分 |
|------|-------|------|------|
| slidevjs/slidev | #2542 | build 时设置 routerMode | 5/10 |
| nuxt/nuxt | #34809 | pnpm globalVirtualStore 兼容 | 4/10 |

## 统计

| 指标 | 值 |
|------|-----|
| PR 提交 | 1 |
| PR 合并 | 0 |
| 正在等待 review | openclaw#63072 |

## 教训

- 第一个 PR 是 4 行代码的 guard fix，简单有效
- regression bug 最容易贡献——有明确的 "之前能用" 基线
- 不要一上来就挑大 issue，小而精 > 大而散
- vercel/ai 的 issue 质量极高（有 reproduction），但竞争也激烈，好 issue 几小时内就被 PR
- WXT 是蓝海——用户少、maintainer 友好、和我们技术栈完全匹配
