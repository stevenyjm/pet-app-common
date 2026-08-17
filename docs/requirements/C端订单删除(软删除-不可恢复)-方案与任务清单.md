# 追煮跑 · C 端订单删除（软删除 · 不可恢复）· 方案与任务清单

> **文档类型**：需求确认 / 技术方案 / 三端开发任务拆分（正式版）
> **文档版本**：v1.1
> **编制日期**：2026-08-17
> **产品确认人**：（待产品签字 / 或回链确认贴）
> **适用项目**：追煮跑宠物生鲜电商
> **涉及仓库**：`pet-app-backend`（后端服务）、`pet-app`（C 端微信小程序）、`pet-app-admin-web`（PC Web 商户管理端）、`pet-app-common`（公共规范 & API 规范）
> **前置分析**：[需求分析-订单状态筛选与删除能力-临时.md](./需求分析-订单状态筛选与删除能力-临时.md)（v1.0，2026-06-15）
> **关联备案**：[项目开发说明-小程序前端.md §7.6](./项目开发说明-小程序前端.md#76-交易与用户反馈--回收站与评价修改中期--已备案) · [项目开发说明-后端.md](./项目开发说明-后端.md) · [三端业务错误码与状态码对照.md](./三端业务错误码与状态码对照.md)

---

## 状态图例

| 标记 | 含义 |
|------|------|
| ✅ | 已确认 / 已实现 |
| ❌ | 待实现 |
| 🟡 | 进行中 / 部分完成 |
| P0 | 本期必须交付，阻塞验收 |
| P1 | 建议同期交付 |
| P2 | 二期及以后迭代 |
| 🔗 | 跨端依赖（实现顺序需等待被依赖项完成） |

---

## 1. 文档目的

在前序可行性分析与小程序前端 §7.6 备案基础上，**正式批准并落地** C 端用户「删除订单」能力：

- 仅允许对 **已完成（40 / done）** 与 **已关闭/已取消（50 / cancel）** 两种**终态订单**执行删除；
- 删除后订单在数据库层面**不做物理 DELETE**，采用**逻辑删除（软删除）**：写入 `user_deleted_at`，对当前用户永久不可见；
- **不提供**用户侧「回收站」与「恢复」能力 —— 用户二次确认后即视为最终决定，C 端任何入口均无法找回该订单；
- 管理端与客服侧对全量订单（含用户已删除）**始终可见**，并保留**运营兜底撤销删除**能力（用于账号被盗等异常场景），确保交易留痕、财务对账、售后取证闭环。

本文档覆盖：产品决策结论、现状缺口分析、状态门禁规则、数据库与 API 方案、交互流程、三端（后端 / C 端 / 管理端）阶段任务清单与验收标准，作为产品、开发、测试、验收的正式依据。

> **超范围声明**：
> 1. 本文档**不包含**「用户反馈删除」与「订单评价 24h 单次修改」，二者保留在 §7.6 备案的中期待办，另行立项。
> 2. 本文档**不修改**管理员已有隐藏能力（`PATCH /admin/orders/:id/hide` 与 `is_hidden` 字段），该能力与用户侧删除彼此独立，字段语义不混用。
> 3. 本文档**不实现**用户侧回收站、不实现用户自助恢复；用户删除后唯一可能的还原路径是**联系客服由运营在管理端操作**。

---

## 2. 产品确认结论（2026-08-17 版 · v1.1 修订）

| # | 确认项 | 结论 | 方案落点 |
|---|--------|------|----------|
| P1 | 删除能力是否开放 | **开放**（用户诉求强烈 + 终态无履约风险） | §4 · §5 |
| P2 | 允许删除的订单状态 | **仅 40 已完成**、**仅 50 已关闭/已取消**；**禁止** 10 / 20 / 30 / 60 | §3.2 |
| P3 | 删除语义 | **软删除（逻辑删除）** = 对当前用户永久隐藏，写入 `user_deleted_at` | §4.2 · §4.3 |
| P4 | 数据库删除方式 | **禁止物理 DELETE `order_main`**；禁止级联删除 `order_detail`、`payment_log`、`after_sales`、`order_review` | §4.2 |
| **P5** | **回收站 & 用户恢复** | **不提供**：无回收站 Tab、无恢复 API、无 C 端任何找回入口；用户二次确认后即最终删除 | §4.3 · §5.4 |
| P6 | 管理端可见性 | **完全不受用户删除影响**：列表/详情/导出/SQL 统计仍查全量；新增"用户是否已删除"筛选项（P1）；保留运营兜底「撤销用户删除」能力 | §4.4 · §6 |
| P7 | 删除入口位置 | 订单列表卡片「更多」菜单 + 订单详情页底部「删除订单」按钮 | §5.2 |
| P8 | 二次确认 | **需要**，二次确认弹层文案需**强调不可恢复**，区分 40 与 50 状态 | §5.3 |
| P9 | 已取消独立 Tab | 与本期解耦，沿用 [前置分析 §3](./需求分析-订单状态筛选与删除能力-临时.md#3-订单-tab-筛选项覆盖分析) 已有建议（后续另立） | — |

> **v1.1 修订要点**：原 v1.0 P5 为「提供回收站 + 单条/批量恢复」；本次修订为**不提供任何用户侧恢复能力**，仅保留后端软删除标记与管理端运营兜底撤销。

---

## 3. 现状与需求分析

### 3.1 现状盘点

| 维度 | 当前状态 | 缺口 |
|------|----------|------|
| C 端订单写接口（[routes/order.js](../pet-app-backend/routes/order.js)） | `POST /orders` · `PUT /:order_id/cancel` · `PUT /:order_id/confirm-receipt` · `PUT /:order_id/address` + 评价 2 接口 | ❌ **无** `DELETE /orders/:order_id` |
| C 端前端服务（[pet-app/src/api/order.js](../pet-app/src/api/order.js)） | 封装 cancel、confirm、address、评价 | ❌ **无** deleteOrder |
| C 端订单页 UI | 全部 / 待付款 / 待发货 / 待收货 / 已完成 / 售后中（无已取消 Tab） | ❌ 无「删除订单」按钮 |
| order_main 软删除字段 | `is_hidden` / `hidden_at` / `hidden_by`（管理员端语义，已用于 `PATCH /admin/orders/:id/hide`） | ❌ **缺失用户侧软删除字段** `user_deleted_at` |
| 管理端 | 全量订单可见，管理员 hide/show 已交付 | ❌ 无「用户是否已删除」筛选与展示；无运营撤销用户删除能力 |
| 文档/规范 | 已有可行性分析与备案（见 §1） | ❌ 未立项开发，API 规范未写入 `docs/api-specs/` |

### 3.2 订单状态门禁规则（最终确认）

| 状态码 | 后端语义 | 前端别名 | 前端文案 | 允许删除？ | 若不允许的原因 |
|--------|----------|----------|----------|:----------:|----------------|
| **10** | 待支付 | `pending` | 待付款 | ❌ | 仍在支付回调窗口期；删除会造成「用户无法找到待付款订单完成付款」或「库存无法按预期回滚」 |
| **20** | 待发货 | `ship` | 待发货 | ❌ | 商户履约中；删除会干扰发货流程、物流通知、售后链路，且商家无法通知用户改址/取消等 |
| **30** | 待收货 | `receive` | 待收货 | ❌ | 用户需确认收货；删除将导致"找不到在途订单 → 物流通知失效 → 纠纷" |
| **40** | 已完成 | `done` | 已完成 | ✅ | 交易闭环，不再有状态流转；财务/对账/佣金已落定 |
| **50** | 已关闭 / 已取消 | `cancel` | 已取消 | ✅ | 无资金流转风险（未支付取消、支付超时、售后退款成功等） |
| **60** | 售后中 | `aftersale` | 售后中 | ❌ | 退款/换货/退货链路正在进行；删除主订单会破坏 `after_sales` 关联与客服上下文，导致售后工单丢失 |

> **命名统一**：后端/管理端称 50 为 **已关闭**，小程序 C 端称 **已取消**。本需求内均按「允许删除的终态」处理，**不做字段语义改动**，仅在文案层区分展示。

### 3.3 为什么必须软删除（行业 & 合规原因）

| 约束 | 说明 |
|------|------|
| **财务对账** | `payment_log`、退款、发票、商户结算全部依赖订单主从表主键；物理 DELETE 会破坏对账一致性 |
| **售后窗口** | 已完成订单在售后窗口期（通常 7–15 天）仍可被用户申请售后；软删除保留关联能力 |
| **监管与审计** | 电商交易记录一般要求保留 ≥ 3 年；物理删除无法满足合规 |
| **数据统计口径** | 仪表盘 `today_order_count`、`gmv` 等依赖 `order_main` 聚合；物理删除会导致历史统计回退 |
| **客服证据链** | 管理端客服与 SSE 推送需根据 `order_no` / `order_id` 检索会话与反馈；物理删除会打断证据链 |
| **防盗号保护** | 若账号被盗批量删除订单，软删除可由运营在管理端一键撤销，保护用户权益（用户本人无恢复入口，见 P5） |

### 3.4 与管理员 `is_hidden` 的语义隔离（必须遵守）

| 维度 | 管理员隐藏 `is_hidden` | 用户删除 `user_deleted_at` |
|------|------------------------|---------------------------|
| 操作人 | `admin_user.admin_id` | `user_account.user_id` |
| 存储字段 | `is_hidden` / `hidden_at` / `hidden_by` | `user_deleted_at`（单字段，配合 `user_id`） |
| C 端列表 | 默认 `WHERE is_hidden = false` | 主列表追加 `WHERE user_deleted_at IS NULL`；**无回收站反查** |
| 管理端列表 | 可过滤"是否管理员隐藏" | **不受影响**，全量可见；新增可选"用户是否已删除"筛选 |
| 恢复方式 | `PATCH /admin/orders/:id/show`（管理员侧） | **用户侧无恢复入口**；仅 `PATCH /admin/orders/:id/user-restore`（运营兜底） |
| 冲突 | — | 若某订单 **同时**被管理员隐藏+用户删除，则两侧过滤叠加（C 端两边都过滤，管理员仍可见） |

---

## 4. 技术方案（后端 & 数据库）

### 4.1 方案总览

```
┌───────────────────────────────────────────────────────────────────────┐
│  用户从订单列表/详情页点击「删除订单」                                 │
│  → 前端二次确认（强调不可恢复）→ 调用 DELETE /api/v1/orders/:order_id  │
│  → 后端：                                                              │
│    1. 鉴权（Bearer User JWT）+ 归属校验（order.user_id == ctx.userId） │
│    2. 状态门禁：仅 40 / 50 可删除 → 否则 422 ORDER_CANNOT_DELETE       │
│    3. 幂等：已 user_deleted_at != NULL 直接返回 success + already=true│
│    4. UPDATE order_main SET user_deleted_at = NOW() WHERE …            │
│    5. 写 admin_audit_log（action = order_user_delete，含 order_id）   │
│    6. 返回 { order_id, deleted: true, user_deleted_at: ISO+08:00 }     │
└───────────────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────────────┐
│  【运营兜底 · 非用户路径】管理员在管理端点击「撤销用户删除」           │
│  → PATCH /api/v1/admin/orders/:id/user-restore                        │
│  → 后端：                                                              │
│    1. Admin JWT 鉴权 + 权限校验                                         │
│    2. 若 user_deleted_at IS NULL → 幂等返回 success + already=true   │
│    3. UPDATE SET user_deleted_at = NULL                                │
│    4. 写 admin_audit_log（action = order_user_restore，含 admin_id）  │
│    5. C 端用户下次拉取订单列表时该订单自动重新出现                     │
└───────────────────────────────────────────────────────────────────────┘
```

> **关键差异（v1.1）**：原 v1.0 含用户自助 `PUT /orders/:order_id/restore` 路径，本期移除；恢复能力仅保留在管理端运营侧。

### 4.2 数据库设计（Schema 变更）

#### 4.2.1 `order_main` 表新增字段

沿用 `scripts/schema.sql` 中 `order_main`（行 186 起），追加一个 DATETIME 列即可，**不改动既有 `is_hidden` 三件套**：

```sql
ALTER TABLE `order_main`
  ADD COLUMN `user_deleted_at` DATETIME DEFAULT NULL
  COMMENT 'C端用户逻辑删除时间（NULL=未删除）。与管理员 is_hidden 语义隔离',
  ADD INDEX `idx_user_deleted_time` (`user_id`, `user_deleted_at`, `create_time` DESC);
```

| 字段 | 类型 | 默认 | 说明 |
|------|------|------|------|
| `user_deleted_at` | DATETIME NULL | NULL | NULL = 用户未删除；非 NULL = 用户已删除（=删除时间，东八区） |

**索引设计理由**：
- 主查询（用户查看订单列表）：`WHERE user_id = ? AND is_hidden = 0 AND user_deleted_at IS NULL ORDER BY create_time DESC` → 命中现有 `idx_user_status_time` + 新索引的复合前缀即可满足。
- 管理端筛选：`WHERE user_deleted_at IS [NOT] NULL` 不强制加索引，如后续慢查再加覆盖索引。
- **不再需要**回收站反查索引（已移除回收站功能）。

#### 4.2.2 **禁止**的改动

- 禁止 `DELETE FROM order_main WHERE …`；
- 禁止 `ALTER TABLE order_detail ADD fk_delete_cascade`；
- 禁止物理移除 `order_main` 行导致 `payment_log`、`after_sales`、`order_review`、`service_message.related_order_id`、`user_message.order_id` 等外键/关联失效；
- 禁止复用 `is_hidden` 字段混合「管理员隐藏」与「用户删除」语义（会破坏管理端 show/hide 与 C 端列表的正交性）。

#### 4.2.3 Model 变更（Sequelize）

在 `models/OrderMain.js` 追加字段定义（snake_case，与表一致）：

```javascript
user_deleted_at: {
  type: DataTypes.DATE,
  allowNull: true,
  comment: 'C端用户逻辑删除时间，NULL=未删除',
},
```

### 4.3 后端 API 设计（C 端）

所有接口前缀 `/api/v1/orders`，均需 `Bearer User JWT`（走 `authMiddleware`）。

#### 4.3.1 逻辑删除订单

```
DELETE /orders/:order_id
```

| 项目 | 说明 |
|------|------|
| 鉴权 | 必选；`req.user.userId` |
| 归属校验 | 订单 `user_id` 必须等于登录用户；否则 40301 / 40405 |
| 状态门禁 | `order_status IN (40, 50)`；否则 **42227 ORDER_CANNOT_DELETE**（见 §4.5） |
| 幂等 | `user_deleted_at IS NOT NULL` → 直接返回 `{ deleted: true, already_deleted: true }` |
| 副作用 | 1) `UPDATE order_main SET user_deleted_at = NOW()`；2) 写 `admin_audit_log`（action=`order_user_delete`，含 `order_id` + `order_status_before`）；3) 可选：写 `user_message`「您的订单 {order_no} 已删除」（P2） |
| 响应（code=0） | `{ order_id, out_trade_no, deleted: true, user_deleted_at: ISO+08:00, already_deleted?: true }` |

