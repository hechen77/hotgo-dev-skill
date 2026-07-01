# HotGo Dev Skill SKILL.md 重写 — 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将 SKILL.md 从当前结构重写为按能力域组织的三 Part 结构（知识获取 → 行为规则 → 执行流程），强化各端速查、回答规范，新增规范检查/修复功能。

**Architecture:** 单文件 SKILL.md，三段式组织。Part 1 告诉 AI 去哪找知识，Part 2 约束 AI 的行为方式，Part 3 定义 AI 的标准执行流程。从现有内容出发，重组 + 增强 + 新增，不拆出新文件。

**Tech Stack:** Markdown，无代码逻辑，无外部依赖。

**Spec:** `docs/superpowers/specs/2026-07-01-hotgo-dev-skill-improvements-design.md`

---

## 文件结构

| 文件 | 操作 | 职责 |
|------|------|------|
| `SKILL.md` | 完全重写 | 三 Part 结构的完整 skill 定义文件 |

SKILL.md 内部结构：

```
SKILL.md
├── frontmatter (不变)
├── # HotGo 开发规范 Skill (标题 + 简介)
├── ## 与 goframe-v2 skill 协作 (不变)
├── # Part 1：知识获取
│   ├── ## 文档索引
│   └── ## 各端速查
├── # Part 2：行为规则
│   ├── ## 核心原则（铁律）
│   ├── ## 回答四步链路
│   ├── ## 禁止事项矩阵
│   └── ## 分端回答策略
├── # Part 3：执行流程
│   ├── ## 新建 CRUD 模块
│   └── ## 规范检查与修复
```

---

### Task 1: 写入 Part 1 — 知识获取

**Files:**
- Modify: `E:\Projects\skills\hotgo-dev\SKILL.md`（写入前半段：frontmatter → Part 1 完）

- [ ] **Step 1: 写入 frontmatter + 标题 + goframe-v2 协作 + Part 1**

写入以下完整内容到 SKILL.md：

