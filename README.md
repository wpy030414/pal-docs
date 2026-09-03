# oauth2-dingtalk

钉钉 OAuth2 适配器 —— 将钉钉登录接口转为标准 OAuth2 端点，供第三方系统以标准 OAuth2 协议对接钉钉身份认证。

## 这是什么？

- 定位：钉钉身份认证的标准 OAuth2 适配层
- 解决的核心问题：第三方系统（Nextcloud Social Login、Keycloak 等）无法直接对接钉钉私有的登录 API，本项目将其包装为标准 OAuth2 端点

## 为什么存在？

- 钉钉开放平台的授权登录接口（`login.dingtalk.com/oauth2/auth` + `api.dingtalk.com/v1.0/oauth2/userAccessToken`）虽然也是 OAuth2 风格，但与标准 OAuth2 Provider 的 `/authorize`、`/token`、`/userinfo` 端点存在差异（如 token 端点用 POST body 而非 `Authorization` header、userinfo 端点用自定义 header `x-acs-dingtalk-access-token` 而非标准 Bearer Token 等）。本项目作为中间层屏蔽这些差异，让任何支持标准 OAuth2 的系统都能无感对接钉钉。

## 如何安装和运行？

- 前置要求：Node.js >= 24 (LTS)、pnpm >= 11
- 安装步骤：

```bash
pnpm install
cp .env.example .env  # 然后编辑 .env 填入钉钉应用凭据
```

- 运行 / 启动：

```bash
pnpm dev    # 开发模式（热重载）
pnpm build  # 编译
pnpm start  # 启动编译产物
```

### 环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `CORP_ID` | 企业 CorpID（可选，用于锁定企业） | — |
| `CLIENT_ID` | 钉钉开放平台 AppKey | 必填 |
| `CLIENT_SECRET` | 钉钉开放平台 AppSecret | 必填 |
| `REDIRECT_URI` | OAuth2 回调地址 | 必填 |
| `LOCAL_LISTEN_PORT` | 监听端口 | `28081` |

## 当前状态

- 阶段：稳定（功能完整，可投入使用）
- 已知限制：`expires_in` 硬编码为 7200s，未使用钉钉实际返回值；`CORP_ID` 已读取但暂未使用

## 核心技术

- Hono — 轻量高性能 Web 框架
- @hono/node-server — Hono 的 Node.js 适配器
- TypeScript — 类型安全
- 原生 `fetch` — 零外部 HTTP 依赖

详细架构与决策记录见 `docs/`。