> **不提供** `PUT /orders/:order_id/restore`、`GET /orders/recycle-bin` 接口；C 端用户无任何找回路径。

#### 4.3.2 现有 `GET /orders` 查询变更

在现有 `WHERE user_id = ? AND is_hidden = false` 的基础上**追加**：

```sql
AND user_deleted_at IS NULL
```

> 注意：该改动**不影响**管理端 `GET /admin/orders`（管理端**不过滤** `user_deleted_at`，见 §6）。

### 4.4 后端 API 设计（管理端增量）

管理端复用现有 `GET /admin/orders`、`PATCH /admin/orders/:id/hide`、`PATCH /admin/orders/:id/show`，**不改动语义**。本期新增能力（P1）：

#### 4.4.1 管理端按"用户是否已删除"筛选

```
GET /admin/orders?user_deleted=1   (1=仅用户已删除; 0=仅用户未删除; 不传=全部)
```

管理端订单详情页**增加展示项**：

| 字段 | 说明 |
|------|------|
| `user_deleted_at` | 若不为空，显示"用户已于 YYYY-MM-DD HH:mm 删除此订单"；旁边放「撤销用户删除（P1）」按钮 |

#### 4.4.2 管理员强制撤销用户删除（P1 · 运营兜底 · 唯一恢复路径）

