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

| 数据源 | URL / 方式 | 说明 |
|--------|-----------|------|
| Hacker News | https://news.ycombinator.com/best | 热门帖子 |
| GitHub Trending | https://github.com/trending?since=daily | 每日趋势项目 |
| Product Hunt | https://www.producthunt.com | 每日新品 |
| Twitter/X | 关注的 AI 账号列表 | 需要配置账号列表 |
| OpenAI Blog | https://openai.com/blog | |
| Anthropic Blog | https://www.anthropic.com/news | |
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

## 采集策略
- 数据源列表可在此文件中增减
- 每个源标注优先级（核心/扩展）
- cron 触发后按优先级依次采集
- 采集失败跳过，不阻塞整体流程