```markdown
---
name: hotgo-dev
description: HotGo v2 全栈开发规范。TRIGGER when user asks about HotGo project structure, layered architecture (api/controller/service/logic/dao), CRUD workflow, Web admin (Vue3+NaiveUI), Flutter mobile/desktop, Uni-app mini-program, HotGo plugins, common mistakes, or any GoFrame-based HotGo development. Also trigger when user mentions "hotgo" or needs guidance on building features in a HotGo project.
---

# HotGo 开发规范 Skill

基于《HotGo 深入浅出》全栈开发规范文档。覆盖 Go 服务端、Vue3 Web 后台、Flutter 移动/桌面端、Uni-app 小程序五大平台。

## 与 goframe-v2 skill 协作

本 skill 专注于 HotGo 项目特有的**架构规范**和**文件组织**。当用户编写 Go 代码时，`goframe-v2` skill 负责底层 GoFrame 编码规范（DO 对象操作、gerror 错误处理、变量声明风格等）。

**两 skill 互不冲突**：
- `hotgo-dev` → 文件放哪里、走什么流程、用哪个组件
- `goframe-v2` → Go 代码怎么写、DO 对象怎么用、错误怎么处理

在后端开发场景中，两个 skill 可能同时触发。Claude 会综合两者的规范给出答案。

---

# Part 1：知识获取

> **接到任何 HotGo 相关问题，第一步在此定位正确的文档组合。**

## 文档索引

所有文档位于 `references/` 目录。**回答前必须实际读取对应文件，不可凭记忆回答。**

| 文档 | 内容 | 何时读取 |
|------|------|---------|
| `references/README.md` | 全文档导航、快速定位、核心原则速记 | 首次使用、需要定位文档 |
| `references/01-通用-项目概览与技术栈.md` | 技术架构、四大路由模块、目录结构、请求链路 | 用户首次提问、问项目结构 |
| `references/02-后端-开发规范.md` | 分层架构全解、gf CLI、config.yaml、标准 CRUD 接口清单 | 写 Go 代码、建模块、配 config |
| `references/03-后端-进阶主题.md` | 插件系统、字典/缓存/日志、数据权限、定时任务、消息队列、WebSocket、实战案例 | 问插件、字典、队列、WebSocket |
| `references/04-后台-开发规范.md` | Vite 代理、Pinia Store、列表/编辑页标准写法、字典系统、权限控制 | 写 Vue3 页面、配代理、加菜单 |
| `references/05-移动端-开发规范.md` | pubspec 每个依赖详解、Android 签名、文件清单、Controller 规范、WS 体系 | 写 Flutter App 代码 |
| `references/06-桌面端-开发规范.md` | 窗口管理、系统托盘、自定义标题栏/侧边栏、Forui 主题、与移动端 14 项差异 | 写 Flutter 桌面端代码 |
| `references/07-小程序-开发规范.md` | pages.json、manifest.json、wot-ui (wot-design-uni) 组件、条件编译、rpx 单位 | 写 Uni-app 小程序代码 |
| `references/08-通用-常见错误与禁止清单.md` | 五端禁止行为 + 常见错误模式 + 正确做法对照 | 用户报错、代码 review、自查 |
| `references/09-通用-完整CRUD开发流程.md` | 建表到前端渲染的 16 个文件完整流程 | 新建 CRUD 模块、问"要改哪些文件" |
| `references/10-后端-HotGo详解与扩展点.md` | 30 个子系统、40+ 扩展点速查总表 | 深入理解框架、做扩展开发 |
| `references/11-后端-数据库建表规范.md` | MySQL + PostgreSQL 双版本 DDL、标准字段、树状表、菜单权限体系、插件建表 | 建表、设计菜单权限 |
| `references/12-后端-中间件体系规范.md` | 14 个中间件方法、4 种鉴权体系、DeliverUserContext 机制、Identity.App 规范 | 理解鉴权、写自定义中间件 |

> **提示**：`03-后端-进阶主题` 中的数据库建表和中间件内容已独立为 `11` 和 `12` 号文档，内容更详尽。建表问题优先读 `11`，鉴权/中间件问题优先读 `12`。

## 各端速查

根据用户问题的端侧，定位**全部**相关文档和对应场景。

| 端 | 技术栈 | 全部相关文档 | 核心规则 | 按场景定位 |
|---|--------|------------|---------|----------|
| **后端** | Go + GoFrame v2 | 02 · 03 · 10 · 11 · 12 | 分层严格、gf gen dao、禁止直调 dao | 新建CRUD→02§9 · 建表→11 · 鉴权→12 · 插件→03§1 · 扩展框架→10 · 字典→03§2 · 缓存→03§4 · 队列→03§9 · 避坑→08§1 |
| **Web 后台** | Vue 3 + Naive UI | 04 | BasicTable/BasicModal/BasicForm、字典用 computed | 列表页→04§8 · 编辑页→04§9 · 字典→04§10 · API封装→04§6 · 权限→04§11 · 路由→04§12 · 避坑→08§2 |
| **移动端 App** | Flutter + GetX + TDesign | 05 | Controller 分离、try-catch 全覆盖、AppLogger | 创建→05§1 · pubspec→05§2 · HTTP→05§8 · Controller→05§13 · Service→05§10 · WS→05§15 · 路由→05§11 · 避坑→08§3 |
| **桌面端** | Flutter Desktop + Forui + TDesign | 06 | 窗口管理、托盘、与移动端 14 项差异 | 创建→06§1 · 窗口→06§7 · 托盘→06§8 · 标题栏→06§10 · 侧边栏→06§11 · Forui→06§13 · 差异表→06§17 · 避坑→08§4 |
| **小程序** | Uni-app + wot-ui (wot-design-uni) | 07 | 条件编译、rpx 单位、pages.json 注册 | 创建→07§1 · manifest→07§2 · pages→07§3 · API→07§9 · 页面→07§10 · 生命周期→07§11 · 避坑→08§5 |

> 以上未列出的场景，以及跨端通用问题（如完整 CRUD 流程），读 `08-通用-常见错误与禁止清单` 和 `09-通用-完整CRUD开发流程`。
```

- [ ] **Step 2: 验证 Part 1 完整性**

对照 spec 检查：
- [ ] 文档索引 13 行全部存在（README + 01~12）
- [ ] 各端速查 5 行（后端/Web/移动端/桌面端/小程序）
- [ ] 每行含 5 列（端/技术栈/全部相关文档/核心规则/按场景定位）
- [ ] 后端"全部相关文档"列出 5 个（02/03/10/11/12）
- [ ] 每端"按场景定位"含具体章节引用（如 02§9、11）

