# 09 - 完整 CRUD 模块开发流程（端到端）

> 从数据库建表到前端页面渲染，完整演示一个标准 CRUD 模块的开发全流程。

---

## 目录

- [9.1 流程总览](#9.1-流程总览)
- [9.2 第一步：数据库建表](#9.2-第一步：数据库建表)
- [9.3 第二步：生成代码](#9.3-第二步：生成代码)
- [9.4 第三步：常量 + 字典](#9.4-第三步：常量-+-字典)
- [9.5 第四步：Model 入参/出参](#9.5-第四步：model-入参-出参)
- [9.6 第五步：Service 接口声明](#9.6-第五步：service-接口声明)
- [9.7 第六步：Logic 业务实现](#9.7-第六步：logic-业务实现)
- [9.8 第七步：Controller 薄控制器](#9.8-第七步：controller-薄控制器)
- [9.9 第八步：API 契约](#9.9-第八步：api-契约)
- [9.10 第九步：路由注册](#9.10-第九步：路由注册)
- [9.11 第十步：Web 前端 API 封装](#9.11-第十步：web-前端-api-封装)
- [9.12 第十一步：Web 前端 model.ts](#9.12-第十一步：web-前端-model.ts)
- [9.13 第十二步：后端后台添加菜单权限](#9.13-第十二步：后端后台添加菜单权限)
- [9.14 完整的文件清单（16 个文件）](#9.14-完整的文件清单-16-个文件)

---


## 9.1 流程总览

```
建表（数据库）
  → gf gen dao（生成 entity/do/dao）
  → 常量 + 字典注册（consts）
  → Model 入参/出参（model/input/sysin）
  → Service 接口声明（service/sys.go）
  → Logic 业务实现（logic/sys/）
  → Controller 薄控制器（controller/admin/sys/）
  → API 契约定义（api/admin/）
  → 路由注册（router/genrouter/）
  → Web 前端 API 封装（web/src/api/）
  → Web 前端 model.ts（State/schemas/columns）
  → Web 前端 index.vue（列表页）
  → Web 前端 edit.vue（编辑页）
  → 后端后台添加菜单 + 权限
```

---

## 9.2 第一步：数据库建表

```sql
CREATE TABLE `hg_article` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '文章ID',
    `title` varchar(255) NOT NULL COMMENT '文章标题',
    `content` text COMMENT '文章内容',
    `category_id` bigint(20) DEFAULT NULL COMMENT '分类ID',
    `cover_image` varchar(512) DEFAULT NULL COMMENT '封面图',
    `status` tinyint(1) DEFAULT '1' COMMENT '状态：1启用 2禁用',
    `sort` int(11) DEFAULT '0' COMMENT '排序',
    `remark` varchar(255) DEFAULT NULL COMMENT '备注',
    `created_at` datetime DEFAULT NULL COMMENT '创建时间',
    `updated_at` datetime DEFAULT NULL COMMENT '修改时间',
    `deleted_at` datetime DEFAULT NULL COMMENT '删除时间',
    `created_by` bigint(20) DEFAULT NULL COMMENT '创建者',
    `updated_by` bigint(20) DEFAULT NULL COMMENT '更新者',
    `deleted_by` bigint(20) DEFAULT NULL COMMENT '删除者',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='文章管理';
```

---

## 9.3 第二步：生成代码

```bash
cd server
gf gen dao
```

生成文件：
- `internal/model/entity/article.go`
- `internal/model/do/article.go`
- `internal/dao/internal/article.go`
- `internal/dao/article.go`

---

## 9.4 第三步：常量 + 字典

```go
// server/internal/consts/article.go
package consts

import (
    "hotgo/internal/library/dict"
    "hotgo/internal/model"
)

const (
    ArticleStatusEnabled  = 1
    ArticleStatusDisabled = 2
)

var ArticleStatusOptions = []*model.Option{
    dict.GenSuccessOption(ArticleStatusEnabled, "启用"),
    dict.GenWarningOption(ArticleStatusDisabled, "禁用"),
}

func init() {
    dict.RegisterEnums("ArticleStatusOptions", "文章状态选项", ArticleStatusOptions)
}
```

---

## 9.5 第四步：Model 入参/出参

```go
// server/internal/model/input/sysin/article.go
package sysin

import (
    "context"
    "hotgo/internal/model/entity"
    "hotgo/internal/model/input/form"
    "github.com/gogf/gf/v2/errors/gerror"
)

// 字段白名单
type ArticleUpdateFields struct {
    Title      string `json:"title"`
    Content    string `json:"content"`
    CategoryId int64  `json:"categoryId"`
    CoverImage string `json:"coverImage"`
    Status     int    `json:"status"`
    Sort       int    `json:"sort"`
    Remark     string `json:"remark"`
}
type ArticleInsertFields struct {
    ArticleUpdateFields
    CreatedBy int64 `json:"createdBy"`
}

// 编辑
type ArticleEditInp struct {
    entity.Article
}
func (in *ArticleEditInp) Filter(ctx context.Context) (err error) {
    if in.Title == "" {
        return gerror.New("文章标题不能为空")
    }
    return
}

// 列表
type ArticleListInp struct {
    form.PageInp
    Title  string `json:"title"`
    Status int    `json:"status"`
}
type ArticleListModel struct {
    Id        int64  `json:"id"`
    Title     string `json:"title"`
    Status    int    `json:"status"`
    Sort      int    `json:"sort"`
    CreatedAt string `json:"createdAt"`
}

// 删除
type ArticleDeleteInp struct {
    Ids []int64 `json:"ids"`
}

// 状态
type ArticleStatusInp struct {
    Id     int64 `json:"id"`
    Status int   `json:"status"`
}
```

---

## 9.6 第五步：Service 接口声明

在 `server/internal/service/sys.go` 末尾追加：

```go
// ==================== Article ====================

type ISysArticle interface {
    List(ctx context.Context, in *sysin.ArticleListInp) (list []*sysin.ArticleListModel, totalCount int, err error)
    Edit(ctx context.Context, in *sysin.ArticleEditInp) (id int64, err error)
    Delete(ctx context.Context, in *sysin.ArticleDeleteInp) (err error)
    Status(ctx context.Context, in *sysin.ArticleStatusInp) (err error)
}

var insSysArticle ISysArticle

func SysArticle() ISysArticle {
    if insSysArticle == nil {
        panic("SysArticle 未注册")
    }
    return insSysArticle
}

func RegisterSysArticle(i ISysArticle) {
    insSysArticle = i
}
```

---

## 9.7 第六步：Logic 业务实现

```go
// server/internal/logic/sys/article.go
package sys

import (
    "context"
    "hotgo/internal/dao"
    "hotgo/internal/library/hgorm/handler"
    "hotgo/internal/model/input/sysin"
    "hotgo/internal/service"
    "github.com/gogf/gf/v2/database/gdb"
    "github.com/gogf/gf/v2/errors/gerror"
)

type sSysArticle struct{}

func init() {
    service.RegisterSysArticle(NewSysArticle())
}

func NewSysArticle() *sSysArticle {
    return &sSysArticle{}
}

func (s *sSysArticle) Model(ctx context.Context, option ...*handler.Option) *gdb.Model {
    return handler.Model(dao.Article.Ctx(ctx), option...)
}

func (s *sSysArticle) List(ctx context.Context, in *sysin.ArticleListInp) (list []*sysin.ArticleListModel, totalCount int, err error) {
    mod := s.Model(ctx)
    if in.Title != "" {
        mod = mod.WhereLike(dao.Article.Columns().Title, "%"+in.Title+"%")
    }
    if in.Status > 0 {
        mod = mod.Where(dao.Article.Columns().Status, in.Status)
    }
    totalCount, err = mod.Count()
    if err != nil || totalCount == 0 {
        return
    }
    err = mod.Page(in.Page, in.PageSize).OrderDesc(dao.Article.Columns().Id).Scan(&list)
    return
}

func (s *sSysArticle) Edit(ctx context.Context, in *sysin.ArticleEditInp) (id int64, err error) {
    if err = in.Filter(ctx); err != nil {
        return
    }
    if in.Id > 0 {
        _, err = s.Model(ctx).Fields(sysin.ArticleUpdateFields{}).WherePri(in.Id).Data(in).Update()
        id = in.Id
    } else {
        _, err = s.Model(ctx).Fields(sysin.ArticleInsertFields{}).Data(in).InsertAndGetId()
    }
    return
}

func (s *sSysArticle) Delete(ctx context.Context, in *sysin.ArticleDeleteInp) (err error) {
    if len(in.Ids) == 0 {
        return gerror.New("请选择删除项")
    }
    _, err = s.Model(ctx).WhereIn(dao.Article.Columns().Id, in.Ids).Delete()
    return
}

func (s *sSysArticle) Status(ctx context.Context, in *sysin.ArticleStatusInp) (err error) {
    _, err = s.Model(ctx).WherePri(in.Id).Data(g.Map{dao.Article.Columns().Status: in.Status}).Update()
    return
}
```

---

## 9.8 第七步：Controller 薄控制器

```go
// server/internal/controller/admin/sys/article.go
package sys

import (
    "context"
    "hotgo/api/admin/article"
    "hotgo/internal/service"
)

var Article = cArticle{}

type cArticle struct{}

func (c *cArticle) List(ctx context.Context, req *article.ListReq) (res *article.ListRes, err error) {
    list, totalCount, err := service.SysArticle().List(ctx, &req.ArticleListInp)
    if err != nil {
        return
    }
    res = &article.ListRes{}
    res.List = list
    res.PageRes.Pack(req, totalCount)
    return
}

func (c *cArticle) Edit(ctx context.Context, req *article.EditReq) (res *article.EditRes, err error) {
    id, err := service.SysArticle().Edit(ctx, &req.ArticleEditInp)
    if err != nil {
        return
    }
    res = &article.EditRes{Id: id}
    return
}
```

---

## 9.9 第八步：API 契约

```go
// server/api/admin/article/article.go
package article

import (
    "hotgo/internal/model/input/form"
    "hotgo/internal/model/input/sysin"
    "github.com/gogf/gf/v2/frame/g"
)

type ListReq struct {
    g.Meta `path:"/article/list" method:"get" tags:"文章管理" summary:"获取文章列表"`
    sysin.ArticleListInp
}
type ListRes struct {
    form.PageRes
    List []*sysin.ArticleListModel `json:"list" dc:"文章列表"`
}

type EditReq struct {
    g.Meta `path:"/article/edit" method:"post" tags:"文章管理" summary:"新增/编辑文章"`
    sysin.ArticleEditInp
}
type EditRes struct {
    Id int64 `json:"id" dc:"文章ID"`
}

type DeleteReq struct {
    g.Meta `path:"/article/delete" method:"post" tags:"文章管理" summary:"删除文章"`
    sysin.ArticleDeleteInp
}
type DeleteRes struct{}

type StatusReq struct {
    g.Meta `path:"/article/status" method:"post" tags:"文章管理" summary:"切换文章状态"`
    sysin.ArticleStatusInp
}
type StatusRes struct{}
```

---

## 9.10 第九步：路由注册

```go
// server/internal/router/genrouter/article.go
package genrouter

import "hotgo/internal/controller/admin/sys"

func init() {
    LoginRequiredRouter = append(LoginRequiredRouter, sys.Article)
}
```

---

## 9.11 第十步：Web 前端 API 封装

```ts
// web/src/api/article/index.ts
import { http, jumpExport } from '@/utils/http/axios';

export function List(params) {
    return http.request({ url: '/article/list', method: 'get', params });
}
export function Edit(params) {
    return http.request({ url: '/article/edit', method: 'POST', params });
}
export function Delete(params) {
    return http.request({ url: '/article/delete', method: 'POST', params });
}
export function Status(params) {
    return http.request({ url: '/article/status', method: 'POST', params });
}
export function MaxSort(params) {
    return http.request({ url: '/article/maxSort', method: 'get', params });
}
export function Export(params) {
    jumpExport('/article/export', params);
}
```

---

## 9.12 第十一步：Web 前端 model.ts

```ts
// web/src/views/article/model.ts
import { cloneDeep } from 'lodash-es';
import { FormSchema } from '@/components/Form';
import { BasicColumn } from '@/components/Table';
import { useDictStore } from '@/store/modules/dict';


const dict = useDictStore();

export class State {
    id = 0;
    title = '';
    content = '';
    categoryId = 0;
    coverImage = '';
    status = 1;
    sort = 0;
    remark = '';
}
export function newState(state: State | null): State {
    return state !== null ? cloneDeep(state) : new State();
}

export const rules = {
    title: [{ required: true, message: '请输入标题', trigger: 'blur' }],
};

export const schemas: FormSchema[] = [
    { field: 'title', component: 'NInput', label: '标题' },
    {
        field: 'status',
        component: 'NSelect',
        label: '状态',
        componentProps: {
            options: dict.getOptions('ArticleStatusOptions'),
        },
    },
];

export const columns: BasicColumn[] = [
    { title: 'ID', key: 'id', width: 80 },
    { title: '标题', key: 'title', width: 200 },
    {
        title: '状态',
        key: 'status',
        width: 100,
        render(row) {
            return dict.getLabel('ArticleStatusOptions', row.status);
        },
    },
    { title: '排序', key: 'sort', width: 80 },
    { title: '创建时间', key: 'createdAt', width: 180 },
];

export function loadOptions() {
    dict.loadOptions(['ArticleStatusOptions']);
}
```

---

## 9.13 第十二步：后端后台添加菜单权限

在 HotGo 后台管理系统中：
1. 权限管理 → 菜单权限 → 添加菜单
2. 填写菜单名称（如"文章管理"）、路由路径
3. 绑定 API 权限：`/article/list`、`/article/edit`、`/article/delete`、`/article/status`
4. 权限管理 → 角色权限 → 为角色分配新菜单

---

## 9.14 完整的文件清单（16 个文件）

| # | 文件 | 操作 |
|---|------|------|
| 1 | `server/manifest/sql/article.sql` | 建表 |
| 2 | `server/internal/model/entity/article.go` | 自动生成 |
| 3 | `server/internal/model/do/article.go` | 自动生成 |
| 4 | `server/internal/dao/internal/article.go` | 自动生成 |
| 5 | `server/internal/dao/article.go` | 自动生成 |
| 6 | `server/internal/consts/article.go` | 手写 |
| 7 | `server/internal/model/input/sysin/article.go` | 手写 |
| 8 | `server/internal/service/sys.go` | 追加 |
| 9 | `server/internal/logic/sys/article.go` | 手写 |
| 10 | `server/internal/controller/admin/sys/article.go` | 手写 |
| 11 | `server/api/admin/article/article.go` | 手写 |
| 12 | `server/internal/router/genrouter/article.go` | 手写 |
| 13 | `web/src/api/article/index.ts` | 手写 |
| 14 | `web/src/views/article/model.ts` | 手写 |
| 15 | `web/src/views/article/index.vue` | 手写 |
| 16 | `web/src/views/article/edit.vue` | 手写 |
