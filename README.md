# HotGo Dev Skill

> 基于《HotGo 深入浅出》全栈开发规范，为 AI 编码助手（Claude Code）提供 HotGo v2 项目的完整开发规范支持。

## 这是什么

`hotgo-dev` 是一个 Claude Code **skill**，内部包含 13 份（12 份规范文档 + 1 份总览导航）完整的 HotGo v2 全栈开发规范，以及一份 AI 行为规则文件（`SKILL.md`）。当你在 HotGo 项目中编写代码时，AI 会自动加载这些规范，确保生成的代码符合项目架构约定。

它不是代码库、不是 npm 包、不是框架——它是给 **AI 看的规则书**。

## 能做什么

### 1. 代码生成（按照规范）

让 AI 帮你写代码，自动遵循 HotGo 规范：

```
"帮我新建一个文章管理模块，包含后端 CRUD 和 Web 前端页面"
```

AI 会按规范的 16 个文件清单、正确的分层架构、标准的代码模板，逐个生成完整代码。你不会看到一个 controller 直接调 dao，不会看到硬编码的字段名，不会看到缺少 `onClose()` 的 Flutter Controller。

### 2. 规范检查与修复（像 ESLint 一样）

把现有项目交给 AI 检查，发现所有不合规的地方，然后自动修复：

```
"检查后端代码是否符合 HotGo 规范"
```

AI 会扫描代码，生成结构化报告（严重/警告/建议三档），你确认后逐文件修复。

**检查深度分三级：**

| 级别 | 检查内容 |
|------|---------|
| **Level 1 — 结构层** | 文件位置对不对、命名规不规范、有没有手改生成代码、import 路径对不对 |
| **Level 2 — 代码模式层** | 有没有用禁止 API（直调 dao、`fmt.Println`）、是否缺少必要写法（`onClose()`、try-catch） |
| **Level 3 — 架构层** | 分层调用链对不对、Service 有没有注册、字典有没有注册、路由有没有注册 |

**检查范围灵活：**
- 全项目扫描
- 按端扫描（只查后端 / 只查 Web 前端 / 只查移动端）
- 按模块扫描（只查某个业务模块的 16 个关联文件）
- 按文件扫描（单个文件逐一对照）

### 3. 禁止事项主动预警

AI 在写代码或 review 代码时，会自动对照 5 端共 36 条禁止规则，发现踩红线立即警告：

```
"你这行代码 controller 里直接调了 dao，应该改为 controller → service → logic → dao"
"这个 Flutter Controller 缺少 onClose()，必须加上"
"dict.getOptions() 没有包 computed，字典选项不会响应式更新"
```

### 4. 回答自带审查

AI 每次回答代码问题，都必须走完**四步链路**：读文档 → 自检 → 输出 → 审查。写完代码后还会再检查一遍有没有违规，有就当场修正再给你。

## 覆盖五大平台

| 端 | 技术栈 | 规范文档 |
|---|--------|---------|
| **后端** | Go + GoFrame v2 | 5 份（02 开发规范 · 03 进阶主题 · 10 详解与扩展点 · 11 建表规范 · 12 中间件体系） |
| **Web 后台** | Vue 3 + Naive UI + Pinia + Vite | 1 份（04 后台开发规范） |
| **移动端 App** | Flutter + GetX + TDesign + Dio | 1 份（05 移动端开发规范） |
| **桌面端** | Flutter Desktop + Forui + TDesign + window_manager | 1 份（06 桌面端开发规范） |
| **小程序** | Uni-app + wot-ui (wot-design-uni) | 1 份（07 小程序开发规范） |

通用文档：01 项目概览 · 08 禁止清单 · 09 CRUD 流程

## 核心原则（10 条铁律）

1. **分层严格**：api → controller → service → logic → dao，禁止跨层
2. **代码生成优先**：entity/do/dao 由 `gf gen dao` 生成，禁止手改
3. **扩展点优先**：改框架行为前先查预留的 Hook/Handler/接口
4. **鉴权分层**：Admin 走 JWT+Casbin RBAC，Api/Home 只验 JWT
5. **Identity.App 严填**：一个用户表对应一个 App 常量，禁止跨表混用
6. **字典后端统管**：前端禁止硬编码枚举，统一走字典接口
7. **UI 库唯一**：Web 只用 Naive UI，App 只用 TDesign Flutter
8. **错误快返**：Go 的 `err != nil` 后必须 `return`，Flutter 的 catch 里必须重置 isLoading
9. **日志统一**：Go 用 `g.Log()`，Flutter 用 `AppLogger`，禁止 `print`
10. **数据库双引擎**：建表同时考虑 MySQL 和 PostgreSQL 语法差异

## 怎么用

### 安装

把这个目录放到 Claude Code 能识别为 skill 的位置，比如项目里的 `.claude/skills/` 目录，或者全局 skills 目录。

如果你用 Claude Code 的 plugin 系统，也可以通过 plugin 方式加载。

### 触发

skill 会在以下情况自动触发：