---

### Task 2: 写入 Part 2 — 行为规则

**Files:**
- Modify: `E:\Projects\skills\hotgo-dev\SKILL.md`（在 Part 1 之后追加 Part 2）

- [ ] **Step 1: 追加 Part 2 完整内容**

在 SKILL.md 末尾追加以下内容：

```markdown
---

# Part 2：行为规则

> **AI 在所有回答中的强制行为约束。不可跳过任何一步。**

## 核心原则（铁律）

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

## 回答四步链路

每次回答涉及 HotGo 代码的问题，**必须按以下四步执行，不可跳过任何一步**：

| 步骤 | 动作 | 说明 |
|-----|------|------|
| **① 读文档** | 根据[各端速查](#各端速查)定位并**实际读取**对应 references 文档 | 不可凭记忆回答。该读几个读几个——后端建表需同时读 02+11，鉴权问题需读 02+12 |
| **② 自检** | 对照[禁止事项矩阵](#禁止事项矩阵)逐条检查方案/代码 | 问自己：我的方案/代码有没有踩中禁止清单？踩中则**必须先警告用户**，再给出正确方案 |
| **③ 输出** | 按[分端回答策略](#分端回答策略)组织输出 | 完整代码模板（可直接复制粘贴）+ 解释为什么 + 标注文档出处（文档名 § 章节） |
| **④ 审查** | 输出代码后**再次**对照禁止矩阵逐条检查 | 发现不合规立即告知用户修正，修正后输出最终版本。检查项包括但不限于：代码是否有禁止 API、文件位置是否正确、必要写法是否遗漏 |

> **违反此链路（如未读文档就凭记忆回答、未自检就输出代码、输出后未审查）视为 skill 执行失败。**

## 禁止事项矩阵

回答代码问题或输出代码时，**必须对照下表逐条自检**。触发任何禁止行为时，立即警告用户并给出正确做法。

### 后端 (Go + GoFrame)

| 禁止行为 | 正确做法 |
|---------|---------|
| controller 直接调用 dao | controller → service → logic → dao |
| 硬编码字段名字符串 `"create_time"` | 使用 `dao.Table.Columns().CreatedAt` |
| 手改 `entity/`、`do/`、`dao/internal/` | `gf gen dao` 自动生成，禁止手改 |
| logic 重复做 `Filter()` 已完成的校验 | 校验只在 `sysin.Filter(ctx)` 中完成 |
| 在 `internal/service/sys.go` 之外新建 service 文件 | 统一追加到 `sys.go` |
| 插件 import 另一个插件的包 | 插件间通过主模块 service 通信 |
| 插件路由硬编码路径前缀 | 用 `addons.RouterPrefix()` 自动生成 |
| controller 层手动包装响应 | 交给 `middleware.ResponseHandler` |
| `fmt.Println` 或 `print` 替代日志 | 用 `g.Log().Infof(ctx, ...)` |
| controller 层打印 framework 已记录的 error | 直接 `return` 透传 |
| 修改 `internal/library/` 下的基础库 | 优先查扩展点（Hook/Handler/接口），在 `utility/` 添加工具或提 issue |
| `err != nil` 后不 `return` 继续执行 | 必须立即 `return` |
| 用 `err1/err2` 绕过命名返回值 `err` | 始终使用命名返回值 `err` |

### Web 前端 (Vue 3 + Naive UI)

| 禁止行为 | 正确做法 |
|---------|---------|
| 直接使用 axios | 用 `http.request` / `http.get` / `http.post` |
| 硬编码字典文本 `row.status === 1 ? '启用' : '禁用'` | 用 `dict.getLabel('XxxStatusOptions', row.status)` |
| 引入 Naive UI 以外的 UI 框架（Element Plus、Ant Design Vue 等） | 只用 Naive UI 原生组件 |
| `dict.getOptions()` 不包裹 `computed` | `computed(() => dict.getOptions(...))` |
| 源码中硬编码路由路径 | 路由和菜单由后端数据驱动 |
| 列表页忘记调用 `loadOptions()` | 在 setup 中必须调用 `loadOptions()` |
| 编辑表单回填数据用浅拷贝 | 用 `cloneDeep` 深拷贝（lodash-es） |
| 内联 `style="color: red"` | 用 Naive UI props 或 class |

### 移动端 (Flutter + GetX)

| 禁止行为 | 正确做法 |
|---------|---------|
| `print()` / `debugPrint()` / `Logger()` 裸实例 | `AppLogger.d/i/e()` |
| API 调用不包裹 try-catch | 所有 API 调用必须有 try-catch |
| catch 块中忘记重置 `isLoading` 和 `update()` | catch 中也必须 `isLoading = false; update()` |
| Controller 无 `onClose()` | 所有 GetxController 必须有 `onClose()`（至少调用 super） |
| Service 的 `init()` 无 try-catch | 异常不阻塞启动，用默认值降级 |
| 散落的硬编码 URL / 常量 / 密钥 | 统一放 `AppConfig` / `AppStorage` |
| `TextEditingController` 不在 `onClose()` 中 dispose | 在 `onClose()` 中 `dispose()` |

### 桌面端 (Flutter Desktop)

| 禁止行为 | 正确做法 |
|---------|---------|
| 关闭按钮直接退出程序 | 隐藏到托盘 `windowManager.hide()` |
| 登录页和主界面相同窗口大小 | 登录锁 320×480，主界面 1200×800 |
| `onWindowClose` 中调用 `windowManager.close()` | `windowManager.hide()` |
| `addListener` 后不在 `dispose` 中 `removeListener` | 必须配对 add/remove |

### 小程序 (Uni-app + Vue 3)

| 禁止行为 | 正确做法 |
|---------|---------|
| 直接调用微信原生 API（`wx.xxx`） | 通过 `uni.` API 统一调用 |
| 新增页面不在 `pages.json` 中注册 | 每次新增页面必须注册路由 |
| rpx 和 px 混用 | 统一用 rpx（750rpx = 屏幕宽度） |
| 忘记配置 `manifest.json` 的 `mp-weixin.appid` | 正确填写微信小程序 AppID |

## 分端回答策略

根据用户问题的端侧调整回答重点和必须覆盖的内容：

| 端 | 回答重点 | 必须覆盖 |
|---|---------|---------|
| **后端** | 分层调用链路完整、DAO 字段引用正确、Service 注册完整 | 文件路径 → 分层职责 → 完整代码模板 → 禁止清单逐条对照警告 |
| **Web 前端** | API 封装 → model.ts → index.vue → edit.vue 四人组协同 | 四个文件全部给出 → 字典用法（loadOptions+getLabel+computed） → 组件选型（BasicTable/BasicModal/BasicForm） |
| **移动端** | Controller + View 分离、try-catch 全覆盖、Service 初始化顺序 | Controller 模板（含 onInit/onClose/loadList） → API 调用 → WS 连接/断开 |
| **桌面端** | 窗口管理配对、托盘流程、Forui 组件使用 | 窗口/托盘 Service 完整代码 → 自定义标题栏/侧边栏 → main.dart 启动流程 |
| **小程序** | pages.json 路由注册、条件编译片段、wot-ui (wot-design-uni) 组件 props | 完整页面模板（template + script setup + style scoped） → 配置片段 → rpx 单位 |
```

