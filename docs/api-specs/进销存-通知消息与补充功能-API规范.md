# 进销存系统 · 通知消息与补充功能 API 规范

> **版本**：v1.0
> **日期**：2026-07-23
> **关联文档**：[进销存系统-功能完善任务清单-2026-07-22.md](../requirements/进销存系统-功能完善任务清单-2026-07-22.md)
> **后端说明**：[项目开发说明-后端.md](../../pet-app-backend/docs/项目开发说明-后端.md) §5.33 · §5.34

---

## 目录

1. [通知消息中心 API](#1-通知消息中心-api)
2. [订单过期自动释放预占](#2-订单过期自动释放预占)
3. [通知消息清理机制](#3-通知消息清理机制)
4. [权限中间件集成](#4-权限中间件集成)
5. [定时任务脚本](#5-定时任务脚本)

---

## 1. 通知消息中心 API

### 1.1 接口前缀

```
/api/v1/admin/notifications
```

所有接口均需 `adminAuthMiddleware` 认证。

### 1.2 接口列表

| 方法 | 路径 | 说明 | 审计日志 |
|------|------|------|---------|
| GET | `/` | 通知列表（支持多条件筛选） | - |
| POST | `/` | 手动创建通知 | `notification_create` |
| GET | `/unread-count` | 未读数（按类型分组） | - |
| GET | `/statistics` | 统计概览 | - |
| GET | `/:id` | 通知详情 | - |
| PUT | `/:id/read` | 标记单条已读 | - |
| PUT | `/read-all` | 全部已读（可按类型筛选） | - |
| PUT | `/:id/archive` | 归档通知 | `notification_archive` |
| DELETE | `/:id` | 删除通知 | `notification_delete` |

### 1.3 请求与响应示例

#### GET /admin/notifications - 通知列表

**请求参数**（Query）：

| 参数 | 类型 | 说明 |
|------|------|------|
| `notification_type` | string | 筛选类型：`expiry_warning`/`low_stock`/`manual`/`system` |
| `status` | string | 筛选状态：`unread`/`read`/`archived` |
| `priority` | int | 筛选优先级：0/1/2 |
| `target_type` | string | 筛选目标：`all_admin`/`specific_admin` |
| `target_admin_id` | int | 筛选目标管理员ID |
| `page` | int | 页码，默认1 |
| `page_size` | int | 每页数量，默认20 |
| `export_all` | boolean | 是否返回全量数据 |

**响应**：

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "total": 100,
    "page": 1,
    "page_size": 20,
    "list": [
      {
        "notification_id": 1,
        "title": "临期预警：商品A即将过期",
        "content": "商品A（批次B）将于7天后过期，请及时处理。",
        "notification_type": "expiry_warning",
        "priority": 1,
        "status": "unread",
        "target_type": "all_admin",
        "biz_type": "expiry_warning",
        "biz_id": 123,
        "extra_json": {"sku_id": 456, "batch_id": 789},
        "created_by": 1,
        "read_at": null,
        "created_at": "2026-07-23 10:00:00"
      }
    ]
  }
}
```

#### POST /admin/notifications - 创建通知

**请求体**：

```json
{
  "title": "系统通知",
  "content": "系统将于今晚22:00进行维护升级。",
  "notification_type": "system",
  "priority": 2,
  "target_type": "all_admin",
  "target_admin_id": null,
  "biz_type": "system_maintenance",
  "biz_id": null,
  "extra_json": {"maintenance_time": "2026-07-23 22:00:00"}
}
```

**响应**：

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "notification_id": 101,
    "title": "系统通知",
    "status": "unread",
    "created_at": "2026-07-23 15:00:00"
  }
}
```

#### GET /admin/notifications/unread-count - 未读数

**响应**：

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "total": 5,
    "by_type": {
      "expiry_warning": 3,
      "low_stock": 2
    }
  }
}
```

### 1.4 通知类型枚举

| 类型 | 值 | 说明 |
|------|-----|------|
| 临期预警 | `expiry_warning` | 商品临期自动推送 |
| 低库存预警 | `low_stock` | 库存低于安全阈值自动推送 |
| 手动通知 | `manual` | 管理员手动创建 |
| 系统通知 | `system` | 系统自动生成的通知 |

### 1.5 状态流转

```
unread → read → archived → deleted
            ↓                    ↑
            └────────────────────┘
```

### 1.6 错误码

| 错误码 | HTTP | 说明 |
|--------|------|------|
| `NOTIFICATION_NOT_FOUND` | 404 | 通知不存在 |
| `ACCESS_DENIED` | 403 | 无权访问 |
| `UNAUTHORIZED` | 401 | 未登录或Token失效 |

---

## 2. 订单过期自动释放预占

### 2.1 功能说明

定时扫描已过期但仍为待支付状态的订单，自动取消订单并释放预占库存。

### 2.2 服务文件

```
services/orderExpiryCronService.js
```

### 2.3 核心方法

#### processExpiredOrders()

**功能**：扫描过期未支付订单并自动释放库存

**返回值**：

```javascript
{
  scanned: 10,      // 扫描到的过期订单数
  processed: 8,     // 成功处理的订单数
  cancelled_order_ids: [1, 2, 3, ...]  // 取消的订单ID列表
}
```

### 2.4 处理流程

```
1. 查询 expire_at < NOW 且 order_status = pending 的订单
2. 对每个订单启动事务：
   a. 加锁查询订单状态（防止并发处理）
   b. 更新订单状态为 cancel（4）
   c. 释放订单中所有商品的预占库存
3. 返回处理结果
```

### 2.5 事务保障

使用 Sequelize 事务确保订单状态更新和库存释放的原子性。

---

## 3. 通知消息清理机制

### 3.1 功能说明

定期清理历史通知消息，避免数据无限增长。

### 3.2 服务文件

```
services/notificationCleanupCronService.js
```

### 3.3 核心方法

| 方法 | 说明 |
|------|------|
| `processNotificationCleanup()` | 主清理流程（调用以下三个方法） |
| `autoArchiveReadNotifications()` | 已读7天自动归档 |
| `cleanupArchivedNotifications()` | 归档90天自动清理 |
| `cleanupDeletedNotifications()` | 已删除30天物理删除 |

### 3.4 清理策略

| 状态 | 处理动作 | 保留期限 |
|------|---------|---------|
| `read`（已读） | 自动归档为 `archived` | 7天 |
| `archived`（已归档） | 物理删除 | 90天 |
| `deleted`（已删除） | 物理删除 | 30天 |

### 3.5 分批处理

每次清理最多处理100条，避免大量删除影响数据库性能。

---

## 4. 权限中间件集成

### 4.1 功能说明

提供 API 级别权限检查能力，支持基于角色和权限标识的细粒度访问控制。

### 4.2 中间件文件

```
middleware/permissionMiddleware.js
```

### 4.3 核心方法

| 方法 | 说明 | 参数 | 返回 |
|------|------|------|------|
| `requirePermission(permissions)` | 必需所有指定权限（AND） | `string|string[]` - 权限标识 | Express中间件 |
| `requireAnyPermission(permissions)` | 满足任一权限即可（OR） | `string[]` - 权限标识数组 | Express中间件 |
| `requireRole(roles)` | 角色检查 | `string|string[]` - 角色代码 | Express中间件 |
| `getAdminPermissions(adminId)` | 获取管理员权限列表 | `number` - 管理员ID | `Promise<string[]>` |

### 4.4 使用示例

```javascript
const { requirePermission, requireAnyPermission, requireRole } = require('../middleware/permissionMiddleware');
const { INVENTORY_PERMISSIONS } = require('../utils/inventoryPermissions');

// 需要库存查看权限
router.get('/inventory', 
  adminAuthMiddleware, 
  requirePermission(INVENTORY_PERMISSIONS.VIEW), 
  handler);

// 需要库存入库或出库权限（满足任一）
router.post('/inventory/transfer',
  adminAuthMiddleware,
  requireAnyPermission([INVENTORY_PERMISSIONS.INBOUND, INVENTORY_PERMISSIONS.OUTBOUND]),
  handler);

// 仅限仓库管理员角色
router.get('/warehouse/stats',
  adminAuthMiddleware,
  requireRole('warehouse_admin'),
  handler);
```

### 4.5 权限标识定义

见 `utils/inventoryPermissions.js`：

| 权限标识 | 说明 |
|---------|------|
| `inventory:dashboard` | 仪表盘 |
| `inventory:view` | 库存查看 |
| `inventory:inbound` | 入库管理 |
| `inventory:outbound` | 出库管理 |
| `batch:view` | 批次查看 |
| `expiry:view` | 临期预警查看 |
| `expiry:config` | 临期预警配置 |
| `trace:view` | 追溯查看 |
| `transfer:view` | 调拨查看 |
| `check:view` | 盘点查看 |
| `warehouse:view` | 仓库查看 |
| `location:view` | 库位查看 |
| `purchase:view` | 采购查看 |
| `supplier:view` | 供应商查看 |
| `finance:view` | 财务查看 |
| `log:view` | 操作日志查看 |

### 4.6 角色权限矩阵

| 角色 | 权限数量 | 说明 |
|------|---------|------|
| `super_admin` | 16 | 超级管理员（所有权限） |
| `warehouse_admin` | 11 | 仓库管理员 |
| `purchaser` | 8 | 采购专员 |
| `finance` | 6 | 财务专员 |
| `operations` | 5 | 运营专员 |

### 4.7 超级管理员放行

`role_code = 'super_admin'` 的管理员自动拥有所有权限，无需检查。

---

## 5. 定时任务脚本

### 5.1 脚本列表

| 脚本命令 | 文件 | 执行频率 | 说明 |
|---------|------|---------|------|
| `npm run cron:expiry-warning` | `scripts/cron-expiry-warning.js` | 每小时 | 临期预警扫描与通知推送 |
| `npm run cron:low-stock-warning` | `scripts/cron-low-stock-warning.js` | 每6小时 | 低库存预警扫描与通知推送 |
| `npm run cron:order-expiry` | `scripts/cron-order-expiry.js` | 每5分钟 | 订单过期自动释放预占 |
| `npm run cron:notification-cleanup` | `scripts/cron-notification-cleanup.js` | 每日凌晨3点 | 通知消息清理 |

### 5.2 启动方式

```bash
# 开发环境
npm run cron:order-expiry
npm run cron:notification-cleanup

# 生产环境（建议使用 PM2 管理）
pm2 start npm --name "cron-order-expiry" -- run cron:order-expiry
pm2 start npm --name "cron-notification-cleanup" -- run cron:notification-cleanup
```

### 5.3 日志输出

所有定时任务均输出详细日志，包括：
- 执行时间
- 扫描数量
- 处理结果
- 错误信息

---

## 6. 数据库变更

### 6.1 notification 表（Schema v38）

| 字段 | 类型 | 说明 |
|------|------|------|
| `notification_id` | BIGINT PK | 通知ID |
| `title` | VARCHAR(128) | 通知标题 |
| `content` | TEXT | 通知内容 |
| `notification_type` | VARCHAR(32) | 类型 |
| `priority` | TINYINT | 优先级 |
| `status` | VARCHAR(16) | 状态 |
| `target_type` | VARCHAR(16) | 目标类型 |
| `target_admin_id` | BIGINT | 目标管理员ID |
| `biz_type` | VARCHAR(32) | 业务类型 |
| `biz_id` | BIGINT | 业务ID |
| `extra_json` | TEXT | 扩展信息 |
| `created_by` | BIGINT | 创建人 |
| `read_at` | DATETIME | 阅读时间 |
| `idempotency_key` | VARCHAR(128) | 幂等键 |
| `created_at` | DATETIME | 创建时间 |
| `updated_at` | DATETIME | 更新时间 |

### 6.2 索引

| 索引名称 | 字段 | 类型 |
|---------|------|------|
| `idx_notification_type_status` | `notification_type, status` | 普通索引 |
| `idx_notification_target` | `target_type, target_admin_id` | 普通索引 |
| `idx_notification_created` | `created_at` | 普通索引 |
| `idx_notification_idempotency` | `idempotency_key` | 唯一索引 |

---

## 7. 关联文档

| 文档 | 路径 | 说明 |
|------|------|------|
| 功能完善任务清单 | [进销存系统-功能完善任务清单-2026-07-22.md](../requirements/进销存系统-功能完善任务清单-2026-07-22.md) | P3阶段需求 |
| 后端开发说明 | [项目开发说明-后端.md](../../pet-app-backend/docs/项目开发说明-后端.md) | §5.33 · §5.34 |
| 三端错误码对照 | [三端业务错误码与状态码对照.md](../requirements/三端业务错误码与状态码对照.md) | 错误码规范 |
| 权限定义 | [inventoryPermissions.js](../../pet-app-backend/utils/inventoryPermissions.js) | 权限标识与角色矩阵 |
