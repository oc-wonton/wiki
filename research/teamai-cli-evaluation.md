# TeamAI CLI 调研

项目：<https://github.com/Tencent/teamai-cli>

评估时间：2026-09-10；源码快照：`6ae0619d067b1699bb2c6e435abf3ffe11a21d71`；最新稳定版：`v0.23.1`。

## 定位

TeamAI CLI 不是新的 Coding Agent，而是多个 Agent 之上的“团队协作与知识控制平面”。它用一个 Git 仓库作为团队 Harness 和知识资产的权威源，把 Skills、Rules、Docs、Agent 定义、Hooks、MCP、非敏感 Env 和文化规则同步到 Claude Code、Codex、Cursor、CodeBuddy、OpenClaw 等工具；再从真实 Session 中提取高摩擦经验，建立检索、回灌和淘汰闭环。

它试图解决三类问题：

1. **Team Execution**：不同成员和不同 Agent 的规则、技能和工具配置漂移。
2. **Team Context**：Agent 每次从零理解项目，个人踩坑无法被其他人复用。
3. **Team Improvement**：Session 经验只留在聊天日志，知识只增不减，无法判断哪些资产有用。

## 核心机制

### 1. Git 作为团队控制面

团队仓库存放 `skills/`、`rules/`、`docs/`、`agents/`、`hooks/`、`mcp/`、`culture.md`、`learnings/` 和 `teamai.yaml`。

- `teamai pull`：从团队仓拉取，经角色、项目、Tag、个人排除规则过滤后，转换/同步到各 Agent 的本地目录。
- `teamai push`：把本地新增或修改资源推到分支并创建 PR/MR，经过团队 Review 后合并。
- SessionStart Hook：每次 Agent 会话启动自动 pull，使 Team Repo 合并后的 Harness 进入新会话。

这比普通 dotfiles 多了 Review、作用域、角色/项目过滤、格式适配、Hooks 注入和状态管理。

### 2. 多 Agent Adapter

内置 Agent registry 记录各工具的 Skills 路径，并针对 Rules、Agents、MCP、Hooks、CLAUDE.md/AGENTS.md 等格式做转换。Project scope 把资源放到项目目录；User scope 放到 Home；机器状态和索引放在 `~/.teamai` 或项目分区中，不污染业务仓。

OpenClaw 当前只支持 Skills、Rules、Docs、Env 和知识召回的一部分，不支持 TeamAI 表格中所有 Hooks/MCP/Agent/Usage 能力。OMP 没有明确的一等 Adapter；可能通过 Pi/Agent Skills 兼容路径使用 Skills，但不能假设其他配置自动适配。

### 3. Friction-based Learning

Stop Hook 不按 Session 长度判断是否值得沉淀，而是计算摩擦：用户中断、拒绝工具、纠正方向、工具错误重试、Recall 缺口等。超过阈值只提示使用分享学习 Skill，不自动把整段聊天上传。

`teamai contribute` 把人工整理后的 Markdown 写入 `learnings/`。独立团队仓模式可直接推送；单仓模式通过隔离 worktree 开 PR。贡献后重建本地搜索索引，使新经验立即可召回。

### 4. Recall

Recall 默认关闭。启用后，TeamAI 部署一个 recall subagent：先做相关性预检，再使用 BM25 搜索 Docs、Rules、Skills、Learnings，并结合投票/质量数据排序。代码知识使用 Wiki 图谱进行加权，结果返回来源文件路径。

### 5. Codebase Graph

`teamai codebase --extract` 把代码库抽取为 `teamwiki/`：组件、接口、配置与文件关系。TypeScript/JavaScript、Python、Go 走 WASM tree-sitter AST；Java/Rust 等走正则启发式；AST 失败会降级并记录 gap。Recall 用图的一到两跳邻接关系给 BM25 结果加分。

它不是完整语义编译器或实时 Code Index，图谱会陈旧，也会有启发式误边。

### 6. Knowledge Maintenance

记录 Recall 命中和引用投票，计算 confidence；维护命令可找出低置信或久未使用的 learning，先 dry-run，再 archive/remove；也能提示过期 Skills/Rules/Docs 更新。没有投票数据的文档不会被误删。

### 7. Session/Digest/Dashboard

Hook 记录隐私清洗后的工具序列、Prompt turn、干预和 Token，用于周报与 Dashboard。单仓模式把 sessions/members/votes/stats 放到独立 orphan reports branch，把知识资产留在 main，把本机状态留在仓外。

## 价值

TeamAI 的真正价值不是“集中放 Skills”，而是把团队 Agent 能力做成软件供应链：权威源、Review、版本、分发、按角色订阅、使用反馈、召回和淘汰。适合多人、多仓、多 Agent 工具且已有大量重复规则/经验的组织。

## 边界和风险

- 项目仍是 `0.x`，稳定版 v0.23.1，近期快速发版；Context/Improvement 明确标 beta。
- 自动 SessionStart pull 意味着团队仓的合并内容会进入每个人 Agent 的信任边界；Team Repo Review 权限和供应链安全非常关键。
- Hooks 可执行命令，MCP 可启动外部服务，Packages 可安装 npm/Claude 插件；pull 不自动安装 packages，但管理员声明和成员主动安装仍需审计。
- `env/` 明确不应用于 secrets；单仓模式会把 env 明文提交 main。
- Learnings 直接推送或通过 PR 的行为随模式不同，治理不一致；低质量经验仍可能污染 Recall。
- BM25 + graph boost 不是语义真理；需要 Recall Golden Set，验证召回率、误召回和来源新鲜度。
- 角色、项目、Tag、Source、User/Project scope 同时存在，配置复杂度高，可能出现同名资源覆盖和删除传播问题；Issue #458 等仍显示同步边界在修。
- 支持矩阵不等于每个平台能力完全一致，尤其 OpenClaw/OMP。

## 与现有体系的关系

现有 Wiki 已是知识源，OMP 是 Coding Agent，OpenClaw 负责调度和消息。TeamAI 可补的主要是：

- 多人/多 Agent 的 Skill/Rule 分发与 Review；
- SessionStart 自动同步；
- 摩擦触发的学习候选；
- Recall 使用和健康指标；
- 多项目/角色订阅。

重复或可能冲突的部分：Wiki、Memory Dreaming、Skill Workshop、OpenClaw Skills、OMP Skills、现有 Git 推送和定时任务。如果现在单人直接全量引入，会增加第二套状态、Hook、索引和知识生命周期，收益未必覆盖复杂度。

## 建议

暂不全量安装。先在非关键测试仓做只读/最小 Spike：

1. 建一个专用 private Team Repo，只放 2 个 Skills、2 条 Rules、3 篇 Learnings；禁用 MCP、Hooks、Env、Packages 和 Dashboard 上传。
2. 只接一个 Project scope 和一个 Agent adapter；确认 OMP 是否能可靠消费同步后的 Skill。
3. 准备 10 个真实查询作为 Recall Golden Set，记录 hit@k、错误召回、来源路径和上下文大小。
4. 模拟 Skill 更新、删除、冲突和回滚，验证 push → PR → merge → pull 的完整供应链。
5. 模拟高摩擦 Session，检查是否只是建议贡献而不会上传原始敏感 Transcript。
6. 通过后再决定是否用 TeamAI 替换现有 Wiki/Skill 分发的一部分；不要并行维护两套权威源。

Verdict：**设计方向与现有体系高度相关，团队化价值明显；但我们当前单人/少人环境下有较多重叠，适合隔离评测，不适合立即全量接管。**
