# Hermes Agent 深度架构剖析（基于 v0.8.0）

> 来源: 青雲博客 echovic.com，2026-04-08 版本分析
> 原文标题: "Hermes Agent 深度架构剖析与 OpenClaw 工程化对比"
> 彦祖推荐的微信文章主题相同: "揭秘Hermes Delegate与OpenClaw：多Agent系统中的同步阻塞与异步编排终极对决"

## Hermes 核心架构

### 五层架构
1. **入口与编排层**: CLI (prompt_toolkit TUI) + Gateway (多平台网关)
2. **Agent 核心层**: AIAgent (~3600行), prompt_builder, context_compressor, prompt_caching
3. **工具与注册层**: ToolRegistry 单例, 自注册模式, 工具集组合
4. **状态与持久化层**: SessionDB (SQLite + FTS5), MemoryStore (文件持久化), Cron 调度
5. **平台适配层**: Telegram/Discord/Slack/WhatsApp/Signal/Weixin + ACP + 终端后端

### 关键设计决策

#### 同步循环（非异步）
- 核心瓶颈是 LLM API 延迟，不是 I/O 并发
- 同步循环更容易推理、调试、维护
- 需要并行时用 ThreadPoolExecutor 显式并行
- `_run_async()` 精心处理三种同步-异步桥接场景

#### 工具注册表（最优雅的设计）
- 自注册: 模块导入时调用 `registry.register()`
- 添加新工具只需三步: 创建文件 → 添加导入 → 加入工具集
- 运行时可用性检查: check_fn 让缺依赖时优雅降级
- **动态 Schema 重建**: 避免工具 Schema 里引用不存在的工具导致幻觉（精妙！）
- `max_result_size_chars`: 不同工具不同输出预算，防单个工具淹没上下文

#### 有界策展式记忆
- MEMORY.md (2200字符) + USER.md (1375字符)，字符限制而非 token 限制
- **冻结快照**: 会话开始时冻结，中间写入持久化但不改当前系统提示
  - 精妙：Anthropic 前缀缓存在整个会话期间有效，大幅降成本
- 子字符串匹配操作：比 ID 匹配更适合 LLM 使用模式
- 记忆指导嵌入工具 Schema（非系统提示），跨平台一致

#### 子 Agent 委托
- 每个子 Agent: 全新对话 + 独立 task_id + 受限工具集 + 聚焦系统提示
- 最大深度 2（防递归失控）
- 批量模式最多 3 个并行 (ThreadPoolExecutor)
- 进程全局状态保护: 委托前保存父工具名列表，完毕后恢复

#### 上下文压缩（四阶段）
1. 工具输出修剪（零 LLM 调用）
2. 边界确定（保护头尾）
3. 结构化摘要（辅助 LLM，7段模板）
4. 工具调用对完整性修复
- 迭代更新: 二次压缩在前次基础上增量更新
- 尾部保护用 token 预算（非固定消息数）

### RL 训练集成（独特定位）
- 批量轨迹生成 → JSONL → 微调/RL
- 轨迹压缩: 保护首尾轮次，仅压缩中间，保持训练信号质量
- Atropos RL 环境: 工具调用行为封装为 RL 交互
- Hermes 不只消费 LLM，还生产训练数据（闭环）

## Hermes vs OpenClaw 核心差异

| 维度 | Hermes | OpenClaw |
|------|--------|----------|
| 哲学 | 深度学习循环，自进化 | 全平台产品化 |
| 运行时 | Python + SQLite | Node.js Gateway + WebSocket |
| 编排 | 同步循环 + 委托 | 多 Agent 路由 + 会话工具 |
| 记忆 | 有界策展 + 冻结快照 | 可替换插件 + 工作区文件 |
| 安全 | 默认信任 + 选择性防护 | 默认安全 + 选择性开放 |
| 社区 | 研究导向 (35.7k star) | 产品导向 (352k star) |

### 值得借鉴的模式
1. **动态 Schema 重建** — 移除不存在的跨工具引用，防幻觉
2. **冻结快照记忆** — 前缀缓存成本优化
3. **有界记忆** — 迫使 Agent 学会知识优先级管理
4. **coerce_tool_args()** — LLM 经常返回 "42" 而非 42，防御性强转
5. **进程全局状态保护** — 子 Agent 不污染父 Agent 全局变量

### 微信支持
- Hermes 原生支持微信个人号（通过腾讯 iLink Bot API）
- `gateway/platforms/weixin.py`
- QR 扫码登录，长轮询消息
- CDN 加密媒体传输 (AES-128-ECB)
- 支持文本/图片/语音/文件
- 消息 Markdown 格式化（表格转列表、标题转【】）

## 对我们的启示
- Hermes `hermes claw migrate` 可以一键从 OpenClaw 迁移
- 我们可以在 Studio 上跑 Hermes，在 VM1 上跑 OpenClaw，互补
- Hermes 的技能自创建能力值得关注
- 会话全文搜索 (FTS5) 是我们缺的能力