- [ ] **Step 2: 验证 Part 2 完整性**

对照 spec 检查：
- [ ] 核心原则 10 条全部存在，编号 1-10
- [ ] 回答四步链路表格含 4 步（①-④）
- [ ] 禁止事项矩阵含 5 个子表（后端/Web/移动端/桌面端/小程序）
- [ ] 后端禁止清单 13 条
- [ ] Web 前端禁止清单 8 条
- [ ] 移动端禁止清单 7 条
- [ ] 桌面端禁止清单 4 条
- [ ] 小程序禁止清单 4 条
- [ ] 分端回答策略表格 5 行，每行含"回答重点"和"必须覆盖"两列

---

### Task 3: 写入 Part 3 — 执行流程

**Files:**
- Modify: `E:\Projects\skills\hotgo-dev\SKILL.md`（在 Part 2 之后追加 Part 3）

- [ ] **Step 1: 追加 Part 3 完整内容**

在 SKILL.md 末尾追加以下内容：

```markdown
---

# Part 3：执行流程

> **AI 执行新建模块和规范检查两个标准操作的标准流程。**

## 新建 CRUD 模块

当用户说"我要新建一个 XX 管理模块"时，严格按以下步骤操作。

### 第一步：先列出全量文件清单

从下方表格告知用户需要创建/修改的全部文件（后端 12 + Web 前端 4 = 16 个），让用户确认后再逐个编写。

| #   | 文件                                                 | 操作                                   | 端   |
| --- | -------------------------------------------------- | ------------------------------------ | --- |
| 1   | `server/manifest/sql/<module>.sql`                 | 建表 SQL（遵循[数据库建表规范](references/11-后端-数据库建表规范.md)） | 后端  |
| 2   | `server/internal/model/entity/<module>.go`         | `gf gen dao` 自动生成                    | 后端  |
| 3   | `server/internal/model/do/<module>.go`             | `gf gen dao` 自动生成                    | 后端  |
| 4   | `server/internal/dao/internal/<module>.go`         | `gf gen dao` 自动生成                    | 后端  |
| 5   | `server/internal/dao/<module>.go`                  | `gf gen dao` 自动生成                    | 后端  |
| 6   | `server/internal/consts/<module>.go`               | 手写：常量 + 选项变量 + `init()` 字典注册         | 后端  |
| 7   | `server/internal/model/input/sysin/<module>.go`    | 手写：入参/出参/过滤模型                        | 后端  |
| 8   | `server/internal/service/sys.go`                   | 追加：接口声明 + 注册函数                       | 后端  |
| 9   | `server/internal/logic/sys/<module>.go`            | 手写：业务实现                              | 后端  |
| 10  | `server/internal/controller/admin/sys/<module>.go` | 手写：薄控制器                              | 后端  |
| 11  | `server/api/admin/<module>/<module>.go`            | 手写：API 契约                            | 后端  |
| 12  | `server/internal/router/genrouter/<module>.go`     | 手写：路由注册                              | 后端  |
| 13  | `web/src/api/<module>/index.ts`                    | 手写：8 个标准 API 函数                      | Web |
| 14  | `web/src/views/<module>/model.ts`                  | 手写：State + schemas + columns + rules | Web |
| 15  | `web/src/views/<module>/index.vue`                 | 手写：BasicTable 列表页                    | Web |
| 16  | `web/src/views/<module>/edit.vue`                  | 手写：BasicModal + BasicForm 编辑页        | Web |

### 第二步：逐文件编写

用户确认清单后，按顺序逐个文件编写。每写完一个文件，标注完成状态。

写入顺序：
1. 先数据库建表 → `gf gen dao` 生成 entity/do/dao
2. 再常量 + Model → Service → Logic → Controller → API → Router
3. 最后 Web 前端：API 封装 → model.ts → index.vue → edit.vue

### 第三步：后端菜单配置提醒

全部文件写完后，提醒用户还需在后端后台手动操作：
- 权限管理 → 菜单权限 → 添加菜单 → 绑定 API 权限
- 权限管理 → 角色权限 → 为角色分配新菜单

---

## 规范检查与修复

当用户要求"检查代码规范"或"按 HotGo 规范修复项目"时，进入此流程。

### 触发方式

| 触发方式 | 条件 | 行为 |
|---------|------|------|
| **主动触发** | 用户说"检查后端规范" / "按规范修复这个文件" / "scan and fix" / "lint 检查" | 立即进入[标准检查流程](#标准检查流程) |
| **自动嗅探** | 用户贴出代码让 review、描述操作方案让你确认 | 在[回答四步链路](#回答四步链路)的④审查步骤中执行，发现违规主动告知用户并建议进行全量检查 |

### 检查范围

用户可指定以下任意范围，AI 按需执行：

| 范围 | 用户命令示例 | 覆盖内容 |
|------|------------|---------|
| **全项目** | "检查整个项目的规范" | 五端全部目录扫描 |
| **按端** | "检查后端代码规范" / "检查 Web 前端规范" / "检查移动端规范" | 只扫描该端目录（`server/`、`web/`、`mobile/` 等） |
| **按模块** | "检查 article 模块的规范" | 该模块关联的全部文件（后端 12 + Web 前端 4） |
| **按文件** | "检查这个文件是否符合规范" / "检查 server/internal/logic/sys/article.go" | 单文件逐条对照 |

### 检查深度

默认执行 **Level 3（全深度）**，用户可指定降级。

| 级别 | 名称 | 检查内容 |
|------|------|---------|
| **Level 1** | 结构层 | 文件位置是否正确、命名规范（蛇形/驼峰）、是否手改了 `gf gen dao` 生成的代码、import 路径是否合规 |
| **Level 2** | 代码模式层（含 L1） | 是否使用了禁止 API（直调 dao、`fmt.Println`、裸 axios）、是否缺少必要写法（`onClose()`、try-catch、`loadOptions()`）、字段引用是否用 `dao.Columns()` 而非字符串硬编码、日志是否用统一工具 |
| **Level 3** | 架构层（含 L1+L2） | 分层调用链是否完整（controller→service→logic→dao）、Service 注册/引用是否完整、字典是否在 consts 中注册、路由是否在 genrouter 中注册、中间件/鉴权是否按规范配置、插件通信是否通过主模块 service 而非直接 import |

### 规则索引

AI 执行检查时，**按被检查的端**加载对应文档中的全部规则：

| 端 | 规则来源（必须全部读取） |
|---|----------------------|
| **后端 (Go)** | `02-后端-开发规范` + `03-后端-进阶主题` + `08-通用-常见错误与禁止清单` §8.1 + `10-后端-HotGo详解与扩展点` + `11-后端-数据库建表规范` + `12-后端-中间件体系规范` |
| **Web 前端 (Vue3)** | `04-后台-开发规范` + `08-通用-常见错误与禁止清单` §8.2 |
| **移动端 (Flutter)** | `05-移动端-开发规范` + `08-通用-常见错误与禁止清单` §8.3 |
| **桌面端 (Flutter Desktop)** | `06-桌面端-开发规范` + `08-通用-常见错误与禁止清单` §8.4 |
| **小程序 (Uni-app)** | `07-小程序-开发规范` + `08-通用-常见错误与禁止清单` §8.5 |

### 标准检查流程

```
Step 1: 诊断扫描 → 生成报告
  ├── AI 逐文件扫描，按规则索引 + 禁止矩阵对照
  └── 生成结构化报告：

  ## 规范检查报告 — [项目名 / 模块名 / 文件名]

  ### 概览
  | 端 | 扫描文件数 | 违规数 | 🔴 严重 | 🟡 警告 | 🔵 建议 |
  |---|----------|-------|---------|---------|---------|
  | 后端 | 45 | 12 | 3 | 6 | 3 |
  | Web | 12 | 4 | 1 | 2 | 1 |

  ### 🔴 严重问题（必须修复 — 违反核心铁律或触犯禁止清单）
  | # | 文件:行号 | 违规类型 | 当前写法 | 规范要求 |
  |---|---------|---------|---------|---------|
  | 1 | controller/sys/article.go:15 | 跨层调用 | controller 直接调用 dao.Xxx | controller → service.Xxx() → logic → dao |
  | 2 | logic/sys/user.go:42 | 禁止API | fmt.Println("debug:", user) | g.Log().Infof(ctx, "debug: %+v", user) |
  | 3 | service/member.go:1 | 文件位置错误 | 在 sys.go 之外新建了 service 文件 | 统一追加到 internal/service/sys.go |
  | 4 | views/article/model.ts:55 | 缺少必要写法 | dict.getOptions() 未包裹 computed | computed(() => dict.getOptions(...)) |

  ### 🟡 警告（强烈建议修复 — 偏离推荐模式，但不阻塞运行）
  | # | 文件:行号 | 违规类型 | 当前写法 | 规范要求 |
  |---|---------|---------|---------|---------|
  | 5 | pages/login/controller.dart:23 | 缺少 onClose | Controller 无 onClose() | 所有 Controller 必须有 onClose() |

  ### 🔵 建议（可选改进 — 不影响功能，但可以让代码更规范）
  | # | 文件:行号 | 改进点 | 建议 |
  |---|---------|-------|------|
  | 6 | consts/article.go:10 | 常量未分组注释 | 按类型分组并加注释分隔 |

  ---
  共发现 3 严重 + 1 警告 + 1 建议。请选择修复范围。