- 你在 HotGo 项目中写 Go/Vue/Flutter/Uni-app 代码
- 你说"帮我新建一个 XX 管理模块"
- 你提到 "hotgo"、"gf gen dao"、"GoFrame" 等关键词
- 你贴代码让 AI review
- 你说"检查后端代码规范"或"按规范修复这个文件"

### 和 goframe-v2 skill 的协作

`hotgo-dev` 和 `goframe-v2` 两个 skill **互不冲突**，可以同时触发：

| 问题 | 由谁回答 |
|------|---------|
| "这个 Controller 文件应该放在哪个目录？" | **hotgo-dev**（文件组织结构） |
| "怎么用 DO 对象做 Update？" | **goframe-v2**（GoFrame 编码写法） |
| "新建一个 CRUD 模块要改哪些文件？" | **hotgo-dev**（16 文件清单 + 流程） |
| "gerror 怎么用？" | **goframe-v2**（GoFrame API） |

## 文件结构

```
hotgo-dev/
├── README.md                                          ← 你正在看的文件
├── SKILL.md                                           ← skill 主文件（三 Part 结构）
│
├── references/                                        ← 规范文档
│   ├── README.md                                      ← 文档导航 + 快速定位
│   ├── 01-通用-项目概览与技术栈.md
│   ├── 02-后端-开发规范.md
│   ├── 03-后端-进阶主题.md
│   ├── 04-后台-开发规范.md
│   ├── 05-移动端-开发规范.md
│   ├── 06-桌面端-开发规范.md
│   ├── 07-小程序-开发规范.md
│   ├── 08-通用-常见错误与禁止清单.md
│   ├── 09-通用-完整CRUD开发流程.md
│   ├── 10-后端-HotGo详解与扩展点.md
│   ├── 11-后端-数据库建表规范.md
│   └── 12-后端-中间件体系规范.md
│
└── docs/superpowers/                                  ← 设计文档
    ├── specs/2026-07-01-hotgo-dev-skill-improvements-design.md
    └── plans/2026-07-01-hotgo-dev-skill-rewrite.md
```

### SKILL.md 内部结构

skill 主文件按 **AI 的行为模式** 组织为三个 Part：

```
SKILL.md
├── Part 1：知识获取     ← AI 接到问题先在这定位要读哪些规范文档
│   ├── 文档索引（13 份文档的完整索引：内容 + 何时读取）
│   └── 各端速查（5 端的技术栈 + 全部相关文档 + 场景→章节映射）
│
├── Part 2：行为规则     ← AI 在所有回答中必须遵守的约束
│   ├── 核心原则（10 条铁律）
│   ├── 回答四步链路（读文档→自检→输出→审查）
│   ├── 禁止事项矩阵（5 端 × 36 条禁止规则）
│   └── 分端回答策略（每端回答重点 + 必须覆盖的内容）
│
└── Part 3：执行流程     ← AI 执行两个标准操作的标准流程
    ├── 新建 CRUD 模块（16 文件清单 + 三步流程）
    └── 规范检查与修复（诊断→确认→修复→验证 完整流程）
```

## 设计理念

### 为什么是"给 AI 看的规则书"

传统开发规范是给人看的文档——人读了规范，然后凭记忆写代码。人会忘、会出疏漏、会"这次赶时间先跳过"。

`hotgo-dev` 把规范变成了 AI 的**强制执行规则**。AI 不能"凭记忆回答"——必须先读文档。不能"赶时间跳过"——必须走完四步链路。写完代码不能"应该没问题吧"——必须逐条自检 36 条禁止清单。

这就像 ESLint 和团队编码规范的关系：人可以看规范文档，但 ESLint 会在你违反时直接报错。`hotgo-dev` 对 AI 的作用也是如此。

### 为什么按"知识→规则→执行"三段式组织

这是一个经过深思熟虑的设计决策。AI 的思考链路是：

1. 我先搞清楚这是什么问题 → **Part 1：知识获取**
2. 我该怎么做、不该怎么做 → **Part 2：行为规则**
3. 具体怎么操作 → **Part 3：执行流程**

三段式结构让 AI 读到哪做到哪，不会跳步，不会漏掉关键约束。比把所有内容线性堆在一起的"从头读到尾"结构更可靠。

## 贡献

- 规范文档有误或过时？修改 `references/` 下对应文件
- AI 行为规则需要调整？修改 `SKILL.md`
- 文档结构需要大改？参考 `docs/superpowers/specs/` 下的设计文档

## 相关资源

- [HotGo 官方仓库](https://github.com/bufanyun/hotgo)
- [GoFrame 官方文档](https://goframe.org)
- [Naive UI](https://www.naiveui.com)
- [TDesign Flutter](https://tdesign.tencent.com/flutter)
- [Flutter](https://flutter.dev)
- [Uni-app](https://uniapp.dcloud.net.cn)
- [wot-ui (wot-design-uni)](https://wot-ui.cn/) · [插件市场](https://ext.dcloud.net.cn/plugin?id=27534)