```
PATCH /admin/orders/:id/user-restore
```

| 项目 | 说明 |
|------|------|
| 鉴权 | `Bearer Admin JWT`；`admin_permission` 新增 `order:user-restore`（P2 若不做权限精细化则默认可用） |
| 适用场景 | 账号被盗批量删除、用户误删求助客服、运营核查发现误删等 |
| 幂等 | `user_deleted_at IS NULL` → 返回 `{ restored: true, already_restored: true }` |
| 副作用 | 1) `UPDATE SET user_deleted_at = NULL`；2) 写 `admin_audit_log`（action=`order_user_restore`，含 `admin_id` + `order_id`） |
| 响应 | `{ order_id, restored: true, user_deleted_at: null, already_restored?: true }` |
| 联动 | C 端用户下次拉取订单列表时该订单自动重新出现（无需用户侧任何操作） |

> 这是本期**唯一的订单恢复路径**，且仅限管理端运营人员使用。

### 4.5 错误码（写入 [三端业务错误码与状态码对照.md](./三端业务错误码与状态码对照.md)）

沿用现有 422xx 段，当前已使用至 42226，本期新增：

| 错误码 | 常量名 | HTTP | 含义 | 触发场景 |
|--------|--------|------|------|----------|
| **42227** | `ORDER_CANNOT_DELETE` | 422 | 当前订单状态不允许删除 | 订单 status ∉ {40, 50} |
| **40405** | `ORDER_NOT_FOUND` | 404 | 订单不存在或不归当前用户 | 归属校验失败或主键不存在 |
| **40301** | `FORBIDDEN` | 403 | 无权限（管理员撤销删除权限校验失败用） | P1 管理员侧 |