Step 2: 用户确认修复范围
  AI 输出报告后询问用户：
  - "全部修复（严重+警告+建议）？"
  - "只修复严重+警告？"
  - "只修复严重问题？"
  - "逐项选择？（请告诉我要修哪些编号）"
  - "跳过，只保留报告？"

Step 3: 批量修复
  用户确认后，AI 逐文件修改：
  - 每修复一个文件，标注进度（如：✓ 已修复 1/3 — controller/sys/article.go）
  - 修复内容严格对照规范要求，不凭空发挥
  - 修复完成后输出汇总：

  ### 修复完成
  | 状态 | 数量 | 明细 |
  |------|------|------|
  | ✅ 已修复 | 4 | #1, #2, #4, #5 |
  | ⏭️ 已跳过 | 1 | #6（用户选择跳过） |
  | ❌ 需手动处理 | 0 | — |

Step 4: 修复后验证
  全部修复完成后，AI 对修改过的文件再做一轮快速检查：
  - 确认修复项没有引入新的违规（零新增）
  - 确认修复方案本身没有违反其他规范
  - 输出验证结果：

  ### 修复后验证
  | 检查项 | 结果 |
  |-------|------|
  | 新增违规 | 0 |
  | 残留违规 | 0 |
  | 修复完整性 | ✅ 全部通过 |
