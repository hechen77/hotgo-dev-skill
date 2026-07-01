# HotGo 详解 —— 框架设计哲学与完整架构剖析

> 本文档深度剖析 HotGo v2 框架的完整设计理念、架构思想和所有功能模块。基于 HotGo 官方源码和文档深入研究而成，涵盖框架全部内置能力与 40+ 处预留扩展点。

---

## 目录

1. [10.1 — 设计哲学](#101--设计哲学)
2. [10.2 — 整体架构概览](#102--整体架构概览)
3. [10.3 — 多服务进程架构](#103--多服务进程架构)
4. [10.4 — 插件系统深度剖析](#104--插件系统深度剖析)
5. [10.5 — 代码生成系统](#105--代码生成系统)
6. [10.6 — 字典系统](#106--字典系统)
7. [10.7 — 认证与权限体系](#107--认证与权限体系)
8. [10.8 — 缓存与存储驱动](#108--缓存与存储驱动)
9. [10.9 — 消息队列系统](#109--消息队列系统)
10. [10.10 — 定时任务系统](#1010--定时任务系统)
11. [10.11 — WebSocket 体系](#1011--websocket-体系)
12. [10.12 — TCP 服务器体系](#1012--tcp-服务器体系)
13. [10.13 — 支付网关](#1013--支付网关)
14. [10.14 — 多租户体系](#1014--多租户体系)
15. [10.15 — 国际化支持](#1015--国际化支持)
16. [10.16 — 中间件体系](#1016--中间件体系)
17. [10.17 — 第三方服务集成](#1017--第三方服务集成)
18. [10.18 — 部署与运维](#1018--部署与运维)
19. [10.19 — HTTP 请求生命周期钩子](#1019--http-请求生命周期钩子)
20. [10.20 — ORM 钩子系统](#1020--orm-钩子系统)
21. [10.21 — ORM 处理器体系](#1021--orm-处理器体系)
22. [10.22 — 全局事件系统](#1022--全局事件系统)
23. [10.23 — 路由注册扩展点](#1023--路由注册扩展点)
24. [10.24 — WebSocket 扩展点](#1024--websocket-扩展点)
25. [10.25 — TCP 网络扩展点](#1025--tcp-网络扩展点)
26. [10.26 — 支付与回调扩展点](#1026--支付与回调扩展点)
27. [10.27 — 驱动替换体系](#1027--驱动替换体系)
28. [10.28 — Redis 集群同步](#1028--redis-集群同步)
29. [10.29 — 日志与全局初始化钩子](#1029--日志与全局初始化钩子)
30. [10.30 — 请求预处理接口](#1030--请求预处理接口)

---

## 10.1 — 设计哲学

### 1.1 核心设计目标

HotGo 的设计哲学可以概括为四个关键词：

| 关键词 | 含义 |
|--------|------|
| **插件化** | 业务功能以插件形式组织，独立开发、独立部署、独立复用 |
| **代码生成** | 通过后台 UI 一键生成完整 CRUD 模块（前后端代码 + 菜单 + 权限） |
| **驱动可替换** | 缓存、队列、上传、短信、支付等基础设施全部抽象为接口，支持多驱动任意切换 |
| **约定优于配置** | 建表约定、目录结构约定、命名约定，减少决策成本 |

### 1.2 框架层级关系

```
┌─────────────────────────────────────────────────┐
│                  HotGo v2                       │
│  (插件系统 + 代码生成 + 后台管理 + 驱动适配)    │
├─────────────────────────────────────────────────┤
│               GoFrame v2                        │
│  (Web/ORM/Cache/Config/Log/Cron/Queue/TCP...)   │
├─────────────────────────────────────────────────┤
│                 Go 标准库                       │
└─────────────────────────────────────────────────┘
```

HotGo 不重复造轮子，而是基于 GoFrame v2 做了大量企业级增强：
- 在 GoFrame 的 Web 框架上增加了**插件化模块系统**
- 在 GoFrame 的 ORM 上增加了**数据权限过滤**和**多租户 Hook**
- 在 GoFrame 的缓存上增加了**统一驱动适配层**
- 封装了**可视化代码生成器**（后台 UI 操作，非 CLI 命令）
- 集成了**Casbin RBAC**、**JWT 令牌**、**支付网关**等企业必需组件

### 1.3 "主模块 + 插件" 的双层架构

HotGo 将业务代码分为两个层级：

```
主模块 (internal/)
    ├── 核心系统功能（用户/角色/菜单/配置/日志）
    ├── 基础设施（中间件/缓存/队列/定时任务）
    └── Service 接口（供插件调用）

插件模块 (addons/)
    ├── 独立微架构（与主模块结构镜像）
    ├── 通过 Service 接口与主模块通信
    └── 插件间完全隔离（禁止互相 import）
```

这种设计的优势：
- 核心系统保持稳定，业务功能灵活扩展
- 插件可以独立版本管理，跨项目复用
- 团队可以并行开发不同插件，减少代码冲突
- 插件可以独立安装/卸载/升级

---

## 10.2 — 整体架构概览

### 2.1 服务端架构全景图

```
                        ┌─────────────┐
                        │ Nginx/Caddy │
                        └──────┬──────┘
                               │
                ┌──────────────┼──────────────┐
                │              │              │
           ┌────▼────┐   ┌────▼────┐   ┌────▼────┐
           │  HTTP   │   │WebSocket│   │   TCP   │
           │ :8000   │   │ :8000   │   │ :8099   │
           └────┬────┘   └────┬────┘   └────┬────┘
                │              │              │
           ┌────▼──────────────▼──────────────▼────┐
           │           中间件链                    │
           │  Ctx → CORS → Blacklist → Addon →     │
           │  DeliverUser → Auth → Controller      │
           └────────────────┬──────────────────────┘
                            │
           ┌────────────────▼──────────────────────┐
           │          路由模块分发                 │
           │  /admin(后台)  /api(前台API)          │
           │  /home(移动端)  /socket(WebSocket)    │
           └────────────────┬──────────────────────┘
                            │
           ┌────────────────▼──────────────────────┐
           │          Controller 层 (薄层)         │
           └────────────────┬──────────────────────┘
                            │
           ┌────────────────▼──────────────────────┐
           │           Service 接口层              │
           └────────────────┬──────────────────────┘
                            │
           ┌────────────────▼──────────────────────┐
           │           Logic 业务逻辑层            │
           │  (插件逻辑 ←→ 主模块逻辑 通过Service) │
           └────────────────┬──────────────────────┘
                            │
           ┌────────────────▼──────────────────────┐
           │           DAO / ORM 层                │
           │  (handler: 权限过滤 + 多租户过滤)     │
           │  (hook: 自动维护租户关系)             │
           └────────────────┬──────────────────────┘
                            │
           ┌────────────────▼──────────────────────┐
           │         PostgreSQL / MySQL            │
           └───────────────────────────────────────┘
```

### 2.2 多进程独立部署架构

HotGo 的设计从一开始就考虑了微服务化部署的可能。它将 HTTP 服务、消息队列消费、定时任务、TCP 服务拆分为四个独立可启动的进程：

```
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│   HTTP   │  │   Queue  │  │   Cron   │  │   TCP    │
│   进程   │  │   进程   │  │   进程   │  │   进程   │
│  :8000   │  │          │  │          │  │  :8099   │
└────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │             │             │
     └─────────────┴─────────────┴─────────────┘
                      │
               ┌──────▼──────┐
               │    Redis    │ ← 进程间协调
               └─────────────┘
```

**启动方式**：

```bash
# 开发环境：一键启动所有服务
go run main.go                    # 或 gf run main.go

# 生产环境：独立部署
go run main.go http               # 仅 HTTP + WebSocket
go run main.go queue              # 仅消息队列消费者
go run main.go cron               # 仅定时任务调度
go run main.go auth               # 仅 TCP 认证服务
```

这种设计的好处：
- 队列消费和定时任务可独立扩缩容
- 某个服务重启不影响其他服务
- 可以按需部署（例如不需要定时任务的场景只部署 HTTP 和 Queue）

---

## 10.3 — 多服务进程架构

### 3.1 CMD 命令体系

HotGo 基于 GoFrame 的 `gcmd` 实现了灵活的命令行系统：

```go
// server/internal/cmd/cmd.go
// 注册的子命令：
//   http    - HTTP + WebSocket 服务
//   queue   - 消息队列消费者
//   cron    - 定时任务调度器
//   auth    - TCP 认证服务器
//   tools   - 常用工具（如刷新 Casbin 权限）
//   up      - 系统升级
```

### 3.2 优雅关闭

HotGo 实现了优雅的信号处理和关闭流程：

```go
// server/internal/cmd/handler_shutdown.go
// 1. 监听 SIGINT/SIGTERM 信号
// 2. 停止接收新请求
// 3. 等待当前请求处理完成
// 4. 关闭数据库连接池
// 5. 关闭 Redis 连接
// 6. 清理临时文件
```

### 3.3 TCP 内部通信

HTTP 服务通过内置的 TCP 客户端与定时任务服务进行 RPC 通信：

```yaml
# config.yaml
tcp:
  server:
    address: ":8099"
  client:
    cron:
      group: "cron"
      name: "cron1"
      address: "127.0.0.1:8099"
      appId: "1002"
      secretKey: "hotgo"
```

这使得定时任务可以独立部署，后台仍然可以通过 HTTP → TCP RPC 调用来管理任务。

---

## 10.4 — 插件系统深度剖析

### 4.1 插件生命周期

```
创建 → 安装 → 启动 → 运行 → 停止 → 升级 → 卸载
  │      │      │                      │       │
  │      │      │                      │       └→ UnInstall()
  │      │      │                      └→ Upgrade()
  │      │      └→ Start() → Stop()
  │      └→ Install()
  └→ newModule() → addons.RegisterModule()
```

### 4.2 Module 接口

```go
// server/internal/library/addons/module.go

type Module interface {
    Start(option *Option) error          // 启动插件
    Stop() error                         // 停止插件
    Ctx() context.Context                // 获取上下文
    GetSkeleton() *Skeleton              // 获取骨架信息
    Install(ctx context.Context) error   // 安装（建表/初始化数据/迁移文件）
    Upgrade(ctx context.Context) error   // 升级
    UnInstall(ctx context.Context) error // 卸载（清理表/清理文件）
}
```

### 4.3 Skeleton 骨架

每个插件带有一套完整的元信息：

```go
type Skeleton struct {
    Label       string      // 显示名称
    Name        string      // 唯一标识（与目录名一致）
    Group       int         // 分组
    Logo        string      // Logo
    Brief       string      // 一句话简介
    Description string      // 详细描述
    Author      string      // 作者
    Version     string      // 版本号
    RootPath    string      // 插件根路径
    View        *gview.View // 模板引擎实例
}
```

### 4.4 插件注册与发现

```
启动流程:
main.go
  → import _ "hotgo/addons/modules"     (隐式注册所有插件)
    → 各插件 main.go 的 init()
      → newModule() → addons.RegisterModule(m)
  → addons.StartAllModules()
    → 遍历已安装的插件
      → module.Start(option)
        → global.Init(ctx, skeleton)
        → router.Admin(ctx, group)     (注册后台路由)
        → router.Api(ctx, group)       (注册前台 API 路由)
        → router.Home(ctx, group)      (注册首页路由)
```

### 4.5 插件的完全隔离

插件之间的隔离体现在多个层面：

| 层面             | 实现机制                                                                |
| -------------- | ------------------------------------------------------------------- |
| **包隔离**        | 每个插件独立的 Go package 路径 `hotgo/addons/<name>/`                        |
| **路由隔离**       | 每个插件独立的路由前缀 `/admin/<name>/`、`/api/<name>/`                         |
| **Service 隔离** | 每个插件有自己独立的 service 包，不与主模块混用                                        |
| **数据隔离**       | 插件表建议以 `hg_addon_<name>_` 为前缀；多租户通过 `tenant_id` 隔离                  |
| **前端隔离**       | 前端代码在 `web/src/views/addons/<name>/` 和 `web/src/api/addons/<name>/` |

### 4.6 跨插件通信的唯一通道

插件不直接 import 另一个插件的包，而是通过主模块的 Service 接口：

```
插件A → 主模块Service接口 → 主模块Logic → [数据库/缓存/...]
插件B → 主模块Service接口 → 主模块Logic → [数据库/缓存/...]
插件A ← 不允许直接引用 → 插件B
```

---

## 10.5 — 代码生成系统

### 5.1 生成系统架构

HotGo 的代码生成系统是其最具特色的功能。它不是一个简单的 CLI 工具，而是一套完整的**可视化代码生成平台**。

```
后台管理界面（开发工具 → 代码生成）
    │
    ▼
选择数据库 → 选择表 → 配置字段/关联/菜单
    │
    ▼
模板引擎渲染
    │
    ├── Go API 层 (api/admin/)
    ├── Controller 层 (internal/controller/)
    ├── Logic 层 (internal/logic/)
    ├── Model Input 层 (internal/model/input/)
    ├── Router 层 (internal/router/genrouter/)
    ├── Web API 层 (web/src/api/)
    ├── Web 页面 (web/src/views/)
    ├── SQL 文件
    └── 菜单权限数据
```

### 5.2 三种生成模式

| 模式 | 适用场景 | 表要求 |
|------|---------|--------|
| **普通 CRUD** | 标准增删改查列表 | `id` 主键 |
| **关联表 CRUD** | 带外键关联的列表 | 主表 + 关联表，主表有 `xxx_id` 字段 |
| **树形 CRUD** | 层级结构（部门/分类/菜单） | `id` + `pid` + `level` + `tree` |

### 5.3 生成模板系统

```yaml
# config.yaml - hggen 配置
hggen:
  application:
    crud:
      templates:
        - group: "default"           # 主模块模板
          isAddon: false
          templatePath: "./resource/generate/default/curd"
          apiPath: "./api/admin"
          controllerPath: "./internal/controller/admin/sys"
          logicPath: "./internal/logic/sys"
          inputPath: "./internal/model/input/sysin"
          routerPath: "./internal/router/genrouter"
          webApiPath: "../web/src/api"
          webViewsPath: "../web/src/views"
        - group: "addon"             # 插件模板
          isAddon: true
          templatePath: "./resource/generate/default/curd"
          # {$name} 自动替换为插件名
          apiPath: "./addons/{$name}/api/admin"
          # ...
    queue:
      templates:
        - group: "default"
          templatePath: "./resource/generate/default/queue"
    cron:
      templates:
        - group: "default"
          templatePath: "./resource/generate/default/cron"
```

模板可以完全自定义，开发者可以基于 default 模板修改，创建自己的生成模板分组。

### 5.4 内置 gf-cli

HotGo 将 GoFrame 的 CLI 工具完整内置到 `server/internal/library/hggen/internal/` 中，使得：
- 不依赖外部 `gf` 命令安装
- 代码生成功能在后台 UI 即可操作
- 可以固定 gf-cli 版本，避免版本升级带来的不兼容
- 支持在线执行 `gf gen dao`、`gf gen service`、`gf gen enums` 等命令

### 5.5 数据库智能映射

HotGo 的代码生成器能根据数据库字段类型和名称**智能推断**：

| 数据库特征 | 自动推断 |
|-----------|---------|
| `status` 字段 (tinyint) | 使用系统状态字典，表单用 Select 组件 |
| `created_at` 字段 (datetime) | 列表用时间范围查询，表单隐藏（自动维护） |
| `sort` 字段 | 自动获取最大排序值 +1 填充 |
| `mobile` 字段 | 手机号格式校验 |
| `email` 字段 | 邮箱格式校验 |
| 字段 NOT NULL | 必填验证 |
| UNI 索引 | 唯一性验证 |
| text 类型 (>500) | 富文本编辑器 |
| varchar (200-500) | 文本域 |

### 5.6 强制覆盖与增量生成

代码生成支持两种模式：
- **增量生成**：不覆盖已有文件，仅生成新文件
- **强制覆盖**：完全覆盖已有文件（适合初始开发或模板升级后重新生成）

### 5.7 多数据库支持

```yaml
database:
  default:
    link: "mysql:..."
    Prefix: "hg_"
  default2:
    link: "pgsql:..."
    Prefix: ""
```

代码生成器可以从多个数据库中选表生成，生成后的 DAO 代码会自动带上对应的 group 标识。

---

## 10.6 — 字典系统

### 6.1 字典类型总览

HotGo 提供三种字典类型：

| 类型 | 存储方式 | ID 特征 | 适用场景 |
|------|---------|---------|---------|
| **系统字典** | 表 `hg_sys_dict_type` + `hg_sys_dict_data` | >0 的 int64 | 运营人员通过后台维护的动态数据 |
| **枚举字典** | 代码 `consts` 中定义 + `init()` 注册 | 以 -20000 开头 | 开发阶段固定的枚举值 |
| **方法字典** | 实现 `FuncDict` 接口 + `init()` 注册 | 以 -30000 开头 | 从数据库/文件/第三方动态获取的选项 |

### 6.2 内置字典和系统字典的分工

```
系统字典：运营可改，后台界面维护
    └── hg_sys_dict_type: "支付方式"
    └── hg_sys_dict_data: {key:"wechat", label:"微信支付"}, {key:"alipay", label:"支付宝"}

枚举字典：开发者定义，代码中固定
    └── consts.StatusEnabled = 1, consts.StatusDisabled = 2
    └── dict.RegisterEnums("statusOptions", "状态选项", StatusOptions)

方法字典：动态获取，代码中定义获取逻辑
    └── dict.RegisterFunc("postOptions", "岗位选项", service.AdminPost().Option)
    └── 每次调用时从 hg_admin_post 表实时查询
```

### 6.3 字典工作流程

```
1. 后端定义 + 注册
   → 2. 前端通过固定接口拉取所有字典数据
   → 3. 前端 dict Store 缓存
   → 4. 表格/表单/搜索 通过 dict.getLabel()/dict.getOptions() 渲染
```

### 6.4 代码生成中的字典关联

在生成 CRUD 时，字段如果属于字典类型，系统会**自动生成**字典渲染代码：
- `status` 字段 → 自动关联系统状态字典
- 自定义字典字段 → 可在生成配置中指定字典类型

---

## 10.7 — 认证与权限体系

### 7.1 双令牌机制

HotGo 采用 **JWT + 缓存** 的混合令牌方案：

```yaml
token:
  secretKey: "hotgo123"        # JWT 签名密钥
  expires: 604800              # 7天有效期（秒）
  autoRefresh: true            # 自动续约
  refreshInterval: 86400       # 1天内只允许刷新一次
  maxRefreshTimes: 30          # 最大刷新次数
  multiLogin: true             # 允许多端登录
```

**为什么不用纯 JWT？**
- 纯 JWT 无法主动失效（服务端无法"踢人"）
- HotGo 在 JWT 基础上叠加 Redis 缓存
- 登录时：生成 JWT → 存入 Redis
- 鉴权时：先验证 JWT → 再查 Redis 是否存在
- 登出时：删除 Redis 中的记录 → JWT 即使未过期也无法使用

### 7.2 四种认证中间件

HotGo 为四大路由模块分别实现了鉴权中间件：

| 中间件 | 适用模块 | 鉴权方式 |
|--------|---------|---------|
| `AdminAuth` | `/admin` | JWT + Casbin RBAC |
| `ApiAuth` | `/api` | JWT + 可配置豁免路径 |
| `HomeAuth` | `/home` | JWT + 可配置豁免路径 |
| `WebSocketAuth` | `/socket` | URL 参数 token 鉴权 |

### 7.3 Casbin RBAC 权限模型

```
用户 (User) → 角色 (Role) → 菜单权限 (Menu Permission)
                              └── API 权限 (/xxx/list, /xxx/edit)
                              └── 按钮权限 (hasPermission)
```

**权限验证流程**：
```
1. 用户请求 /admin/member/list
2. AdminAuth 中间件 → 验证 JWT
3. 从 JWT 中获取用户角色
4. Casbin Enforcer.Enforce(userRole, /member/list, GET)
5. 有权限 → 继续处理
6. 无权限 → 返回 62 状态码 "无访问权限"
```

**权限刷新**：
```bash
go run main.go tools -m=casbin -a1=refresh
```
菜单权限添加或修改后，API 权限实时生效，无需重启服务。Casbin 规则通过 WebSocket 实时同步到内存。

### 7.4 数据权限（行级过滤）

除了菜单权限（控制能访问哪些接口），HotGo 还支持**数据权限**（控制能看到哪些数据）：

| 数据范围 | 说明 |
|---------|------|
| 全部权限 | 不做过滤 |
| 当前部门 | 仅看所属部门数据 |
| 当前及下级部门 | 所属 + 下级部门 |
| 自定义部门 | 指定多个部门 |
| 仅自己 | `created_by = 当前用户ID` |
| 自己和直属下级 | 自己 + 直属下级用户的数据 |
| 自己和全部下级 | 自己 + 所有下级用户的数据 |

使用方式：
```go
// 在 ORM 链中加 Handler 即可
dao.Xxx.Ctx(ctx).Handler(handler.FilterAuth).Scan(&res)
```

---

## 10.8 — 缓存与存储驱动

### 8.1 缓存驱动

HotGo 将缓存抽象为统一接口 `cache.Instance()`，底层可切换驱动：

```yaml
cache:
  adapter: "file"              # memory | redis | file
  fileDir: "./storage/cache"   # adapter=file 时生效
```

| 驱动 | 特点 | 适用场景 |
|------|------|---------|
| `memory` | 进程内存，最快，重启丢失 | 开发环境 |
| `redis` | 分布式共享，持久化 | 生产环境、多实例 |
| `file` | 文件存储，不依赖外部服务 | 单机部署、简单场景 |

多进程部署时建议使用 `redis` 驱动，以保证多进程共享缓存（HTTP + Queue + Cron 进程间数据一致）。

### 8.2 上传/存储驱动

HotGo 支持多种云存储驱动：

| 驱动 | 配置 |
|------|------|
| 本地存储 | 默认，文件存在 `storage/` |
| 阿里云 OSS | 需配置 AccessKey + Bucket |
| 腾讯云 COS | 需配置 SecretId + Bucket |
| 七牛云 | 需配置 AccessKey + Bucket |
| MinIO | 私有化对象存储 |

驱动切换只需在后台"系统设置 → 配置管理 → 上传配置"中修改，无需改代码。

### 8.3 消息队列驱动

```yaml
queue:
  driver: "disk"               # disk | redis | rocketmq | kafka
```

| 驱动 | 特点 |
|------|------|
| `disk` | 默认，文件队列，无需外部依赖 |
| `redis` | 支持延迟队列 |
| `rocketmq` | 企业级消息队列 |
| `kafka` | 高吞吐量 |

---

## 10.9 — 消息队列系统

### 9.1 队列架构

HotGo 的消息队列基于生产者-消费者模式：

```
生产者 (Producer)                 消费者 (Consumer)
     │                                  │
     ├─ queue.Push(topic, data) ──→  ┌──┴──┐
     │                               │ 队列│
     └─ queue.SendDelayMsg(...) ──→  │ 存储│
                                     └──┬──┘
                                        │
消费者进程 (go run main.go queue)       │
     │                                  │
     ├─ Consumer1.Handle(msg) ←────────┘
     ├─ Consumer2.Handle(msg)
     └─ Consumer3.Handle(msg)
```

### 9.2 Consumer 接口

```go
type Consumer interface {
    GetTopic() string
    Handle(ctx context.Context, mqMsg MqMsg) error
}
```

任何实现了该接口的结构体，通过 `queue.RegisterConsumer()` 在 `init()` 中注册，即可自动被消费者进程加载。

### 9.3 延迟队列

```go
// Redis 延迟队列（延迟秒数）
queue.SendDelayMsg(consts.QueueLogTopic, data, 10)

// RocketMQ 延迟队列（延迟级别：1s 5s 10s 30s 1m 2m ... 2h）
queue.SendDelayMsg(consts.QueueLogTopic, data, 2)
```

---

## 10.10 — 定时任务系统

### 10.1 设计特点

HotGo 的定时任务系统最大的特点是**后台可视化管理**：

- **代码注册**：在代码中定义任务逻辑
- **后台配置**：在后台配置 cron 表达式、启用/禁用
- **在线操作**：支持手动立即执行、查看调度日志
- **进程分离**：定时任务可独立进程运行，通过 TCP RPC 与 HTTP 服务通信

### 10.2 Cron 接口

```go
type Cron interface {
    GetName() string
    Execute(ctx context.Context, parser *Parser) error
}
```

### 10.3 调度日志

每次任务执行都会记录：
- 执行时间
- 执行状态（成功/失败）
- 错误信息（如果失败）
- 任务耗时

---

## 10.11 — WebSocket 体系

### 11.1 服务端架构

```
WebSocket 连接请求
    │
    ▼
WebSocketAuth 中间件（URL中的token鉴权）
    │
    ▼
Client 管理（Manager）
    ├── clientMap: map[clientID]*Client
    ├── userClientMap: map[userID][]*Client  (支持多端)
    └── tagMap: map[tag][]*Client            (群组)
    │
    ▼
Message Router（事件路由）
    ├── EventHandler "admin/xxx/test" → handler
    ├── EventHandler "kick" → kickHandler
    └── ...
    │
    ▼
具体 Handler 处理 → SendSuccess/SendError 回复
```

### 11.2 消息格式

```json
{
    "event": "admin/addons/hgexample/testMessage",
    "data": { "message": "hello" },
    "timestamp": 1684145107
}
```

### 11.3 发送方式

```go
websocket.SendToAll()                        // 广播所有人
websocket.SendToClientID(clientID, msg)      // 发给指定连接
websocket.SendToUser(userID, msg)            // 发给指定用户（所有端）
websocket.SendToTag(tag, msg)                // 发给指定群组
websocket.SendSuccess(client, event, data)   // 成功响应
websocket.SendError(client, event, err)      // 错误响应
```

### 11.4 客户端封装

HotGo 提供了完整的 Web 端 WebSocket 客户端封装：

```ts
// 全局注册消息监听
addOnMessage(SocketEnum.EventKick, kickHandler);

// 单页面注册监听（页面销毁时自动移除）
onMounted(() => addOnMessage(event, handler));
onBeforeUnmount(() => removeOnMessage(event));

// 发送消息
sendMsg(event, data);
```

---

## 10.12 — TCP 服务器体系

### 12.1 设计目的

HotGo 基于 GoFrame 的 `gtcp` 组件构建了一套 TCP 服务器框架，用于：

1. **内部服务通信**：HTTP 服务与定时任务服务之间的 RPC 通信
2. **第三方授权**：为外部系统提供 TCP 授权验证服务
3. **自定义 TCP 应用**：如物联网设备通信、游戏服务器等

### 12.2 Server/Client 双模式

```go
// 服务端
serv := tcp.NewServer(&tcp.ServerConfig{
    Name: "hotgo",
    Addr: ":8002",
})

// 注册普通消息路由
serv.RegisterRouter(onMessage)

// 注册 RPC 消息路由（有返回值）
serv.RegisterRPCRouter(onRPCMessage)

// 注册拦截器
serv.RegisterInterceptor(interceptor1, interceptor2)

// 启动
serv.Listen()

// ======

// 客户端
client := tcp.NewClient(&tcp.ClientConfig{
    Addr:          "127.0.0.1:8002",
    AutoReconnect: true,
    Auth: &tcp.AuthMeta{
        Group:     "cron",
        Name:      "cron1",
        AppId:     "1002",
        SecretKey: "hotgo",
    },
})

// 普通消息（不等待回复）
conn.Send(ctx, &TestMsgReq{Name: "Tom"})

// RPC 消息（等待回复）
var res TestRPCMsgRes
conn.RequestScan(ctx, &TestRPCMsgReq{Name: "Tony"}, &res)
```

### 12.3 消息类型

| 类型 | 发送方式 | 是否等待回复 | 适用场景 |
|------|---------|-------------|---------|
| 普通消息 | `conn.Send()` | 否 | 通知、推送 |
| RPC 消息 | `conn.RequestScan()` | 是 | 查询、指令 |

### 12.4 许可证认证

TCP 客户端连接时需要提供许可证（AppId + SecretKey），后台"系统监控 → 在线服务 → 许可证列表"中可以管理授权。

### 12.5 拦截器链

```
请求 → Interceptor1 → Interceptor2 → Interceptor3 → 路由处理
         │               │               │
         └─ 可中断返回错误 └─ 可中断返回错误 └─ 可中断返回错误
```

---

## 10.13 — 支付网关

### 13.1 设计概览

HotGo 内置了统一支付网关，基于 [go-pay/gopay](https://github.com/go-pay/gopay) 封装：

```
业务层
    │
    ▼
service.Pay().Create()      ← 统一支付创建接口
service.PayRefund().Refund() ← 统一退款接口
    │
    ▼
支付网关层 (Payment Gateway)
    ├── 微信支付 (wxpay)
    │   ├── 扫码支付 (scan)
    │   ├── 公众号支付 (jsapi)
    │   ├── App支付 (app)
    │   └── H5支付 (h5)
    ├── 支付宝 (alipay)
    │   ├── 扫码支付
    │   ├── 网页支付
    │   └── App支付
    └── QQ支付 (qqpay)
```

### 13.2 支付流程

```
1. 用户下单
2. 业务服务创建订单
3. 调用 service.Pay().Create() 创建支付单
4. 获取支付链接/二维码
5. 用户完成支付
6. 支付平台异步通知 → 支付网关验签
7. 验签通过 → 回调业务订单处理
8. 更新订单状态 → 发放产品
```

### 13.3 回调注册

```go
// server/internal/logic/pay/notify.go
func (s *sPay) RegisterNotifyCall() {
    payment.RegisterNotifyCallMap(map[string]payment.NotifyCallFunc{
        consts.OrderGroupAdminOrder: service.AdminOrder().PayNotify,
        // 添加更多回调...
    })
}
```

---

## 10.14 — 多租户体系

### 14.1 多租户模型

HotGo 采用**共享数据库 + Schema 隔离**方案，通过 `tenant_id` 字段区分租户数据：

```
同一数据库
├── hg_tenant (租户表)
├── hg_xxx (业务表，含 tenant_id)
└── hg_yyy (业务表，含 tenant_id)
```

### 14.2 四层身份体系

| 身份 | 标识字段 | 职责 |
|------|---------|------|
| Company | `dept_type=company` | 管理整个平台 |
| Tenant | `tenant_id` | 独立租户实体 |
| Merchant | `merchant_id` | 租户下的商户 |
| User | `user_id` | 最终消费者 |

### 14.3 自动维护租户关系

```go
// 查询：自动过滤租户数据
func (s *sXxx) Model(ctx context.Context) *gdb.Model {
    return handler.Model(dao.Xxx.Ctx(ctx), &handler.Option{
        FilterTenant: true,    // 自动加 WHERE tenant_id = ?
    })
}

// 增改：自动写入租户信息
dao.Xxx.Ctx(ctx).
    Fields(XxxInsertFields{}).
    Hook(hook.SaveTenant).    // 自动设置 tenant_id/merchant_id/user_id
    Data(in).Insert()
```

### 14.4 身份判断

```go
// 后端
contexts.IsCompanyDept(ctx)
contexts.IsTenantDept(ctx)
contexts.IsMerchantDept(ctx)
contexts.IsUserDept(ctx)

// 前端
userStore.isCompanyDept
userStore.isTenantDept
userStore.isMerchantDept
userStore.isUserDept
```

---

## 10.15 — 国际化支持

### 15.1 支持语言

HotGo v2.18.6 起内置三种语言包：

| 语言 | 标识 |
|------|------|
| 简体中文 | `zh-CN`（默认） |
| 繁体中文 | `zh-TW` |
| 英文 | `en` |

### 15.2 配置

```yaml
system:
  i18n:
    switch: true
    defaultLanguage: "zh-CN"
```

### 15.3 使用方式

**后端**：
```go
// 设置语言
gi18n.WithLanguage(ctx, "en")

// 翻译
gi18n.T(ctx, "你好，美丽世界")           // → "Hello, Beautiful World"
gi18n.Tf(ctx, "剩余%v余额", 100)        // → "Remaining 100 Balance"
```

**前端**：
```vue
<div>{{ t('你好，美丽世界') }}</div>              <!-- → Hello, Beautiful World -->
<div>{{ t('剩余{num}余额', {num:100}) }}</div>  <!-- → Remaining 100 Balance -->
```

后台右上角自动显示语言切换按钮。

---

## 10.16 — 中间件体系

### 16.1 全局中间件列表

| 顺序 | 中间件 | 功能 |
|------|--------|------|
| 1 | `Ctx` | 初始化上下文、i18n、链路追踪 |
| 2 | `CORS` | 跨域请求处理 |
| 3 | `Blacklist` | IP 黑名单拦截 |
| 4 | `DemoLimit` | 演示模式：禁止 POST 请求 |
| 5 | `Addon` | 从 URL 解析插件模块名 |
| 6 | `PreFilter` | 请求输入预处理（隐式调用 Filter 接口） |
| 7 | `DeliverUserContext` | JWT 解析，按 App 类型绑定用户到 context |
| 8 | `ResponseHandler` | 统一响应格式化 + 错误过滤 |

### 16.2 鉴权中间件（路由组级别）

| 中间件 | 适用模块 |
|--------|---------|
| `AdminAuth` | `/admin` 路由组 |
| `ApiAuth` | `/api` 路由组 |
| `HomeAuth` | `/home` 路由组 |
| `WebSocketAuth` | `/socket` 路由组 |

### 16.3 响应中间件

- **统一响应格式**：`{ code, message, timestamp, traceID, data }`
- **错误过滤**：debug 模式返回完整堆栈，生产模式只返回友好提示
- **多格式支持**：JSON（默认）、XML、HTML、SSE（Server-Sent Events）

### 16.4 前置过滤中间件

当 API 的 `XxxReq` 结构体实现了 `validate.Filter` 接口时，`PreFilter` 中间件会自动调用其 `Filter(ctx)` 方法：

```go
// 实现了 Filter 接口
type EditReq struct {
    g.Meta `path:"/xxx/edit" method:"post"`
    sysin.XxxEditInp
}

// PreFilter 中间件自动调用 XxxEditInp.Filter(ctx)
```

---

## 10.17 — 第三方服务集成

### 17.1 短信服务

HotGo 内置短信发送驱动：

| 驱动 | 配置 |
|------|------|
| 阿里云短信 | 需配置 AccessKey + 签名 + 模板 |
| 腾讯云短信 | 需配置 AppId + 签名 + 模板 |

```go
// 发送短信
service.Sms().Send(ctx, mobile, templateCode, params)
```

### 17.2 邮件服务

内置邮件发送功能（基于 SMTP）。

### 17.3 微信公众号

内置微信公众号集成：
- 自动获取 OpenID
- AccessToken 自动管理和刷新
- 微信 JS SDK 配置

### 17.4 地理位置

- IP 归属地查询
- 内置缓存机制避免重复查询

---

## 10.18 — 部署与运维

### 18.1 编译打包

HotGo 支持将 Web 前端静态资源和后端模板打包进单个可执行文件：

```yaml
gfcli:
  build:
    packSrc: "resource"                    # 打包 resource 目录
    packDst: "internal/packed/packed.go"   # 生成 Go 嵌入代码
    output: "./temp/hotgo"                 # 可执行文件
```

```bash
# 一键编译
make build
# 或分步
cd web && pnpm run build
cd ../server && gf build
```

### 18.2 Nginx 反向代理

```nginx
# WebSocket（必须配置 Upgrade 头）
location = /socket {
    proxy_pass http://127.0.0.1:8000/socket;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
}

# HTTP
location / {
    proxy_pass http://127.0.0.1:8000/;
}
```

### 18.3 模式控制

```yaml
system:
  mode: "dev"     # dev: 显示开发工具; product: 隐藏开发工具
  debug: true     # 开启详细错误信息
```

---

## 10.19 — HTTP 请求生命周期钩子

### 19.1 IHook 接口 — 全局 HTTP 钩子

这是在 HTTP 服务启动时注册的两个全局钩子，作用于每一个 HTTP 请求的生命周期：

**接口定义** — `server/internal/service/hook.go`：

```go
type IHook interface {
    BeforeServe(r *ghttp.Request)
    AfterOutput(r *ghttp.Request)
}
```

**注册方式** — `server/internal/logic/hook/init.go`：

```go
type sHook struct{}

func init() {
    service.RegisterHook(New())
}
```

**挂载到 HTTP Server** — `server/internal/cmd/http.go`：

```go
s.BindHookHandler("/*any", ghttp.HookBeforeServe, service.Hook().BeforeServe)
s.BindHookHandler("/*any", ghttp.HookAfterOutput, service.Hook().AfterOutput)
```

**默认实现的功能**：

| 钩子方法 | 执行时机 | 默认行为 |
|---------|---------|---------|
| `BeforeServe` | 每个 HTTP 请求开始处理之前 | 记录访问日志、更新管理员最后活跃时间 |
| `AfterOutput` | 每个 HTTP 响应输出之后 | 写入服务日志、登录日志 |

**如何自定义**：实现 `service.IHook` 接口，替换 `service.RegisterHook()` 注册。可在 `BeforeServe` 中添加自定义请求前置逻辑（统计、限流、审计），在 `AfterOutput` 中添加后置处理（自定义日志格式、埋点上报、慢查询告警等）。

### 19.2 GoFrame 原生 HTTP Hook

GoFrame `ghttp.Server` 支持更细粒度的原生钩子——你可以在任何路由上按需注册：

```go
// GoFrame 内置支持的 Hook 常量
ghttp.HookBeforeServe    // 请求进入前
ghttp.HookAfterServe     // 请求离开后
ghttp.HookBeforeOutput   // 响应输出前
ghttp.HookAfterOutput    // 响应输出后
ghttp.HookBeforeBind     // 参数绑定前
ghttp.HookAfterBind      // 参数绑定后

// 绑定到特定路由
s.BindHookHandler("/custom/*path", ghttp.HookBeforeServe, yourHandlerFunc)
```

这意味着你可以在全局 `IHook` 之外，为特定的路径或路径模式添加专属的钩子回调。

---

## 10.20 — ORM 钩子系统

HotGo 基于 GoFrame 的 `gdb.HookHandler` 构建了一套完整的 ORM 生命周期钩子系统：

```go
// GoFrame 内置类型
type HookHandler struct {
    Select func(ctx context.Context, in *HookSelectInput) (result Result, err error)
    Insert func(ctx context.Context, in *HookInsertInput) (result sql.Result, err error)
    Update func(ctx context.Context, in *HookUpdateInput) (result sql.Result, err error)
    Delete func(ctx context.Context, in *HookDeleteInput) (result sql.Result, err error)
}
```

所有 Hook 位于 `server/internal/library/hgorm/hook/` 目录。

### 20.1 hook.MemberInfo — 自动填充成员详情

**文件**：`server/internal/library/hgorm/hook/member.go`

在 `Select` 后自动将 `dept_id` → `deptName`、`role_id` → `roleName`，并清除 `password_hash`、`salt`、`auth_key` 等敏感字段。

```go
dao.AdminMember.Ctx(ctx).Hook(hook.MemberInfo).Scan(&list)
```

### 20.2 hook.MemberSummary — 自动填充操作人摘要

**文件**：`server/internal/library/hgorm/hook/member.go`

在 `Select` 后自动将 `created_by` / `updated_by` / `deleted_by` / `member_id` 替换为包含 `{id, realName, username, avatar}` 的摘要对象。前端渲染时可直接使用这些摘要展示操作人信息。

```go
dao.XxxTable.Ctx(ctx).Hook(hook.MemberSummary).Scan(&list)
```

### 20.3 hook.SaveTenant — 自动维护多租户关系

**文件**：`server/internal/library/hgorm/hook/tenant.go`

在 `Insert` 和 `Update` 时根据当前用户的部门类型自动填充 `tenant_id`、`merchant_id`、`user_id`：

| 用户身份 | 自动填充行为 |
|---------|------------|
| Company（平台级） | `tenant_id=0, merchant_id=0, user_id=0`（全可见） |
| Tenant（租户） | 自动填充当前租户ID |
| Merchant（商户） | 自动填充租户ID + 商户ID |
| User（普通用户） | 自动填充全部三级ID |

```go
dao.Xxx.Ctx(ctx).Hook(hook.SaveTenant).Data(in).Insert()
```

### 20.4 hook.CityLabel — 自动填充行政区划标签

**文件**：`server/internal/library/hgorm/hook/provinces.go`

在 `Select` 后自动将 `city_id` / `province_id` 替换为对应的省市名称文本。

### 20.5 自定义 ORM Hook

开发者完全可以创建自己的 ORM Hook，并叠加使用：

```go
var MyBusinessHook = gdb.HookHandler{
    Select: func(ctx context.Context, in *gdb.HookSelectInput) (result gdb.Result, err error) {
        result, err = in.Next(ctx) // 先执行实际查询
        // 后置处理：字段脱敏、数据转换、扩展信息填充...
        return
    },
    Insert: func(ctx context.Context, in *gdb.HookInsertInput) (result sql.Result, err error) {
        in.Data["app_id"] = GetAppId(ctx) // 自动注入业务字段
        return in.Next(ctx)
    },
    Update: func(ctx context.Context, in *gdb.HookUpdateInput) (result sql.Result, err error) {
        in.Data["version"] = gdb.Raw("version+1") // 乐观锁版本号自增
        return in.Next(ctx)
    },
}

// 可以叠加多个 Hook
model.Hook(hook.MemberSummary).Hook(MyBusinessHook).Scan(&list)
```

---

## 10.21 — ORM 处理器体系

### 21.1 Hook 与 Handler 的区别

| 机制          | 作用阶段                    | 用法              |
| ----------- | ----------------------- | --------------- |
| **Hook**    | 数据库操作前后，可拦截和修改 SQL 执行结果 | `.Hook(xxx)`    |
| **Handler** | 构建 SQL 之前，修改查询条件        | `.Handler(xxx)` |

Handler 是 `func(m *gdb.Model) *gdb.Model` 类型的纯函数。

### 21.2 统一入口：`handler.Model()`

**文件**：`server/internal/library/hgorm/handler/handler.go`

```go
type Option struct {
    FilterAuth   bool // 是否开启数据权限过滤
    FilterTenant bool // 是否开启多租户权限过滤
    ForceCache   bool // 是否强制走缓存
}

func Model(m *gdb.Model, opt ...*Option) *gdb.Model
```

所有 Logic 层的 `Model()` 方法都应该调用这个统一入口，确保权限过滤和缓存策略的一致性。

### 21.3 handler.FilterAuth — 角色数据权限过滤

**文件**：`server/internal/library/hgorm/handler/filter_auth.go`

根据当前登录用户的角色数据权限范围自动添加 WHERE 条件。支持 7 种数据范围：全部 / 当前部门 / 当前及下级 / 自定义部门 / 仅自己 / 自己和直属下级 / 自己和全部下级。

```go
// 默认过滤 created_by 或 member_id 字段
dao.Xxx.Ctx(ctx).Handler(handler.FilterAuth).Scan(&res)

// 指定自定义过滤字段
dao.Xxx.Ctx(ctx).Handler(handler.FilterAuthWithField("owner_id")).Scan(&res)
```

### 21.4 handler.FilterTenant — 多租户数据过滤

**文件**：`server/internal/library/hgorm/handler/tenant.go`

根据用户身份自动 WHERE `tenant_id` / `merchant_id` / `user_id`。

### 21.5 handler.ForceCache — 强制查询缓存

**文件**：`server/internal/library/hgorm/handler/force_cache.go`

对当前查询强制开启 GoFrame 的 Model Cache。

### 21.6 handler.Sorter — 多列排序处理器

**文件**：`server/internal/library/hgorm/handler/sorter.go`

```go
type ISorter interface {
    GetSorters() []*form.Sorter
}
```

只需让你的请求 Input 结构体实现 `ISorter` 接口，然后 `.Handler(handler.Sorter(in))` 即可自动生成多列排序的 ORDER BY 语句。

---

## 10.22 — 全局事件系统

### 22.1 simple.Event — 全局事件总线

**文件**：`server/utility/simple/event.go`

```go
type EventFunc func(ctx context.Context, args ...interface{})

simple.Event().Register(group string, callback EventFunc)   // 注册监听
simple.Event().Call(group string, ctx context.Context, args ...interface{})  // 触发事件
simple.Event().Remove(group string)                          // 移除某组监听
simple.Event().Clear()                                       // 清空全部监听
```

### 22.2 内置事件

**事件常量** — `server/internal/consts/event.go`：

```go
const EventServerClose = "server.close"  // 服务关闭事件
```

### 22.3 已注册的关闭监听

- **Jaeger 链路追踪服务**：关闭时清理 tracer（`internal/global/init.go`）
- **RocketMQ 队列**：关闭时断开 producer/consumer（`library/queue/rocketmq.go`）
- **KafkaMQ 队列**：关闭时断开 producer/consumer（`library/queue/kafkamq.go`）

### 22.4 自定义事件示例

```go
// 1. 定义事件常量
const EventOrderPaid = "order.paid"

// 2. 在业务模块 init() 中注册监听
func init() {
    simple.Event().Register(EventOrderPaid, func(ctx context.Context, args ...interface{}) {
        orderId := args[0].(int64)
        // 发送通知、生成报表、触发后续流程...
    })
}

// 3. 在支付成功回调中触发
simple.Event().Call(EventOrderPaid, ctx, orderId)
```

---

## 10.23 — 路由注册扩展点

### 23.1 genrouter — 零侵入模块路由注册

**文件**：`server/internal/router/genrouter/init.go`

```go
var (
    NoLoginRouter       []interface{}  // 不需要登录的控制器列表
    LoginRequiredRouter []interface{}  // 需要登录的控制器列表
)
```

**使用方式**：为每个新模块在 `genrouter/` 下新建一个文件：

```go
// genrouter/your_module.go
package genrouter

import "hotgo/internal/controller/admin/sys"

func init() {
    LoginRequiredRouter = append(LoginRequiredRouter, sys.YourModule)
}
```

这种设计的巧妙之处在于：添加新模块只需新增文件，**完全不改动主路由文件** `admin.go`。Go 的 `init()` 函数自动完成注册链的串联。

### 23.2 插件版 genrouter

插件的 genrouter 额外接受 `ctx` 和 `group` 参数，并使用 `addons.RouterPrefix()` 生成插件专属路由前缀：

```go
func Register(ctx context.Context, group *ghttp.RouterGroup) {
    prefix := addons.RouterPrefix(ctx, consts.AppAdmin, global.GetSkeleton().Name)
    group.Group(prefix, func(group *ghttp.RouterGroup) {
        group.Middleware(service.Middleware().AdminAuth)
        if len(LoginRequiredRouter) > 0 {
            group.Bind(LoginRequiredRouter...)
        }
    })
}
```

### 23.3 IMiddleware — 中间件完整可替换

**文件**：`server/internal/service/middleware.go`

```go
type IMiddleware interface {
    Ctx(r *ghttp.Request)
    CORS(r *ghttp.Request)
    Blacklist(r *ghttp.Request)
    DemoLimit(r *ghttp.Request)
    Addon(r *ghttp.Request)
    DeliverUserContext(r *ghttp.Request) (err error)
    PreFilter(r *ghttp.Request)
    ResponseHandler(r *ghttp.Request)
    AdminAuth(r *ghttp.Request)
    ApiAuth(r *ghttp.Request)
    HomeAuth(r *ghttp.Request)
    WebSocketAuth(r *ghttp.Request)
    Develop(r *ghttp.Request)
    IsExceptAuth(ctx context.Context, appName string, path string) bool
    IsExceptLogin(ctx context.Context, appName string, path string) bool
}
```

通过 `service.RegisterMiddleware()` 注册你自己的实现，即可替换任何一个中间件的行为——修改 CORS 策略、自定义鉴权逻辑、增加请求限流、替换响应格式等。

---

## 10.24 — WebSocket 扩展点

### 24.1 EventHandler — 消息路由注册

**文件**：`server/internal/websocket/router.go`

```go
type EventHandler func(client *Client, req *WRequest)
type EventHandlers map[string]EventHandler

// 注册消息路由
websocket.RegisterMsg(handlers EventHandlers)
```

**注册示例** — 插件 `router/websocket.go`：

```go
ws.RegisterMsg(ws.EventHandlers{
    "admin/addons/hgexample/testMessage": handler.Index.TestMessage,
    "admin/addons/hgexample/orderNotify": handler.Order.Notify,
    // 持续添加新的消息事件...
})
```

### 24.2 Client 生命周期事件

**文件**：`server/internal/websocket/client_manager.go`

```go
EventRegister(client *Client)    // 新连接建立时
EventLogin(login *login)         // 用户认证通过时
EventUnregister(client *Client)  // 连接断开时
```

可扩展用于实现上线通知、离线消息队列、多端同步（踢掉同一用户的旧连接）等。

### 24.3 标签系统

```go
client.AddTag("room_123")
client.RemoveTag("room_123")
websocket.Manager().GetClientByTag("room_123")  // 获取标签下所有客户端
websocket.SendToTag("room_123", response)       // 向标签群组广播
```

通过标签系统可以构建聊天室、多房间订阅、按业务维度分组推送等场景。

---

## 10.25 — TCP 网络扩展点

### 25.1 客户端回调事件

**文件**：`server/internal/library/network/tcp/client.go`

```go
type CallbackEvent func()

type ClientConfig struct {
    Addr          string
    AutoReconnect bool
    Auth          *AuthMeta
    LoginEvent    CallbackEvent  // 登录成功后的回调
    CloseEvent    CallbackEvent  // 连接关闭后的回调
}
```

### 25.2 方法路由注册

```go
client.RegisterRouter(handlers ...interface{})      // 普通消息路由
client.RegisterRPCRouter(handlers ...interface{})   // RPC 消息路由（等待返回值）
server.RegisterRouter(handlers ...interface{})
server.RegisterRPCRouter(handlers ...interface{})
```

### 25.3 拦截器链

**文件**：`server/internal/library/network/tcp/msg_parser.go`

```go
type Interceptor func(ctx context.Context, msg *Message) (err error)

client.RegisterInterceptor(interceptors ...Interceptor)
server.RegisterInterceptor(interceptors ...Interceptor)
```

执行顺序与注册顺序一致，任意一个返回 error 则中断后续所有处理。适用场景：鉴权验证、消息解密、协议转换、流量控制等。

### 25.4 许可证认证机制

TCP 客户端连接需要提供 `AppId + SecretKey`，服务端通过后台"在线许可证"管理授权。未授权或已过期的连接将被拒绝。

---

## 10.26 — 支付与回调扩展点

### 26.1 支付成功回调注册

**文件**：`server/internal/library/payment/notifycall.go`

```go
type NotifyCallFunc func(ctx context.Context, pay *payin.NotifyCallFuncInp) (err error)

payment.RegisterNotifyCall(group string, f NotifyCallFunc)          // 单个注册
payment.RegisterNotifyCallMap(calls map[string]NotifyCallFunc)      // 批量注册
```

**示例** — `server/internal/logic/pay/notify.go`：

```go
func (s *sPay) RegisterNotifyCall() {
    payment.RegisterNotifyCallMap(map[string]payment.NotifyCallFunc{
        consts.OrderGroupAdminOrder:  service.AdminOrder().PayNotify,   // 后台充值订单
        consts.OrderGroupMemberOrder: service.MemberOrder().PayNotify,  // 会员订单
        // 持续注册更多业务回调...
    })
}
```

### 26.2 PayClient 支付驱动接口

**文件**：`server/internal/library/payment/payment.go`

```go
type PayClient interface {
    CreateOrder(ctx context.Context, in payin.CreateOrderInp) (res *payin.CreateOrderModel, err error)
    Notify(ctx context.Context, in payin.NotifyInp) (res *payin.NotifyModel, err error)
    Refund(ctx context.Context, in payin.RefundInp) (res *payin.RefundModel, err error)
}
```

如果要新增支付渠道（如 Stripe、Apple Pay），只需实现 `PayClient` 接口并在 `payment.New()` 中添加对应分支即可。

---

## 10.27 — 驱动替换体系

HotGo 将多个基础设施抽象为接口，通过配置文件切换驱动。每种驱动都是可替换、可扩展的：

| 基础设施 | 接口文件 | 配置位置 | 内置驱动 |
|---------|---------|---------|---------|
| 缓存 | `library/cache/cache.go` | `cache.adapter` | memory, redis, file |
| 消息队列 | `library/queue/queue.go` | `queue.driver` | disk, redis, rocketmq, kafka |
| 短信 | `library/sms/sms.go` | 后台配置管理 | 阿里云, 腾讯云 |
| 文件存储 | `library/storager/upload.go` | 后台配置管理 | Local, UCloud, COS, OSS, QiNiu, Minio |
| 支付 | `library/payment/payment.go` | 后台配置管理 | 微信支付, 支付宝, QQ支付 |

以存储驱动为例，新增驱动只需实现一个接口：

```go
type UploadDrive interface {
    Upload(ctx context.Context, file *ghttp.UploadFile) (fullPath string, err error)
    CreateMultipart(ctx context.Context, in *CheckMultipartParams) (res *MultipartProgress, err error)
    UploadPart(ctx context.Context, in *UploadPartParams) (res *UploadPartModel, err error)
}
```

然后在 `storager.New()` 中注册即可。

---

## 10.28 — Redis 集群同步

### 28.1 Pub/Sub 跨进程同步

**文件**：`server/internal/library/hgrds/pubsub/subscribe.go`

```go
type SubHandler func(ctx context.Context, message *gredis.Message)

pubsub.Subscribe(channel string, hr SubHandler)                              // 单频道订阅
pubsub.SubscribeMap(channels map[string]SubHandler)                           // 多频道订阅
pubsub.Publish(ctx context.Context, channel string, message interface{})     // 发布消息
```

### 28.2 内置同步频道

**文件**：`server/internal/global/cluster.go`

```go
func SubscribeClusterSync(ctx context.Context) {
    pubsub.SubscribeMap(map[string]pubsub.SubHandler{
        consts.ClusterSyncSysconfig:    handler,  // 系统配置变更 → 所有节点实时同步
        consts.ClusterSyncSysBlacklist: handler,  // IP 黑名单变更 → 所有节点实时同步
        consts.ClusterSyncSysSuperAdmin: handler, // 超管数据变更 → 所有节点实时同步
    })
}
```

当 HTTP 服务有多个实例时，通过 Redis Pub/Sub 实时同步配置变更，确保所有节点一致。开发者可按相同模式添加自己的同步频道。

---

## 10.29 — 日志与全局初始化钩子

### 29.1 自定义日志处理器

**文件**：`server/internal/global/init.go`

```go
// 在全局初始化时设置自定义日志处理器
glog.SetDefaultHandler(LoggingServeLogHandler)
```

`LoggingServeLogHandler` 拦截所有日志输出，将异常级别的日志自动写入数据库的服务日志表，实现了日志的持久化和可追溯。

**如何扩展**：你可以替换或追加自己的 Handler，实现日志转发到 ELK、报警触发等。

### 29.2 全局初始化函数

**文件**：`server/internal/global/init.go`

```go
func Init(ctx context.Context) {
    SetGFMode(ctx)                            // 设置 GoFrame 运行模式
    glog.SetDefaultHandler(LoggingServeLogHandler) // ← 日志钩子，可替换
    gtime.SetTimeZone("Asia/Shanghai")        // 设置时区
    InitTrace(ctx)                            // 初始化链路追踪
    cache.SetAdapter(ctx)                     // ← 缓存驱动选择，可扩展
    service.SysConfig().InitConfig(ctx)       // 加载系统配置
    service.AdminMember().LoadSuperAdmin(ctx) // 加载超管数据
    SubscribeClusterSync(ctx)                 // ← 集群同步订阅，可追加频道
}
```

在 `Init()` 中可以添加自己的初始化逻辑——预热缓存、注册自定义组件、启动后台协程等。

### 29.3 Casbin 权限热刷新

**文件**：`server/internal/library/casbin/enforcer.go`

```go
casbin.InitEnforcer(ctx)   // 初始化 Casbin 权限规则
casbin.Refresh(ctx)        // 热刷新所有权限策略（无需重启服务）
casbin.Clear(ctx)          // 清空所有策略
```

权限变更后通过 WebSocket 实时同步到各节点的内存中，无需重启。

---

## 10.30 — 请求预处理接口

### 30.1 validate.Filter — 隐式接口

**文件**：`server/utility/validate/filter.go`

```go
type Filter interface {
    Filter(ctx context.Context) error
}
```

### 30.2 自动调用机制

这是一个**隐式接口**——不需要显式注册。`PreFilter` 中间件会在请求到达 Controller 之前，自动检测 `XxxReq` 是否实现了 `Filter` 接口，如果实现了就自动调用：

```go
// 只要你的 Input 结构体实现了 Filter 方法
func (in *XxxEditInp) Filter(ctx context.Context) error {
    // 1. 复杂的业务前置校验
    // 2. 默认值设置
    // 3. 数据格式转换
    return nil
}

// PreFilter 中间件自动调用，无需在 Controller 中手动处理
```

---

## 扩展点速查总表

以下是 HotGo 全部 40 个扩展点的完整速查：

| #   | 类别          | 扩展点                             | 注册/使用方式                                      | 源码位置                                   |
| --- | ----------- | ------------------------------- | -------------------------------------------- | -------------------------------------- |
| 1   | HTTP        | `IHook.BeforeServe/AfterOutput` | `service.RegisterHook()`                     | `internal/service/hook.go`             |
| 2   | HTTP        | `s.BindHookHandler()` 6种事件      | GoFrame 原生                                   | `internal/cmd/http.go`                 |
| 3   | ORM Hook    | `hook.MemberInfo`               | `.Hook(hook.MemberInfo)`                     | `library/hgorm/hook/member.go`         |
| 4   | ORM Hook    | `hook.MemberSummary`            | `.Hook(hook.MemberSummary)`                  | `library/hgorm/hook/member.go`         |
| 5   | ORM Hook    | `hook.SaveTenant`               | `.Hook(hook.SaveTenant)`                     | `library/hgorm/hook/tenant.go`         |
| 6   | ORM Hook    | `hook.CityLabel`                | `.Hook(hook.CityLabel)`                      | `library/hgorm/hook/provinces.go`      |
| 7   | ORM Hook    | 自定义 `gdb.HookHandler`           | `.Hook(MyHandler)`                           | 按需创建                                   |
| 8   | ORM Handler | `handler.FilterAuth`            | `.Handler(handler.FilterAuth)`               | `library/hgorm/handler/filter_auth.go` |
| 9   | ORM Handler | `handler.FilterAuthWithField`   | `.Handler(handler.FilterAuthWithField("f"))` | `library/hgorm/handler/filter_auth.go` |
| 10  | ORM Handler | `handler.FilterTenant`          | `.Handler(handler.FilterTenant)`             | `library/hgorm/handler/tenant.go`      |
| 11  | ORM Handler | `handler.ForceCache`            | `.Handler(handler.ForceCache)`               | `library/hgorm/handler/force_cache.go` |
| 12  | ORM Handler | `handler.Sorter` + `ISorter`    | `.Handler(handler.Sorter(in))`               | `library/hgorm/handler/sorter.go`      |
| 13  | ORM Handler | `handler.Model()` 统一入口          | `handler.Model(m, &Option{...})`             | `library/hgorm/handler/handler.go`     |
| 14  | 全局事件        | `simple.Event().Register/Call`  | `simple.Event().Register(n, cb)`             | `utility/simple/event.go`              |
| 15  | 路由          | `genrouter.LoginRequiredRouter` | `append(...)` 切片追加                           | `internal/router/genrouter/init.go`    |
| 16  | 路由          | `genrouter.NoLoginRouter`       | `append(...)` 切片追加                           | `internal/router/genrouter/init.go`    |
| 17  | 路由          | 插件 genrouter `Register()`       | 传入 ctx + group                               | `addons/*/router/genrouter/`           |
| 18  | 中间件         | `IMiddleware` 接口（14个方法）         | `service.RegisterMiddleware()`               | `internal/service/middleware.go`       |
| 19  | WebSocket   | `EventHandler`                  | `websocket.RegisterMsg()`                    | `internal/websocket/router.go`         |
| 20  | WebSocket   | Client 生命周期 3个事件                | `EventRegister/Login/Unregister`             | `internal/websocket/client_manager.go` |
| 21  | WebSocket   | Tag标签分组                         | `AddTag/RemoveTag/SendToTag`                 | `internal/websocket/`                  |
| 22  | TCP         | `CallbackEvent` 两个回调            | 配置 `ClientConfig.LoginEvent/CloseEvent`      | `library/network/tcp/client.go`        |
| 23  | TCP         | `RegisterRouter`                | `client/server.RegisterRouter(...)`          | `library/network/tcp/`                 |
| 24  | TCP         | `RegisterRPCRouter`             | `client/server.RegisterRPCRouter(...)`       | `library/network/tcp/`                 |
| 25  | TCP         | `RegisterInterceptor`           | `client/server.RegisterInterceptor(...)`     | `library/network/tcp/msg_parser.go`    |
| 26  | 支付          | `NotifyCallFunc` 回调             | `payment.RegisterNotifyCall()`               | `library/payment/notifycall.go`        |
| 27  | 支付          | `PayClient` 驱动接口                | 实现接口 + `payment.New()`                       | `library/payment/payment.go`           |
| 28  | 短信          | `sms.Drive` 驱动接口                | 实现接口 + `sms.New()`                           | `library/sms/sms.go`                   |
| 29  | 存储          | `storager.UploadDrive` 驱动接口     | 实现接口 + `storager.New()`                      | `library/storager/upload.go`           |
| 30  | 缓存          | `gcache.Adapter`                | `cache.SetAdapter()`                         | `library/cache/cache.go`               |
| 31  | 队列生产者       | `MqProducer` 接口                 | `Register*MqProducer()`                      | `library/queue/queue.go`               |
| 32  | 队列消费者       | `queue.Consumer` 接口             | `queue.RegisterConsumer()`                   | `library/queue/consumer.go`            |
| 33  | Redis       | `pubsub.SubHandler`             | `pubsub.Subscribe(ch, h)`                    | `library/hgrds/pubsub/subscribe.go`    |
| 34  | Dict        | `dict.RegisterEnums()`          | `dict.RegisterEnums(k,l,opts)`               | `library/dict/dict_register_enums.go`  |
| 35  | Dict        | `dict.RegisterFunc()`           | `dict.RegisterFunc(k,l,fun)`                 | `library/dict/dict_register_func.go`   |
| 36  | 日志          | `glog.SetDefaultHandler()`      | 设置全局日志拦截器                                    | `internal/global/init.go`              |
| 37  | 全局初始化       | `global.Init()`                 | 在 cmds 中调用                                   | `internal/global/init.go`              |
| 38  | 权限          | `casbin.InitEnforcer/Refresh`   | Casbin 热刷新                                   | `library/casbin/enforcer.go`           |
| 39  | 树表          | `hgorm.AutoUpdateTree`          | 自动维护树形关系                                     | `library/hgorm/dao_tree.go`            |
| 40  | 请求预处理       | `validate.Filter` 接口            | 隐式实现（无需注册）                                   | `utility/validate/filter.go`           |



---
## 总结

HotGo 是一个功能非常全面的企业级全栈框架，其核心优势在于：

1. **极致开发效率**：可视化代码生成器 + 约定优于配置，10 分钟完成一个 CRUD 模块
2. **灵活架构**：插件化 + 多进程部署，从小项目到大项目平滑过渡
3. **企业级完备**：内置 RBAC、多租户、支付、短信、WebSocket、TCP、消息队列等全套组件
4. **驱动可替换**：缓存/队列/上传/短信/支付全部支持多驱动无缝切换
5. **丰富的扩展点**：HTTP 钩子、ORM 钩子与处理器、全局事件、TCP 拦截器、WebSocket 事件路由、驱动适配器、字典注册等 40+ 处预留扩展点
6. **GoFrame 生态**：站在 GoFrame 的肩膀上，享受其完善的文档和活跃的社区
