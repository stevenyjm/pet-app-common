# 需求分析（临时文档）

> **文档性质**：产品 / 技术可行性评估与修复方案  
> **创建日期**：2026-06-15  
> **触发来源**：Dock 槽位配置与 Phase3 增强需求  
> **涉及模块**：小程序前端 · Web 管理端 · 后端服务  
> **状态**：待实施

---

## 1. 需求背景

### 1.1 Dock 槽位配置现状

当前 Dock 槽位配置存在以下问题：
- 槽位状态管理不够灵活
- 缺少启用/禁用控制
- 无法支持动态配置

### 1.2 Phase3 增强需求

基于业务发展，需要对 Dock 槽位进行以下增强：
- 支持槽位启用/禁用
- 支持动态配置槽位内容
- 支持优先级排序
- 支持多语言配置

---

## 2. 需求分析

### 2.1 Dock 槽位启用/禁用

| 维度 | 评估 |
|------|------|
| **产品合理性** | ✅ 高。支持动态控制槽位显示，便于运营调整 |
| **技术可行性** | ✅ 高。后端新增 `enabled` 字段，前端根据状态显示/隐藏 |
| **当前状态** | ❌ 未实现 |

### 2.2 动态配置槽位内容

| 维度 | 评估 |
|------|------|
| **产品合理性** | ✅ 高。支持运营人员在管理端动态配置槽位内容 |
| **技术可行性** | ✅ 高。后端提供配置接口，前端读取配置并渲染 |
| **当前状态** | ❌ 未实现 |

### 2.3 优先级排序

| 维度 | 评估 |
|------|------|
| **产品合理性** | ✅ 中。支持槽位排序，优化用户体验 |
| **技术可行性** | ✅ 高。后端新增 `priority` 字段，前端按优先级排序显示 |
| **当前状态** | ❌ 未实现 |

### 2.4 多语言配置

| 维度 | 评估 |
|------|------|
| **产品合理性** | ✅ 中。支持多语言环境下的槽位内容展示 |
| **技术可行性** | ✅ 中。后端支持多语言存储，前端根据语言环境读取 |
| **当前状态** | ❌ 未实现 |

---

## 3. 方案设计

### 3.1 Dock 槽位数据模型

```json
{
  "id": 1,
  "name": "首页 Banner",
  "type": "banner",
  "enabled": true,
  "priority": 1,
  "content": {
    "zh-CN": {
      "title": "夏季清凉特惠",
      "description": "全场满100减20",
      "image_url": "https://example.com/banner.jpg",
      "link_url": "/pages/promo/summer"
    },
    "en-US": {
      "title": "Summer Sale",
      "description": "Save 20% on orders over $100",
      "image_url": "https://example.com/banner-en.jpg",
      "link_url": "/pages/promo/summer"
    }
  },
  "created_at": "2026-06-15T00:00:00Z",
  "updated_at": "2026-06-15T00:00:00Z"
}
```

### 3.2 API 设计

#### 3.2.1 获取 Dock 槽位列表

```http
GET /api/v1/dock-slots
```

**响应**：

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [
      {
        "id": 1,
        "name": "首页 Banner",
        "type": "banner",
        "enabled": true,
        "priority": 1,
        "content": {
          "title": "夏季清凉特惠",
          "description": "全场满100减20",
          "image_url": "https://example.com/banner.jpg",
          "link_url": "/pages/promo/summer"
        }
      }
    ],
    "total": 10
  }
}
```

#### 3.2.2 更新 Dock 槽位

```http
PUT /api/v1/dock-slots/{id}
```

**请求体**：

```json
{
  "name": "首页 Banner",
  "type": "banner",
  "enabled": true,
  "priority": 1,
  "content": {
    "zh-CN": {
      "title": "夏季清凉特惠",
      "description": "全场满100减20",
      "image_url": "https://example.com/banner.jpg",
      "link_url": "/pages/promo/summer"
    }
  }
}
```

**响应**：

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "id": 1,
    "name": "首页 Banner",
    "type": "banner",
    "enabled": true,
    "priority": 1,
    "content": {
      "zh-CN": {
        "title": "夏季清凉特惠",
        "description": "全场满100减20",
        "image_url": "https://example.com/banner.jpg",
        "link_url": "/pages/promo/summer"
      }
    },
    "updated_at": "2026-06-15T00:00:00Z"
  }
}
```

---

## 4. 实施任务清单

