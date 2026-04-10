# Anthropic Managed Agents — Harness 层被接管

> 来源：Anthropic 官方博客 + 微信公众号「智享科技社」
> 日期：2026-04-09
> 原文：https://www.anthropic.com/engineering/managed-agents

## 这是什么

Anthropic 发布 **Claude Managed Agents**，一套托管式 API，把 Agent 的基础设施层（Harness）全部接管。开发者只定义任务、工具和安全边界，Anthropic 负责运行。

定价：标准 token 费 + 每活跃会话小时 $0.08。

## 核心架构：大脑和双手分离

```
Session（会话）  ← 只增日志，记录所有事件
    ↕
Harness（控制器）← 调用 Claude + 路由工具请求
    ↕
Sandbox（沙箱） ← 执行代码和文件编辑
```

三者解耦，各自独立替换。核心洞察：**不要把所有东西放一个容器里**。

### 为什么解耦

耦合设计的问题：
- 容器变成「宠物」—— 挂了就全丢了
- 调试困难 —— WebSocket 事件流无法定位故障层
- 无法接入客户私有云 —— 控制器假设资源在同一个容器里

解耦后：
- 容器变「牲口」—— 挂了就换一个，用标准配方 `provision({resources})` 重建
- 控制器也是牲口 —— 崩溃后 `wake(sessionId)` + `getSession(id)` 恢复
- 大脑和双手位置无关 —— 私有云、公有云随意接入

### 性能提升
- p50 首 token 延迟 **降 60%**
- p95 首 token 延迟 **降 90%+**
- 原因：不需要沙箱的会话不用等容器启动

## 安全设计

**核心原则：token 永远不进沙箱**

- Git 凭证：初始化时克隆仓库，沙箱内 git push/pull 正常工作但永远碰不到 token
- MCP 工具：OAuth token 存在外部保险库，Claude 通过专用代理调用，代理负责取凭证
- 防提示注入：即使 Claude 被说服执行恶意代码，也无法从沙箱读取环境变量

## 会话 ≠ 上下文窗口

长任务超出上下文窗口时的处理：
- 会话日志是活在上下文窗口**外部**的上下文对象
- `getEvents()` 允许按位置/时间切片查询
- 传入上下文窗口前可做任意变换（压缩、缓存优化）
- **决策权留给控制器，会话只保证持久可查**

## 多大脑 + 多双手

- 一个大脑接多双手：每只手都是 `execute(name, input) → string`
- 多个大脑协调：无状态控制器按需启动
- 大脑之间可以互相传递双手

## 已有客户

| 公司 | 用法 |
|------|------|
| Notion | 团队成员在 Notion 里直接把任务委托给 Claude |
| Rakuten | 跨部门企业级智能体，一周内部署 |
| Asana | AI Teammates 与人类并肩工作 |
| Vibecode | 从提示词到部署上线，速度快 10 倍 |
| Sentry | 调试 → 写补丁 → 发 PR 一气呵成 |

## 对 OpenClaw 的影响和思考

### 重叠的部分
- Agent 编排（Managed Agents 的 Harness ≈ OpenClaw 的 Gateway + Agent 调度）
- 会话持久化（Session log ≈ OpenClaw 的 memory 系统）
- 工具管理（MCP 支持 ≈ OpenClaw 的 skills + MCP）
- 多 Agent 协调（Multi-agent ≈ OpenClaw 的 subagent）

### OpenClaw 的差异化
1. **多模型** — Managed Agents 只能用 Claude，OpenClaw 支持任何模型
2. **本地控制** — 数据在自己机器上，不在 Anthropic 云上
3. **多渠道** — 微信/飞书/Discord/Telegram，Managed Agents 没有
4. **个人助手定位** — 不只是 coding，是生活 + 工作全方位助手
5. **开源** — Managed Agents 是闭源商业产品

### 值得学习的
1. **大脑/双手/会话三层解耦** — 这个架构思想很精辟，OpenClaw 可以参考
2. **容器从宠物变牲口** — 无状态控制器 + 持久化会话 = 高可靠
3. **token 永远不进沙箱** — 安全设计的黄金法则
4. **会话日志作为外部上下文** — 比硬塞上下文窗口优雅得多
5. **`execute(name, input) → string`** — 统一的工具接口，简单但强大

### 启示
Anthropic 正在把 harness 层标准化为云服务。对于只用 Claude 的开发者来说，Managed Agents 会成为默认选择。OpenClaw 的未来在于：**做 Managed Agents 做不了的事** —— 多模型、本地化、个人化、多渠道。
