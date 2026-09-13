# OpenViking 项目评估

项目：<https://github.com/volcengine/OpenViking>

评估时间：2026-09-13；源码快照：`46129f143a21d6902791246d2cb1a62bb43d4858`；最新稳定版：`v0.4.19`。

## 结论

OpenViking 不是换皮向量库，而是一套面向 Agent 的“上下文数据库”：把 Resource、Memory、Skill 统一映射为 `viking://` 虚拟文件树，摄取时生成目录级摘要，查询时先找目录再递归下钻，按 token 预算渐进加载；会话提交时归档消息并异步抽取长期记忆。

方向有价值，尤其适合跨大量文档、长会话和多种上下文类型的 Agent。但项目仍处于高频变化的 `0.4.x` 阶段，锁、队列、Session 边界、检索延迟、记忆抽取成本和本地模型兼容仍有公开严重问题。适合隔离 benchmark，不适合现在替换 OpenClaw Memory/Wiki 作为权威层。

## 解决的问题

传统 Agent 上下文通常分裂成三套：向量 RAG 管知识库、Memory 产品管用户/经历、Skills 目录管流程。它们有不同标识、检索 API、权限和加载策略；扁平 chunk 检索还会丢失文档结构，并一次塞入过多正文。

OpenViking 的主张是：

1. 用类文件系统目录统一管理 Memory、Resource、Skill；
2. 保留目录和文档结构，而不是只存离散 chunk；
3. 用 L0/L1/L2 逐级加载，减少无关 token；
4. 把 Session 归档、摘要和长期记忆抽取纳入同一生命周期；
5. 让内容可浏览、可 grep/glob/read，而不仅是不可解释的向量命中。

## 数据模型与架构

### 统一 URI

典型根：

- `viking://~/memories/`
- `viking://resources/`
- `viking://~/skills/`
- `viking://~/sessions/<id>/`

FSService 提供 `ls/mkdir/rm/mv/tree/stat/read/abstract/overview/grep/glob`；SearchService 提供 `find/search`；SessionService 管会话与 commit；ResourceService 管资源/Skill 导入；PackService 管导入导出、备份恢复。

### L0/L1/L2

- L0：很短的 abstract，用于检索结果与快速判断。
- L1：overview，提供目录或文件的结构化概要。
- L2：完整正文/原始资源。

输入文档先解析为目录树和内容，再由 SemanticQueue 自底向上异步生成 L0/L1 和 embedding。目录摘要让检索可以先定位“哪棵树”，再进入子目录，而不是全库扁平 Top-K。

### 双层存储

- AGFS/VikingFS：内容真源，保存目录、L0/L1/L2、多媒体和 Session 归档。
- Vector Index：只保存 URI、向量和元数据，不保存正文。

这种单一内容真源比“向量库里复制一份正文”更容易浏览和迁移，但内容/index 的一致性、异步队列和锁成为新的复杂边界。

## 数据流

### 摄取

```text
PDF/HTML/Markdown/代码/Skill
→ Parser
→ TreeBuilder
→ AGFS
→ SemanticQueue
→ 自底向上生成 abstract/overview
→ Embedding / Vector Index
```

解析阶段可不调用 LLM；摘要和语义生成异步执行。资源可在完成前返回任务 ID，调用方需要等待处理状态。

### 检索

`find()` 是低延迟单查询；`search()` 会使用 Session 摘要、最近五条消息和当前请求，让 LLM IntentAnalyzer 生成 0–5 个按 Memory/Resource/Skill 分类的 TypedQuery。

随后：

```text
全局向量搜索目录入口
→ Rerank 入口
→ 优先队列递归搜索子目录
→ 分数传播与三轮收敛判断
→ 返回 MatchedContext(L0 abstract + URI + score)
→ 调用方按需要读 L1/L2
```

