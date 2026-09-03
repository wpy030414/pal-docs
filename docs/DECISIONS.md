# DECISIONS

## ADR-001：使用 Hono 替代 Express

- 日期：2026-09-03
- 状态：已采纳
- 背景：原实现使用 Express，项目技术栈规范要求 Web Gateway 层采用 TypeScript + Hono。
- 考虑过的方案：Express（原方案）、Fastify、Hono
- 决策：使用 Hono + @hono/node-server
- 为什么选这个：与项目全局技术栈一致；Hono 轻量、TypeScript 原生支持好、API 简洁；在 Node.js 上通过 @hono/node-server 运行，未来可零改动迁移到 Cloudflare Workers / Bun / Deno。
- 为什么不选其他：Express 无 TypeScript 原生支持、中间件模型较重；Fastify 功能丰富但对本项目的简单透传场景过于重量。
- 后果：依赖从 express + axios 简化为 hono + @hono/node-server，HTTP 请求改用原生 fetch。
- 何时重新审视：如果需要添加中间件（日志、限流、CORS 等）时评估 Hono 内置能力是否足够。

## ADR-002：使用原生 fetch 替代 axios

- 日期：2026-09-03
- 状态：已采纳
- 背景：原实现使用 axios 发起 HTTP 请求。Node.js 18+ 已内置全局 fetch API。
- 考虑过的方案：axios（原方案）、node-fetch、原生 fetch
- 决策：使用原生 fetch
- 为什么选这个：零依赖、无额外包安装；Node 24 LTS 的 fetch 实现已稳定成熟；项目仅需简单的 POST/GET 请求，不需要 axios 的拦截器等高级功能。
- 为什么不选其他：axios 和 node-fetch 都引入了额外依赖，对本项目的简单需求来说不必要。
- 后果：减少一个运行时依赖，package 更精简。
- 何时重新审视：如果需要请求重试、超时控制、拦截器等能力时重新评估。

## ADR-003：expires_in 硬编码 7200s

- 日期：2026-09-03
- 状态：已采纳（待改进）
- 背景：钉钉 token 接口返回 `expireIn`（注意大小写），原实现硬编码返回 `expires_in: 7200`。
- 考虑过的方案：硬编码 7200s（当前）、透传钉钉返回的 `expireIn`
- 决策：暂时保持硬编码，后续改为透传
- 为什么选这个：与原实现行为一致，避免引入未测试的行为变更。
- 为什么不选其他：透传方案更正确，但需要额外验证钉钉返回的字段名和实际值，作为后续改进。
- 后果：token 过期时间可能与钉钉实际值不完全一致。
- 何时重新审视：下次迭代时改为透传钉钉实际的 `expireIn` 值。
