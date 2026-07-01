# HotGo 深入浅出 —— 全栈开发规范指南

> 全面解析 HotGo v2 框架的全栈开发规范。涵盖服务端（Go + GoFrame v2）、Web 管理后台（Vue 3 + Naive UI）、移动端 App（Flutter + GetX）、桌面端应用（Flutter Desktop）、微信小程序（Uni-app + wot-ui (wot-design-uni)）等五大平台的完整开发规范。

---

## 📚 文档导航

| 章节 | 内容 | 适合读者 |
|------|------|---------|
| [01 - 项目概览与技术栈](01-通用-项目概览与技术栈.md) | 整体技术架构、目录结构、路由模块、请求链路 | **所有人先读** |
| [02 - 后端开发规范](02-后端-开发规范.md) | 从零搭建、Go 分层架构全解、gf CLI 命令大全、config.yaml 详解 | Go 后端开发者 |
| [03 - 后端进阶主题](03-后端-进阶主题.md) | 插件系统、字典、错误/缓存/日志、数据权限、定时任务、消息队列、WebSocket、实战案例 | Go 后端开发者 |
| [04 - Web 后台管理系统](04-后台-开发规范.md) | Vue 3 + Naive UI 从零搭建、Vite 代理、环境变量、Pinia Store、插件/主模块权限控制、CLI 命令 | Web 前端开发者 |
| [05 - 移动端 App 开发规范](05-移动端-开发规范.md) | Flutter 从零创建、pubspec 每个依赖详解、Android 签名配置、完整 CLI 命令 | Flutter 移动端开发者 |
| [06 - 桌面端应用开发规范](06-桌面端-开发规范.md) | Flutter Desktop 从零创建、窗口管理、系统托盘、Forui 主题、自定义标题栏/侧边栏 | Flutter 桌面端开发者 |
| [07 - 小程序开发规范](07-小程序-开发规范.md) | Uni-app 从零创建、pages.json/manifest.json 详解、wot-design-uni 组件、条件编译 | 小程序开发者 |
| [08 - 常见错误与禁止清单](08-通用-常见错误与禁止清单.md) | 各端严格禁止行为 + 常见错误模式 + 正确做法对照 | **所有人必读** |
| [09 - 完整 CRUD 开发流程](09-通用-完整CRUD开发流程.md) | 从建表到前端页面渲染的端到端流程（16 个文件清单） | 新人入门实战 |
| [10 - HotGo 详解与扩展点](10-后端-HotGo详解与扩展点.md) | 框架设计哲学、30 个子系统全景剖析、40+ 扩展点速查 | **想深入理解框架者** |
| [11 - 数据库建表规范](11-后端-数据库建表规范.md) | MySQL + PostgreSQL 双版本 DDL、树状表、菜单权限体系、插件建表 | Go 后端开发者 |
| [12 - 中间件体系规范](12-后端-中间件体系规范.md) | 14 个中间件方法、4 种鉴权体系、DeliverUserContext 机制、Identity.App 规范 | Go 后端开发者 |

---

## 🎯 快速定位

### 我要做后端开发
→ [02-后端-开发规范](02-后端-开发规范.md) → [03-后端-进阶主题](03-后端-进阶主题.md) → [08-常见错误与禁止清单](08-通用-常见错误与禁止清单.md)
→ [11-数据库建表规范](11-后端-数据库建表规范.md) · [12-中间件体系规范](12-后端-中间件体系规范.md)

### 我要建数据库表 / 设计菜单权限
→ [11-数据库建表规范](11-后端-数据库建表规范.md)（MySQL + PostgreSQL 双版本 DDL · 树状表 · 菜单权限体系 · 插件建表）

### 我要理解鉴权机制 / 写自定义中间件
→ [12-中间件体系规范](12-后端-中间件体系规范.md)（14 个中间件方法 · 4 种鉴权对比 · Identity.App 规范 · 自定义中间件编写）

### 我要做 Web 后台开发
→ [04-后台-开发规范](04-后台-开发规范.md) → [08-常见错误与禁止清单](08-通用-常见错误与禁止清单.md)

### 我要做移动端 App 开发
→ [05-移动端-开发规范](05-移动端-开发规范.md) → [08-常见错误与禁止清单](08-通用-常见错误与禁止清单.md)

### 我要做桌面端开发
→ [06-桌面端-开发规范](06-桌面端-开发规范.md) → [08-常见错误与禁止清单](08-通用-常见错误与禁止清单.md)

### 我要做小程序开发
→ [07-小程序-开发规范](07-小程序-开发规范.md) → [08-常见错误与禁止清单](08-通用-常见错误与禁止清单.md)

### 我是新人，想完整走一遍 CRUD
→ [01-项目概览与技术栈](01-通用-项目概览与技术栈.md) → [09-完整CRUD开发流程](09-通用-完整CRUD开发流程.md) → [08-常见错误与禁止清单](08-通用-常见错误与禁止清单.md)

### 我想深入理解 HotGo 框架的全部扩展能力
→ [10-HotGo详解与扩展点](10-后端-HotGo详解与扩展点.md)（40+ 扩展点速查总表）

---

## 🔑 核心原则速记

1. **分层严格**：api → controller → service → logic → dao，禁止跨层调用
2. **代码生成优先**：entity/do/dao 由 `gf gen dao` 生成，禁止手改
3. **扩展点优先**：修改框架行为前先查 Hook/Handler/接口等预留扩展点，不动 `internal/library/` 核心代码
4. **鉴权分层**：Admin 端走 JWT + Casbin RBAC（查库绑定）；Api / Home 端只验 JWT 登录态，不做路由权限验证
5. **Identity.App 严填**：一个用户表对应一个 App 常量（admin / api / home），禁止跨表混用
6. **字典由后端统管**：前端禁止硬编码枚举值，统一通过字典接口获取
7. **UI 库唯一**：Web 后台只用 Naive UI，移动端 App 只用 TDesign Flutter
8. **错误要快返**：Go `err != nil` 后必须 `return`，Flutter catch 中必须重置 isLoading
9. **日志要统一**：Go 用 `g.Log()`，Flutter 用 `AppLogger`，禁止 `print` / `debugPrint`
10. **数据库双引擎**：建表需同时考虑 MySQL 和 PostgreSQL 语法差异，严格遵循标准字段规范

---

## 📖 相关资源

- [HotGo 官方仓库](https://github.com/bufanyun/hotgo)
- [GoFrame 官方文档](https://goframe.org)
- [Naive UI 官方文档](https://www.naiveui.com)
- [TDesign Flutter 文档](https://tdesign.tencent.com/flutter)
- [Flutter 官方文档](https://flutter.dev)
- [Uni-app 官方文档](https://uniapp.dcloud.net.cn)
- [wot-ui (wot-design-uni) 文档](https://wot-ui.cn/) · [插件市场](https://ext.dcloud.net.cn/plugin?id=27534)

---