Rerank 当前官方文档列出的后端主要是火山引擎 Doubao Seed Rerank；失败时退回向量分数。复杂 `search()` 比 `find()` 多一次查询规划和多轮目录检索，成本与延迟显著更高。

### Session 与 Memory

Session 生命周期为 Create → Add Message/Tool/Context Parts → Commit。

Commit 分两阶段：

1. 同步把当前消息写入 `history/archive_N/messages.jsonl`、清空活跃消息并返回 task ID；
2. 后台生成 archive abstract/overview、按 MemoryType schema 抽取和去重长期记忆、写 `memory_diff.json` 和 `.done`。

长期记忆目录包含 profile、preferences、entities、events、cases、trajectories、experiences 等。记忆更新支持 schema、merge policy 和审计 diff，但抽取仍依赖 LLM，因此不是事实数据库。

## 部署与接入

- Python >=3.10，AGPL-3.0。
- HTTP Server、Python SDK、Go SDK、CLI、MCP endpoint、LangChain retriever。
- 可配置 Embedding、VLM/query planner、rerank、向量后端；本地向量索引和火山 VikingDB 路径并存。
- 仓库提供通用 Agent plugin（MCP + `openviking-memory`/troubleshoot Skills），不是 OpenClaw 核心内置模块。OpenClaw 集成仍需插件/Hook 将消息、reset、commit 和 recall 生命周期正确映射。

## 与其他方案的区别

### 普通向量 RAG

普通 RAG：文档切 chunk → embedding → Top-K。简单、成熟、低延迟，但结构丢失，跨章节证据弱。OpenViking 增加目录摘要、层级检索和渐进读取，适合结构化大文档；代价是摄取 LLM 成本、队列和一致性复杂度。

### GraphRAG

GraphRAG 抽取实体/关系并沿知识图谱检索，适合关系推理但建图昂贵。OpenViking 的主结构是目录树和 URI，不要求完整实体图，工程更直观；对非树状关系的表达通常弱于专门 GraphRAG。

### Mem0

Mem0 聚焦从会话抽取用户事实/偏好并做向量检索，接入轻。OpenViking 管理范围更广：资源、Skill、会话归档与文件操作都在同一层；部署和治理明显更重。

### Letta/MemGPT

Letta 更像带工作记忆/归档记忆和自编辑记忆的 Stateful Agent Runtime。OpenViking 更偏独立 Context Database，可被不同 Agent 调用；它不负责完整 Agent 执行循环。

### OpenClaw Memory + Wiki

OpenClaw 当前体系：Markdown 是可审计真源，SQLite 管 Session/索引/运行状态，`memory_search/get` 做检索，Wiki 用 Git Review 和人工知识生命周期。OpenViking可以增强结构化文档检索和自动 Session Memory，但会新增一套 AGFS、向量库、模型配置、异步任务、锁和备份体系；也会与 Memory Dreaming、Wiki 和 OMP Skills 重叠。

## Benchmark 判断

官方 2026-05-29 报告声称：

- LoCoMo：OpenClaw/Hermes/Claude Code 接入后均超过 80%；
- tau2-bench：Retail +6.87pp、Airline +11.87pp；
- HotpotQA：Top-20 91%，检索 0.23s；
- 多个任务 token 和延迟下降。

这些结果证明值得测试，但不能直接当成独立结论：

1. 报告由项目团队发布，基线配置、模型、Prompt 和集成质量可能不对称；
2. OpenClaw native memory 24.2% 与巨额 token 的基线非常差，需要核对是否代表当前版本和合理配置；
3. HotpotQA Top-20 使用 12,533 tokens/QA，和 Naive Top-K 的预算不同；
4. 单轮 RAG 中 OpenViking 66.87% 低于 LightRAG 76%，优势主要是延迟/索引成本折中；
5. 新的 VikingRAG 论文（arXiv:2609.11390）报告结构化文档上较低 token，但刚于 2026-09-10提交，尚不足以替代我们自己的数据集复现。

