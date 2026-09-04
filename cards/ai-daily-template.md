# AI 日报模板

## 模板

```markdown
# 🤖 AI 日报 — YYYY-MM-DD

> 旺财每日整理，北京时间 09:00 发送

---

## 🔥 今日头条（1-2 条最重要的）
- **[标题]** — 一句话概括 + 为什么重要

---

## 📰 行业动态

### 模型 & 研究
- 新模型发布、benchmark 突破、论文亮点

### 产品 & 工具
- 新产品上线、重大更新、开发者工具

### 公司 & 融资
- 融资、收购、战略变动

### 开源 & 社区
- GitHub Trending、热门开源项目、社区事件

---

## 🔧 跟我们相关的
- 对旺财/OpenClaw/AI agent 生态直接有用的信息
- 可以立刻行动的（比如新 skill、新模型可以试）

---

## 💡 一句话观点
> 旺财对今天最值得关注的事的一句话思考

---

来源：Twitter/X · GitHub Trending · Hacker News · Product Hunt · AI 公司博客
```

## 数据源配置

### 核心源（每天必须检查）

| 数据源 | URL / 方式 | 说明 |
|--------|-----------|------|
| GitHub Trending | https://github.com/trending?since=daily | 记录仓库全名；同一仓库不因连续上榜而重复推送 |
| Anthropic News | https://www.anthropic.com/news | 官方新闻与研究发布 |
| Anthropic Engineering | https://www.anthropic.com/engineering | 工程实践与技术文章 |
| Claude Blog | https://claude.com/blog | Claude 产品、开发实践与使用指南 |

### 常规源

| 数据源 | URL / 方式 | 说明 |
|--------|-----------|------|
| Hacker News | https://news.ycombinator.com/best | 热门帖子 |
| Product Hunt | https://www.producthunt.com | 每日新品 |
| Twitter/X | 关注的 AI 账号列表 | 需要配置账号列表 |
| OpenAI Blog | https://openai.com/blog | |
| Google AI Blog | https://blog.google/technology/ai/ | |
| Hugging Face Blog | https://huggingface.co/blog | |
| ArXiv AI | https://arxiv.org/list/cs.AI/recent | 最新论文 |
| 36kr AI | https://36kr.com/information/AI/ | 中文 AI 资讯 |
| 量子位 | https://www.qbitai.com | 中文 AI 资讯 |

### 可选扩展源
| 数据源 | URL / 方式 | 说明 |
|--------|-----------|------|
| Reddit r/MachineLearning | https://reddit.com/r/MachineLearning | |
| Reddit r/LocalLLaMA | https://reddit.com/r/LocalLLaMA | |
| The Verge AI | https://www.theverge.com/ai-artificial-intelligence | |
| TechCrunch AI | https://techcrunch.com/category/artificial-intelligence/ | |
| OpenClaw Releases | https://github.com/openclaw/openclaw/releases | 跟我们直接相关 |
| ClawHub New Skills | https://clawhub.ai/skills?sort=recent | 新 skill |

## 增量采集与去重策略

### 持久化账本
- 使用 `wiki/ai-daily/seen-items.jsonl` 保存**已经向彦祖推送过**的条目，一行一个 JSON 对象。
- 字段至少包含：`key`、`title`、`url`、`source`、`firstSentAt`。
- 账本是长期历史，不按天清空；写入前先检查，禁止重复 `key`。
- 如果账本不存在，先扫描 `wiki/ai-daily/*.md` 历史日报，将能识别出的链接、GitHub 仓库和标题写入账本，再开始生成当天日报。

### 唯一键规则
按以下优先级生成 `key`：
1. GitHub 项目：`github:<owner>/<repo>`，统一转小写并移除末尾 `/`。
2. 有文章链接：规范化 URL；移除 `utm_*`、`ref` 等跟踪参数、fragment 和末尾 `/`，生成 `url:<normalized-url>`。
3. 没有稳定链接：`title:<source>:<normalized-title>`；标题转小写、压缩空白并移除纯标点差异。

### 每日流程
1. 读取 `seen-items.jsonl`，建立已推送 key 集合。
2. **每天必须检查所有核心源**，再按优先级检查常规源和扩展源。
3. 为候选内容记录标题、原始 URL、来源、发布日期和 key。
4. 排除账本中已有 key；同一事件被多个媒体转载时合并为一条，以官方原始链接为准。
5. 只保留有实际信息增量、且值得彦祖关注的内容；“仍在 Trending”不算更新，项目发布新版本或新增重要能力才算新事件。
6. 生成并发送日报后，**仅将实际发送的条目**追加到 `seen-items.jsonl`。写文件或发送失败时不要更新账本。
7. 如果没有重要增量，只发送简短的“今日无值得推送的重要 AI 更新”，不得用旧内容或低价值内容凑数。
8. 采集失败的源跳过并在内部记录，不用旧缓存冒充今日更新，也不阻塞其他源。

### 时间窗口
- 有明确发布时间的新闻、博客和论文，默认只考虑上次成功运行后发布的内容；首次建账时最多回看 72 小时。
- GitHub Trending 没有可靠发布时间，以“仓库是否从未推送过”为增量判断。
- 旧事件出现真正的新进展时可再次报道，但必须使用代表新进展的新文章 URL，并在正文中明确说明新增了什么。