> 建议：DELETE 走**幂等成功**（已删除时返回 `already_deleted: true`）而非抛业务错误，降低前端异常分支。

---

## 5. 小程序前端方案（pet-app）

### 5.1 API 封装（[src/api/order.js](../pet-app/src/api/order.js)）

```javascript
/**
 * 逻辑删除订单（仅 40/50 状态可删；删除后不可恢复）
 * @param {string|number} orderId - order_id 或 out_trade_no（沿用后端现有解析）
 */
export function deleteOrder(orderId) {
  return request({ url: `/orders/${orderId}`, method: 'DELETE' })
}
```

> **不封装** `restoreOrder` / `getRecycleBinOrders`：本期 C 端无恢复能力。

### 5.2 服务封装（[src/services/order.js](../pet-app/src/services/order.js)）

新增 `deleteOrderService` 方法，复用现有 normalize 规则，处理 `{ deleted, already_deleted }` 响应字段。不新增 `restoreOrderService` / `fetchRecycleBin`。

### 5.3 删除入口 UI

#### 5.3.1 订单列表卡片（[pages/order/order.vue](../pet-app/src/pages/order/order.vue)）

- 在现有 `buildDefaultActions(status)` 中为 **done(40)** 与 **cancel(50)** 追加「删除」动作入口。
- 推荐交互：卡片**左滑显示操作区**（与微信消息列表一致），按钮文案红色描边「删除」。
- 若左滑交互改动过大，可退而求其次：卡片底部「更多 ─ ─ ┤」菜单展开「删除订单」。