必须用我们的 Memory/Research Golden Set 做 A/B，而不是只看官方表格。

## 成熟度

- 约 36.8k stars、2.8k forks；创建于 2026-01；开发非常活跃。
- 稳定版 `v0.4.19`；Python >=3.10；AGPL-3.0。
- 仓库有约 763 个测试文件，主分支检查、API/CLI 集成、OpenClaw2OpenViking Memory、Security Scan、Docker 等多条 CI。
- 当前 open issue 约 701，部分是功能请求，但大量近期问题触及核心正确性和可用性。

公开高风险信号包括：

- #4963：working-memory summary 在 commit 时重处理整个累计 Session，产生 300k+ token VLM Prompt；
- #4948：`find` 被 rerank 主导，延迟约31秒；
- #4943/#4938/#4503：PathLock 续租、过期和竞争导致误删、永久阻塞或服务无响应；
- #4919：永久 AGFS 类型错误被误判为瞬态，无限重试并卡死整个语义摄取队列；
- #4884：OpenClaw `/reset` 未创建新 OV Session，旧摘要被重新注入；
- #4879：自托管记忆抽取实际对 Doubao/云路径依赖较强，本地模型体验有限；
- #4578：批量摄取可饿死交互式 Session commit 数天；
- #4482：文件级 embedding 已存但 search 不返回；
- #4377：量化 IP 索引分数溢出，Memory 结果挤掉 Resource。

这说明项目的抽象已成型，但并发、一致性、成本与集成边界仍未稳定。

## 安全、隐私与运维风险

- 自动摄取外部文档和 Skill 时，摘要/记忆抽取模型会读取不可信内容；必须把检索内容当数据，防 Prompt Injection，不能让文档指令自动获得工具权限。
- Memory 是 LLM 抽取结果，可能把错误、恶意内容或一次性信息固化；需要来源、置信度、人工修正和删除传播。
- 多租户必须验证 URI、账户/用户/Agent 隔离，不应只依赖目录命名。
- AGPL-3.0 对网络服务修改和分发有合规影响，商业嵌入需法务确认。
- 完整备份必须覆盖 AGFS 内容、向量索引或可重建策略、配置、加密密钥和任务状态；不能只备份向量库。
- 摄取会产生摘要、embedding、VLM和 rerank 成本；批量导入前必须估算队列吞吐和 token。
- Query planner/reranker/embedding 可依赖外部云服务，敏感文档可能离开本机。

## 对我们的建议

暂不安装到生产 OpenClaw，也不替换现有 Wiki/Memory。做隔离、只读 A/B Spike：

1. 固定 `v0.4.19`，独立目录/容器，本地向量存储；不导入个人聊天、Secrets、完整 Home 或 Skills。
2. 只导入 50–100 篇公开 Wiki Research/Readings 文档，保留原 Git/Markdown 为唯一真源。
3. 关闭自动长期 Memory 和 Skill extraction，先只测 Resource ingestion + `find/search/read`。
4. 准备 30–50 个真实查询和人工期望来源，比较 OpenViking 与 OpenClaw `memory_search`/Wiki grep：Recall@5、MRR、来源正确率、unsupported-claim rate、P50/P95、索引 token、查询 token。
5. 测试文档删除/更新、并发导入、进程中断、索引重建、队列卡死和备份恢复。
6. 放入 Prompt Injection 文档，验证检索结果不会变成执行指令。
7. Resource 检索明显胜出后，再用少量匿名化 Session 测 Memory；`/reset`、用户隔离、删除权和人工纠错必须通过。
8. 若采用，只让 OpenViking做派生检索层，Markdown/Wiki仍为权威源；不要同时让 OpenViking、Memory Dreaming 和 TeamAI 自动改写同一类长期记忆。

Verdict：**架构值得研究，层级检索可能对大规模 Research/Wiki 有收益；当前核心并发、成本和集成问题过多，只适合受控评测，不适合生产替换。**