```

- [ ] **Step 2: 验证 Part 3 完整性**

对照 spec 检查：
- [ ] 新建 CRUD 模块章节完整（三步流程 + 16 文件清单）
- [ ] 规范检查与修复章节含：触发方式（2 种）、检查范围（4 级）、检查深度（3 级）、规则索引（5 端）、标准检查流程（4 步）
- [ ] 报告模板含概览表 + 严重/警告/建议三类问题
- [ ] Step 2 含 5 种用户选择（全部/严重+警告/仅严重/逐项/跳过）
- [ ] Step 3 含修复进度标注和汇总表
- [ ] Step 4 含验证检查表

---

### Task 4: 最终验证

**Files:**
- Check: `E:\Projects\skills\hotgo-dev\SKILL.md`

- [ ] **Step 1: 结构验证**

打开 SKILL.md，逐项确认：
- [ ] frontmatter 完整（name, description）
- [ ] Part 1 标题 `# Part 1：知识获取` 存在
- [ ] Part 2 标题 `# Part 2：行为规则` 存在
- [ ] Part 3 标题 `# Part 3：执行流程` 存在
- [ ] 三个 Part 之间用 `---` 分隔
- [ ] 与 goframe-v2 skill 协作 位于 Part 1 之前

- [ ] **Step 2: 完整性验证**