#### 5.3.2 订单详情页

- 在底部操作栏（当前 cancel 状态下为空数组）为 40 / 50 追加红色描边按钮「删除订单」。
- 与列表按钮共用同一个二次确认弹窗组件（可复用 `uni.showModal`）。

#### 5.3.3 二次确认文案（P8 产品确认 · 强调不可恢复）

- **状态 40（已完成）**：`确认删除该订单？删除后将无法恢复，且无法在订单列表中再次查看。如需找回，请联系客服。删除不影响已完成的售后与客服记录。`
- **状态 50（已取消/已关闭）**：`确认删除该订单？删除后将无法恢复，且无法在订单列表中再次查看。如后续有退款处理需查询，请联系客服协助。`
- confirm 按钮颜色：`#ff4d4f`，文字「确认删除」；cancel「取消」。

删除成功：
1. `uni.showToast({ title: '已删除', icon: 'success' })`
2. 订单列表本地移除卡片 → 或触发 `loadOrders` 重新拉取。
3. 全量订单数量 Tab 角标同步刷新。

> **文案要点**：不出现「回收站」「移入回收站」「30 天可恢复」等暗示可恢复的字眼，避免误导用户。

### 5.4 无回收站说明

**本期不实现**以下任何一项（均为 v1.0 已移除范围）：

- ❌ 订单 Tab 追加「回收站」Tab
- ❌ 「我的 → 订单中心 → 订单回收站」入口
- ❌ 已删除订单列表查询页
- ❌ 单条 / 批量「恢复订单」按钮与交互
- ❌ `src/api/order.js` 的 `restoreOrder` / `getRecycleBinOrders` 封装
- ❌ `src/services/order.js` 的 `restoreOrderService` / `fetchRecycleBin` 方法
- ❌ `src/constants/order.js` 的 `recycle` Tab 常量

用户删除后若需找回，**唯一路径**是联系客服 → 客服转运营 → 运营在管理端执行「撤销用户删除」（见 §4.4.2）。

### 5.5 与电商屏蔽机制的兼容（Dock slot `enabled=false`）

根据 [电商交易功能屏蔽清单-v2.0](./电商交易功能屏蔽清单-小程序前端.md)，当前 `mall` 与 `order` slot 可能被设为 `enabled=false`。本需求新增能力**不绕过现有四层屏蔽**：

- 若 `order` slot 未启用 → 「订单 Tab」整体隐藏；删除入口也**不会**以其他路径泄露。
- 若后续 `order` slot 重新启用 → 删除能力**自动生效**，无需额外配置。

### 5.6 前端常量 & Mock 补齐

- `src/constants/order.js`：`ORDER_STATUS_TABS` **不追加**回收站 Tab；仅保留现状（全部 / 待付款 / 待发货 / 待收货 / 已完成 / 售后中）。
- `MOCK_ORDERS` 补齐 1 条 `cancel(50)` 与 1 条 `done(40)`，方便 UI 联调删除入口。

---

## 6. 管理端 Web 方案（pet-app-admin-web）

### 6.1 原则

**管理端永远不替用户做"看不见订单"的决定**。管理端列表、详情、导出、客服会话订单上下文**全部不受** `user_deleted_at` 过滤。

### 6.2 增量改动清单

| 项 | 说明 | 优先级 |
|----|------|--------|
| 列表筛选项 | 新增 `用户删除状态` 筛选：全部 / 仅已删除 / 仅未删除；query 参数 `user_deleted` | P1 |
| 列表行标识 | 已用户删除的订单行，追加灰色 chip「用户已删除」+ tooltip 显示删除时间 | P1 |
| 订单详情 | 基础信息块追加一行：用户删除时间 `user_deleted_at`；下方追加操作按钮 `撤销用户删除`（调用 `PATCH /admin/orders/:id/user-restore`） | P1 |
| 导出 CSV | `exportOrdersCsv` 追加列 `用户删除时间(UTC+8)`，确保运营对账可见 | P1 |
| 客服工作台侧栏 | 订单侧栏摘要若对应用户已删除，**仍正常展示**，追加一行小字提示「用户已删除该订单」（不影响客服回复；客服可据此引导用户联系运营撤销） | P2 |
| 权限 | `order:user-restore` 权限点登记（P2 若不做精细化权限，默认可由任意运营账号使用） | P2 |

> 管理端「撤销用户删除」是本期**唯一**的订单恢复路径（对应 §4.4.2 后端 API）。

---

## 7. 三端阶段任务划分 & 任务清单

