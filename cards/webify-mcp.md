# Webify MCP — 浏览器扩展作为 MCP 适配器

## 是什么
通用框架，把 Chrome 扩展的网络拦截数据转化为 MCP 工具，让 AI Agent 能访问没有公开 API 的 Web 平台。

## 核心架构
1. Chrome 扩展拦截 `webRequest`，捕获平台 API 响应
2. 适配包（JSON Schema）定义 URL pattern → 字段映射
3. MCP Server 通过 WebSocket 接收数据，暴露为标准工具

## 为什么不用 Puppeteer
- 信号质量：原始 JSON vs 解析 DOM
- 稳定性：后端 API 比前端 DOM 稳定一个数量级
- 身份：复用用户登录态，无需模拟登录

## 技术栈
- WXT 框架（Vite-based 扩展开发）
- TypeScript
- WebSocket 通信

## 接入新平台
写一份适配包 JSON 配置即可，无需编写 MCP server 代码。

## Repo
https://github.com/oc-wonton/webify-mcp