逐项确认以下 element 都存在且正确：
- [ ] 文档索引表 13 行
- [ ] 各端速查表 5 行 × 5 列
- [ ] 核心原则 10 条（1-10 编号）
- [ ] 回答四步链路表 4 行（①-④）
- [ ] 禁止事项矩阵 5 个子表
- [ ] 分端回答策略表 5 行
- [ ] 新建 CRUD 16 文件清单
- [ ] 规范检查触发方式表 2 行
- [ ] 规范检查范围表 4 行
- [ ] 检查深度表 3 行（Level 1-3）
- [ ] 规则索引表 5 行
- [ ] 标准检查流程 4 步（Step 1-4）
- [ ] 报告模板含概览表 + 🔴严重 + 🟡警告 + 🔵建议
- [ ] 修复后验证表

- [ ] **Step 3: 链接有效性验证**

用 Grep 搜索所有 `references/` 引用，确认引用的文件名与实际文件名一致：
```
references/README.md
references/01-通用-项目概览与技术栈.md
references/02-后端-开发规范.md
references/03-后端-进阶主题.md
references/04-后台-开发规范.md
references/05-移动端-开发规范.md
references/06-桌面端-开发规范.md
references/07-小程序-开发规范.md
references/08-通用-常见错误与禁止清单.md
references/09-通用-完整CRUD开发流程.md
references/10-后端-HotGo详解与扩展点.md
references/11-后端-数据库建表规范.md
references/12-后端-中间件体系规范.md
```

- [ ] **Step 4: 输出最终统计**

报告最终 SKILL.md 的行数和结构概览。
```