### 4.1 后端任务

| ID | 任务 | 说明 |
|----|------|------|
| **BE-DOCK-01** | 新增 Dock 槽位表 | 创建 `dock_slots` 表，包含 `id`、`name`、`type`、`enabled`、`priority`、`content`、`created_at`、`updated_at` 字段 |
| **BE-DOCK-02** | 新增获取列表接口 | `GET /api/v1/dock-slots`，支持分页、筛选、排序 |
| **BE-DOCK-03** | 新增获取单个接口 | `GET /api/v1/dock-slots/{id}` |
| **BE-DOCK-04** | 新增创建接口 | `POST /api/v1/dock-slots` |
| **BE-DOCK-05** | 新增更新接口 | `PUT /api/v1/dock-slots/{id}` |
| **BE-DOCK-06** | 新增删除接口 | `DELETE /api/v1/dock-slots/{id}` |
| **BE-DOCK-07** | 更新文档 | 更新 `docs/项目开发说明-后端.md`，添加 Dock 槽位接口文档 |

### 4.2 小程序前端任务

| ID | 任务 | 说明 |
|----|------|------|
| **FE-DOCK-01** | API 封装 | 在 `src/api/dock.js` 中封装 Dock 槽位相关 API |
| **FE-DOCK-02** | 服务层 | 在 `src/services/dock.js` 中实现业务逻辑 |
| **FE-DOCK-03** | 组件开发 | 开发 Dock 槽位组件，支持根据 `enabled` 状态显示/隐藏 |
| **FE-DOCK-04** | 首页集成 | 在首页集成 Dock 槽位组件 |
| **FE-DOCK-05** | 更新文档 | 更新 `docs/项目开发说明-小程序前端.md` |

### 4.3 Web 管理端任务

| ID | 任务 | 说明 |
|----|------|------|
| **WEB-DOCK-01** | API 封装 | 在 `src/api/dock.ts` 中封装 Dock 槽位相关 API |
| **WEB-DOCK-02** | 列表页开发 | 开发 Dock 槽位管理列表页 |
| **WEB-DOCK-03** | 编辑页开发 | 开发 Dock 槽位编辑页，支持启用/禁用、优先级排序、多语言配置 |
| **WEB-DOCK-04** | 更新文档 | 更新 `docs/项目开发说明-Web前端.md` |

---

## 5. 验收标准

| # | 场景 | 预期 |
|---|------|------|
| AC-DOCK-01 | 管理端创建槽位 | 槽位创建成功，数据库记录新增 |
| AC-DOCK-02 | 管理端禁用槽位 | 槽位状态变为禁用，小程序不再显示 |
| AC-DOCK-03 | 管理端调整优先级 | 小程序按优先级排序显示槽位 |
| AC-DOCK-04 | 管理端配置多语言 | 小程序根据语言环境显示对应内容 |
| AC-DOCK-05 | 小程序加载槽位 | 仅显示 `enabled` 为 `true` 的槽位 |

---

## 6. 关联文件索引

| 文件 | 关联任务 |
|------|----------|
| `pet-app-backend/routes/dock-slots.js` | BE-DOCK-02～06 |
| `pet-app-backend/models/dock-slots.js` | BE-DOCK-01 |
| `pet-app/src/api/dock.js` | FE-DOCK-01 |
| `pet-app/src/services/dock.js` | FE-DOCK-02 |
| `pet-app/src/components/dock-slot.vue` | FE-DOCK-03 |
| `pet-app/src/pages/index/index.vue` | FE-DOCK-04 |
| `pet-app-admin-web/src/api/dock.ts` | WEB-DOCK-01 |
| `pet-app-admin-web/src/pages/dock-slots/list.vue` | WEB-DOCK-02 |
| `pet-app-admin-web/src/pages/dock-slots/edit.vue` | WEB-DOCK-03 |

---

## 7. 修订记录

| 版本 | 日期 | 说明 |
|------|------|------|
| v0.1 | 2026-06-15 | 初稿：基于需求分析与方案设计 |
| v0.2 | 2026-06-15 | 补充实施任务清单与验收标准 |
| v0.3 | 2026-06-15 | 补充关联文件索引 |
| v0.4 | 2026-06-15 | 更新 API 设计与数据模型 |
| v0.5 | 2026-06-15 | 补充多语言配置支持 |
| v0.6 | 2026-06-15 | 最终版本，完善所有内容 |