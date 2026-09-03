# ARCHITECTURE — oauth-dingtalk

## 系统概述

单进程、无状态的 HTTP 适配服务，部署在钉钉 API 与第三方系统之间。

```
┌──────────────┐         ┌──────────────────┐         ┌──────────────────┐
│  第三方系统   │  OAuth2 │  oauth-dingtalk  │  钉钉API │  钉钉开放平台     │
│ (Nextcloud/  │◄──────►│  (Hono 服务)      │◄──────►│  login.dingtalk  │
│  Keycloak等) │         │                  │         │  api.dingtalk    │
└──────────────┘         └──────────────────┘         └──────────────────┘
```

## 核心模块

| 模块 | 职责 |
|------|------|
| 配置 (`CONFIG`) | 从环境变量读取凭据、回调地址、端口 |
| `/oauth/authorize` | 构造钉钉授权 URL 并 302 重定向 |
| `/oauth/token` | 接收 `code`，调用钉钉 `userAccessToken` 接口，返回标准 token 响应 |
| `/oauth/userinfo` | 从 `Authorization` header 提取 Bearer Token，调用钉钉 `contact/users/me`，返回标准格式的 userinfo |

## 模块关系

- 三个路由彼此独立、无状态共享，仅共享 `CONFIG`。
- 数据流为单向透传：第三方系统 → 本服务 → 钉钉 API → 本服务 → 第三方系统。

## 数据流

1. 用户在第三方系统点击「钉钉登录」
2. 第三方系统将用户重定向到本服务的 `/oauth/authorize`
3. 本服务 302 到钉钉授权页，用户授权后钉钉回调到 `REDIRECT_URI`（即第三方系统）
4. 第三方系统用 `code` 调本服务的 `/oauth/token`，本服务转发给钉钉 API 并返回标准格式
5. 第三方系统用 `access_token` 调本服务的 `/oauth/userinfo`，本服务转发给钉钉 API 并返回标准格式

## 外部系统

- **钉钉授权页** (`login.dingtalk.com/oauth2/auth`) — 用户授权入口
- **钉钉 Token API** (`api.dingtalk.com/v1.0/oauth2/userAccessToken`) — 授权码换 token
- **钉钉用户 API** (`api.dingtalk.com/v1.0/contact/users/me`) — 获取用户信息

## 重要技术边界

- 无持久化：不存储 token、session 或任何状态
- 无鉴权：本服务本身不验证调用方身份，信任网络层或第三方系统的 client_secret 校验
- 单文件：所有逻辑在 `src/index.ts` 中，不拆分模块