> **执行顺序**（与 `multi-project.md §2.1` 一致）：
> Step 1：API 规范文档 → Step 2：后端 Model + Migration + Controller + Route → Step 3：后端自测 → Step 4：C 端前端对接 → Step 5：管理端对齐筛选与撤销 → Step 6：三端联调 → Step 7：验收
>
> 各阶段以"**交付点 ID**"在任务清单中唯一标识，便于晨会/周报对齐进度。

---

### 阶段 0 · 规范与前置（2 个交付点 · 0.5 人日）

| ID | 交付物 | 负责项目 | 说明 | 优先级 | 依赖 | 状态 |
|----|--------|----------|------|--------|------|------|
| **SPEC-01** | 在 `pet-app-common/docs/api-specs/` 新建 `C端订单软删除-API规范.md`，包含 §4.3 C 端 DELETE 接口与 §4.4 管理端 user-restore 接口的请求/响应字段、错误码、示例 | pet-app-common | 必须在写后端代码前完成；对齐 multi-project.md §2.1 Step 1 | P0 | — | ❌ |
| **SPEC-02** | 更新 [三端业务错误码与状态码对照.md](./三端业务错误码与状态码对照.md) 追加 42227 | pet-app-common | 确保三端常量文件同步 | P0 | — | ❌ |

---

### 阶段 1 · 后端（pet-app-backend）（9 个交付点 · 2.5 人日）

#### 1.1 数据库与 Model

| ID | 交付物 | 说明 | 优先级 | 依赖 | 状态 |
|----|--------|------|--------|------|------|
| **BE-OD-01** | 迁移脚本 `scripts/migrate-schema-vXX-order-user-deleted-at.js`：DDL ALTER TABLE 加 `user_deleted_at` + 索引 | 版本号取当前最新 schema 版本 +1；幂等（先 `IF NOT EXISTS` 检查列） | P0 | SPEC-01 | ❌ |
| **BE-OD-02** | 更新 `scripts/schema.sql`：`order_main` 追加 `user_deleted_at` 列和索引说明 | 保持与迁移脚本一致 | P0 | BE-OD-01 | ❌ |
| **BE-OD-03** | 更新 `models/OrderMain.js`：追加 `user_deleted_at` 字段 | 命名 snake_case | P0 | BE-OD-01 | ❌ |
| **BE-OD-04** | 更新 `scripts/verify-db.js`：新增校验项 `订单 user_deleted_at 列+索引存在` | 纳入 `npm run verify:db` | P0 | BE-OD-02 | ❌ |

#### 1.2 服务与控制器

| ID | 交付物 | 说明 | 优先级 | 依赖 | 状态 |
|----|--------|------|--------|------|------|
| **BE-OD-05** | `services/orderService.js` 新增：`softDeleteUserOrder(userId, orderId)`；修改 `listOrders` 的 `where` 追加 `user_deleted_at IS NULL` | 软删除 = UPDATE；含状态门禁、归属校验；与管理员 `hideOrder` 解耦；**不新增** restore / listRecycleBin 方法 | P0 | BE-OD-03 | ❌ |
| **BE-OD-06** | `controllers/orderController.js`：新增 `deleteOrder` Controller + 复用现有 `findUserOrder` 归属校验 | Controller 仅做 req→svc→res 封装；错误码严格走 `codes.ORDER_CANNOT_DELETE=42227` | P0 | BE-OD-05, SPEC-02 | ❌ |
| **BE-OD-07** | `routes/order.js`：挂载 `DELETE /:order_id`（`authMiddleware`） | **不挂载** `/restore` 与 `/recycle-bin` | P0 | BE-OD-06 | ❌ |

#### 1.3 管理端后端（P1）

| ID | 交付物 | 说明 | 优先级 | 依赖 | 状态 |
|----|--------|------|--------|------|------|
| **BE-OD-08** | `adminOrderService` 支持按 `user_deleted` 过滤 + 新增 `userRestoreOrder(orderId, adminId)`；`adminOrderController` + `routes/admin/orderRoutes.js` 挂载 `PATCH /:id/user-restore` | 运营兜底唯一恢复路径 | P1 | BE-OD-05 | ❌ |
| **BE-OD-09** | `exportOrdersCsv` 追加 `user_deleted_at` 列到 CSV 表头/数据 | 保持导出兼容（NULL 输出空字符串） | P1 | BE-OD-05 | ❌ |

#### 1.4 自测

| ID | 交付物 | 说明 | 优先级 | 依赖 | 状态 |
|----|--------|------|--------|------|------|
| **BE-OD-10** | `scripts/verify-order-delete.js`：≥ 12 条断言自测；覆盖 6 种状态的删除门禁、幂等、归属越权、`GET /orders` 过滤生效、管理端 `user_deleted` 筛选、`user-restore` 恢复；`npm run verify:order-delete` 脚本注册到 `package.json` | 必须通过方可进入前端对接阶段 | P0 | BE-OD-07, BE-OD-08 | ❌ |

