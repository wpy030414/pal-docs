# AGENTS.md

钉钉 OAuth2 适配层，将钉钉登录 API 包装为标准 OAuth2 端点。

## 概述

- 本项目是一个轻量级 HTTP 服务，作为钉钉身份认证与第三方系统之间的协议适配桥梁。
- 单文件实现（`index.ts`），无数据库、无状态，纯透传转发。

## 边界与范围

- 范围内：钉钉 → 标准 OAuth2 的三个端点适配（authorize、token、userinfo）
- 非目标：不做用户管理、不做 token 持久化、不做多租户、不做钉钉企业内部 API 调用

## Agent 操作指南

- 整个应用逻辑在 `src/index.ts` 一个文件内，约 95 行，结构扁平。
- `CONFIG` 从 `process.env` 读取，修改配置项时同步更新 `.env.example`。
- 路由风格为 `/oauth/<name>`，遵循 OAuth2 惯例。
- HTTP 请求使用原生 `fetch`，不要引入 axios 或其他 HTTP 客户端。
- 框架为 Hono，路由注册用 `app.get()` / `app.post()`，响应通过 `c.json()` / `c.text()` / `c.redirect()` 返回。

## 目录速查

- `src/index.ts` — 全部业务逻辑（路由、配置、服务启动）
- `.env.example` — 环境变量模板
- `tsconfig.json` — TypeScript 编译配置
- `package.json` — 依赖与脚本
- `docs/` — 产品需求、架构、决策记录
