# AList API 架构说明

> 基于 `/home/runner/work/alist/alist/server/router.go` 的实际路由定义整理。

## 1. 总体架构

- 框架：Gin
- 根分组：`g := e.Group(conf.URL.Path)`
- 主要模块：
  - 根级公共接口（健康检查、下载、分享页、归档下载）
  - `/api`（业务 API）
  - `/dav`（WebDAV）
  - `/s3`（S3 兼容）
  - `/debug`（仅 debug/dev）

## 2. 中间件分层

- 全局：
  - `SessionRefresh`
  - 条件启用 `ForceHttps`
  - 条件启用 `MaxAllowed`
  - `StoragesLoaded`（在主要业务路由前）
- `/api`：
  - `Auth`：需要登录
  - `Authn`：WebAuthn 管理相关
  - `AuthNotGuest`：禁止访客执行敏感写操作
  - `AuthAdmin`：管理员接口

## 3. 路由分组结构

```text
/{base}
├── /ping
├── /d/*path, /p/*path, /s/:share_id, /sd/:share_id, /sp/:share_id
├── /ad/*path, /ap/*path, /ae/*path
├── /dav/**
├── /s3/**
└── /api
    ├── /auth/*
    ├── /me/*
    ├── /authn/*
    ├── /public/*
    ├── /fs/*
    ├── /share/*
    ├── /task/*
    ├── /label/*
    ├── /label_file_binding/*
    └── /admin
        ├── /meta/*
        ├── /user/*
        ├── /role/*
        ├── /storage/*
        ├── /driver/*
        ├── /setting/*
        ├── /task/* (兼容旧脚本)
        ├── /message/*
        ├── /index/*
        ├── /label/*
        ├── /label_file_binding/*
        └── /session/*
```

## 4. 认证与权限模型

- 无需登录：
  - `/api/auth/login`、`/api/auth/register`
  - `/api/public/*`
  - 分享页与下载相关公开入口
- 需登录（`Auth`）：
  - `/api/me/*`、`/api/fs/*`、`/api/task/*` 等
- 非访客（`AuthNotGuest`）：
  - `/api/share/*`
  - `/api/task/*`
- 管理员（`AuthAdmin`）：
  - `/api/admin/*`
  - `/api/fs/link`

## 5. 资源域划分

- 身份域：`/auth`、`/me`、`/authn`
- 文件域：`/fs`、下载代理 `/d` `/p`
- 分享域：`/share`、公开分享 `/public/share/*`、分享页 `/s/*`
- 任务域：`/task`（上传、复制、离线下载、解压等）
- 管理域：`/admin/*`（用户、角色、存储、设置、索引等）
- 协议域：`/dav`、`/s3`

## 6. 兼容性说明

- 管理端保留 `/api/admin/task/*`，内部复用与 `/api/task/*` 相同任务路由注册逻辑，确保旧自动化脚本可继续使用。

## 7. 维护建议

- 如调整 API，请优先修改 `/home/runner/work/alist/alist/server/router.go`，并同步更新本文件。
- 建议后续补充：
  - 每个接口的请求/响应模型索引
  - 错误码与权限矩阵
  - 与 Apifox 文档的映射表