---

### 阶段 2 · C 端小程序（pet-app）（6 个交付点 · 2 人日）

| ID | 交付物 | 说明 | 优先级 | 依赖 | 状态 |
|----|--------|------|--------|------|------|
| **MP-OD-01** | `src/api/order.js`：新增 `deleteOrder` | 与 SPEC-01 字段严格对齐；**不新增** restore / recycleBin | P0 | SPEC-01, BE-OD-10 | ❌ 🔗 |
| **MP-OD-02** | `src/services/order.js`：封装 `deleteOrderService`、normalize 响应 `{ deleted, already_deleted }` | 与 `fetchOrders` 模式一致 | P0 | MP-OD-01 | ❌ |
| **MP-OD-03** | `src/constants/order.js`：Mock 数据补齐 50/40 各 1 条 | `ORDER_STATUS_TABS` **不追加**回收站 | P0 | MP-OD-02 | ❌ |
| **MP-OD-04** | 订单列表页 `pages/order/order.vue`：done/cancel 状态卡片追加「删除」入口（左滑或菜单）+ 二次确认（强调不可恢复）+ 删除成功 UI 联动 | 角标同步；Dock `order` slot 不启用时整页不可见 | P0 | MP-OD-03 | ❌ |
| **MP-OD-05** | 订单详情页：40/50 底部增加「删除订单」按钮；复用统一二次确认 | 与列表删除逻辑共用同一个 service 方法 | P0 | MP-OD-04 | ❌ |
| **MP-OD-06** | 与 `dock.theme.default.json` 屏蔽机制兼容验证：`order` slot 关闭时，删除入口不可达 | 至少走一遍路由守卫断言 | P0 | MP-OD-04, MP-OD-05 | ❌ |

> **已移除交付点**（v1.0 的 MP-OD-06 回收站 Tab、MP-OD-07 恢复交互、MP-OD-08 屏蔽验证、MP-OD-09 微测清单新增小节，共 4 项）。微测清单合并到阶段 4 QA-01 统一覆盖。

---

### 阶段 3 · 管理端 Web（pet-app-admin-web）（5 个交付点 · 1.5 人日）

| ID | 交付物 | 说明 | 优先级 | 依赖 | 状态 |
|----|--------|------|--------|------|------|
| **AD-OD-01** | `src/api/order.ts`：追加 `userRestoreOrder`、`getOrders` query 支持 `user_deleted` 字段 | 与后端 BE-OD-08 对齐 | P1 | BE-OD-08 | ❌ 🔗 |
| **AD-OD-02** | 订单列表页：`用户删除状态` 筛选器；行内 chip「用户已删除」 + tooltip | 不改变默认全量查询 | P1 | AD-OD-01 | ❌ |
| **AD-OD-03** | 订单详情页：展示 `user_deleted_at` 信息 + 「撤销用户删除」操作按钮（二次确认） | 按钮权限（如 P2 不加则默认全部运营可见） | P1 | AD-OD-02 | ❌ |
| **AD-OD-04** | 订单 CSV 导出 UI 同步：说明「用户删除时间」列已加入；保持导出下载流程不变 | BE-OD-09 已输出列时生效 | P1 | BE-OD-09, AD-OD-01 | ❌ |
| **AD-OD-05** | 客服工作台订单侧栏（如实现）：追加小字提示「用户已删除该订单」 | 不影响会话逻辑；引导用户联系运营撤销 | P2 | FE-908（客服模块） | ❌ |

---

### 阶段 4 · 三端联调 & 验收（2 个交付点 · 1 人日）

| ID | 交付物 | 说明 | 优先级 | 依赖 | 状态 |
|----|--------|------|--------|------|------|
| **QA-01** | 端到端场景用例（至少 8 条）覆盖：删除门禁 6 态、幂等、越权、管理员可见性、管理端撤销删除后 C 端重新可见 | 推荐写入 [小程序体验版-用户测试清单.md](./小程序体验版-用户测试清单.md) + [小程序前端-微信开发工具测试任务清单.md](./小程序前端-微信开发工具测试任务清单.md) | P0 | BE-OD-10, MP-OD-06, AD-OD-03 | ❌ 🔗 |
| **QA-02** | 文档回归：后端 `项目开发说明-后端.md §5.7` 追加 1 条 C 端 DELETE API 与 1 条 Admin PATCH user-restore API；`§4.2.4 order_main` 追加 `user_deleted_at`；小程序 `项目开发说明-小程序前端.md §7.6` 将「已备案」升级为「已交付（不含回收站）」，并登记版本号 | 三端对齐检查清单（multi-project.md §3）完整勾选 | P0 | 各阶段交付完成 | ❌ |

---

## 8. 验收标准（Definition of Done）

### 8.1 后端（BE-OD-01~10 全部满足）

- [ ] `npm run db:sync` 后 `order_main.user_deleted_at` 存在；`npm run verify:db` 通过
- [ ] `npm run verify:order-delete` 全部断言通过（≥ 12 条）
- [ ] 6 种状态删除结果符合 §3.2 门禁表
- [ ] 删除不产生任何 `DELETE` SQL（可通过 `scripts/logs` / 自测 SQL 审计断言 `ROW_COUNT` 无变化）
- [ ] C 端 `GET /orders` 默认不返回 user_deleted 记录
- [ ] 管理端 `GET /admin/orders` **仍**返回 user_deleted 记录（证明用户删除不影响管理端）
- [ ] 管理端 `PATCH /admin/orders/:id/user-restore` 成功后，C 端 `GET /orders` 重新返回该订单

### 8.2 小程序（MP-OD-01~06 全部满足）

- [ ] 10/20/30/60 态订单 UI 无「删除」按钮或点击后错误 Toast 符合 42227
- [ ] 40/50 态订单可删除，二次确认（含"无法恢复"文案）后列表消失
- [ ] 删除后用户在 C 端任何入口**均无法**找回该订单（无回收站、无恢复按钮）
- [ ] `order` slot=false 时，删除入口不可达（路由守卫+Dock 双重保护）
- [ ] 微信开发者工具 & 真机无 Crash、无内存泄漏（连续删除 20 次内存稳态）

### 8.3 管理端（AD-OD-01~05）

- [ ] 订单列表能筛出"用户已删除"订单；详情页可撤销用户删除
- [ ] CSV 导出包含用户删除时间列
- [ ] 撤销删除后，C 端用户侧订单自动重新出现（双向联调验证）

### 8.4 合规 & 可追溯性

- [ ] 每笔删除 / 撤销删除均写 `admin_audit_log`，字段含 `user_id` / `admin_id`、`order_id`、`order_status_before`、`action`
- [ ] `order_detail` / `payment_log` / `after_sales` / `order_review` 行数在删除前后完全一致（物理行数 0 变化断言）

---

## 9. 风险与回滚

| 风险 | 概率 | 影响 | 缓解/回滚方案 |
|------|------|------|---------------|
| 老版本 C 端未升级，调用新增的 DELETE 时前端不存在该按钮 → 无影响 | 低 | 无 | 天然向后兼容（旧前端不会主动调新 API） |
| 加 `user_deleted_at IS NULL` 过滤后，主订单列表**意外过滤掉本不该过滤的历史订单** | 中 | 高 | ① 迁移脚本提供同版本的回滚脚本 `migrate-schema-vXX-rollback-user-deleted-at.js`（DROP INDEX + DROP COLUMN）；② 后端支持 `FEATURE_ORDER_USER_DELETE` 环境变量开关（P0，默认开启，关闭时 `listOrders` 不追加该条件） |
| 用户删除后反悔，找不到找回入口产生客诉 | 中 | 中 | ① 二次确认文案明确提示"如需找回请联系客服"；② 客服工作台订单侧栏 AD-OD-05 展示"用户已删除"提示，引导用户走运营撤销路径；③ 管理 AD-OD-03 撤销入口可快速兜底 |
| 管理端运营误点「撤销用户删除」造成用户困扰 | 低 | 中 | 操作加二次确认；写入 audit_log；后续 P2 加权限控制 |
| 账号被盗批量删除订单 | 低 | 高 | 软删除可由运营批量 `user-restore` 撤销；audit_log 含 `user_id` 可追溯；建议 P2 增加"单用户单日删除上限"风控 |

---

## 10. 修订记录

| 版本 | 日期 | 修订内容 | 修订人 |
|------|------|----------|--------|
| v1.0 | 2026-08-17 | 初版：正式立项文档；覆盖需求、门禁、数据库方案、API 设计、三端任务清单（26 个交付点）、验收标准、风险回滚；包含用户侧回收站与自助恢复 | AI 初版（待产品 & 开发负责人确认） |
| **v1.1** | 2026-08-17 | **移除用户侧回收站与自助恢复能力**：① 标题与文档目的修订；② P5 改为「不提供回收站与恢复」；③ 删除 C 端 `PUT /orders/:id/restore` 与 `GET /orders/recycle-bin` 两条 API；④ 删除前端 restoreOrder / getRecycleBinOrders 封装与回收站页面；⑤ 二次确认文案强调"无法恢复"；⑥ 保留管理端 `PATCH /admin/orders/:id/user-restore` 作为**唯一**恢复路径（运营兜底）；⑦ 任务清单精简至 22 个交付点（阶段 1 由 10→9、阶段 2 由 9→6）；⑧ 更新验收标准与风险项；⑨ 文档重命名为 `C端订单删除(软删除-不可恢复)-方案与任务清单.md`（原 `C端订单删除(软删除+回收站)-方案与任务清单.md`） | AI 修订（待产品 & 开发负责人确认） |
