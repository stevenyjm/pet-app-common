# API 命名规范终态母版

> **生效日期**：2026-08-24（v2.0，基于数据库表重构与数据服务优化需求修订）
> **状态**：终态（已落地三端）
> **母版地位**：此文件为三端 `.trae/rules/naming-conventions.md` 中 API 命名章节的权威来源；修改时先更新此母版，再同步到三端副本。
> **配套文档**：`pet-app-common/docs/requirements/命名规范过渡期收敛-实施计划与回滚方案.md`（Phase 1-5 均已完成）
> **需求来源**：`pet-app-common/docs/requirements/2026-08-24-需求-数据库表重构与数据服务优化.md`

---

## 一、终态转换链路（仅保留 1 层转换）

```
小程序 / 管理端前端代码（camelCase）
  ↓
  请求参数：推荐直接传 snake_case（与接口文档一致）；
           若前端写 camelCase，由请求拦截器/工具函数转 snake_case 后发出
  ↓
后端 API（Controllers / Services / format* 函数）
  所有响应字段严格 snake_case，与数据库字段命名保持一致，
  **绝不做** snake_case → camelCase 的任何转换
  ↓
前端响应拦截器（唯一转换点）
  小程序：src/utils/request.js → deepToCamelCase()
  管理端：src/api/http.ts → toCamelCase()
  ↓
前端代码消费（camelCase）
```

## 二、API 响应字段命名 · 强制规则

| 类别 | 规范 | 示例 |
|------|------|------|
| 所有 API 响应字段 | **snake_case**，与数据库列名保持一致 | `order_code`, `user_code`, `create_time`, `pay_amount`, `avatar_url`, `invite_code`, `page_size`, `has_more` |
| 主键/外键字段 | 业务主键 `{实体}_code`，与数据库一致 | `user_code`, `pet_code`, `order_code`, `sku_code` |
| 业务外键引用 | `{被引用实体}_code`，与数据库一致 | `user_code`（引用用户）、`pet_code`（引用宠物） |
| 时间字段 | `create_time`（创建）/ `last_modified_time`（最后修改）/ 业务时间 `_time` 后缀 | `create_time`, `last_modified_time`, `last_login_time`, `pay_time`, `expire_time` |
| 布尔字段 | `is_` / `has_` / `can_` 前缀 | `is_joint`, `is_new_user`, `is_hidden`, `is_online`, `can_invite` |
| 分页字段 | `page`, `page_size`, `total`, `has_more`, `total_pages` | 一律 snake_case |
| Token 字段 | `access_token`, `refresh_token`, `expires_in`, `token_type` | 禁止 `accessToken` / `expiresIn` |
| 邀请字段 | `invite_code`, `invited_by_code`, `inviter_user_code` | 禁止 `inviteCode` / `invitedByCode` |
| JSON 快照内部字段 | 保留写入时的业务结构（非 API 命名范围） | `address_snapshot` 内部结构无需强制 snake_case |

### 常见错误对照

| ❌ 禁止（camelCase 响应） | ✅ 正确（snake_case 响应） |
|------------------------|--------------------------|
| `accessToken` | `access_token` |
| `refreshToken` | `refresh_token` |
| `expiresIn` | `expires_in` |
| `tokenType` | `token_type` |
| `adminCode` | `admin_code` |
| `lastLoginTime` | `last_login_time` |
| `avatarUrl` | `avatar_url` |
| `isNewUser` | `is_new_user` |
| `createTime` / `lastModifiedTime` | `create_time` / `last_modified_time` |
| `userCode` / `petCode` | `user_code` / `pet_code` |
| `inviteCode` / `invitedByCode` | `invite_code` / `invited_by_code` |
| `isJoint` | `is_joint` |
| `qrLogin` | `qr_login` |

## 三、后端（pet-app-backend）规则

### 3.1 禁止行为

- 禁止在任何 `success(res, { ... })` 返回对象中使用 camelCase 键名
- 禁止在任何 `format*()` 函数（`utils/*Formatters.js`, `utils/*Helpers.js`, `services/*Service.js`）返回值中使用 camelCase 键名
- 禁止在 `res.json({ ... })` 直接响应中使用 camelCase 键名
- **禁止重新注册** `middleware/camelizeResponse.js`（已归档）
- 禁止在 `utils/response.js` 中重新引入 `camelize` 选项

### 3.2 新增接口开发流程

1. 数据库字段 → snake_case（已有规则，不变）；主键使用业务主键 `{entity}_code`，时间字段使用 `create_time`/`last_modified_time`（后端主动推送）
2. Sequelize Model 定义 → 与数据库列名完全一致（snake_case）；`timestamps: false` 统一关闭，在 `beforeCreate`/`beforeUpdate` 钩子中主动写入 `create_time`/`last_modified_time`
3. Controller/Service 拼装响应对象 → **所有键名 snake_case**，与 Model 字段保持一致
4. `format*()` 辅助函数 → 返回对象键名一律 snake_case
5. 调用 `success(res, data)` → 直接传，**不传 camelize 选项**
6. `verify:xxx` 自测脚本 → 断言使用 snake_case 字段名

### 3.3 自检清单

```bash
# 新增代码后运行以下两条，确认无 camelCase 响应键
grep -rn "^\s+(accessToken|refreshToken|expiresIn|tokenType|avatarUrl|isNewUser|createTime|lastModifiedTime|userCode|petCode|orderCode|inviteCode|invitedByCode|isJoint|qrLogin|adminCode|lastLoginTime)\s*:" controllers/ services/ utils/ routes/
# 预期：无匹配项
```

## 四、小程序前端（pet-app）规则

### 4.1 响应字段消费

- 响应拦截器 `deepToCamelCase()` 保留不变，作为唯一的 snake_case → camelCase 转换点
- 页面、组件、Service 层**只使用 camelCase** 字段访问响应数据：
  ```javascript
  // ✅ 正确
  console.log(this.profile.avatarUrl)     // avatar_url → avatarUrl（拦截器转换）
  console.log(this.user.isNewUser)         // is_new_user → isNewUser
  console.log(order.isJoint)               // is_joint → isJoint
  console.log(this.auth.accessToken)       // access_token → accessToken
  console.log(this.user.userCode)          // user_code → userCode
  console.log(this.pet.petCode)            // pet_code → petCode
  console.log(this.order.createTime)      // create_time → createTime
  ```
- 禁止在页面/组件代码中直接访问 snake_case 字段（`data.user_code`、`item.create_time`）。
  若需要兼容旧数据，应放在 `utils/` 或 `services/` 层做兼容处理，并标注为临时兼容。

### 4.2 请求参数发出

- 请求拦截器 `deepToSnakeCase()` 暂保留，避免现有 camelCase 参数名丢失
- 新写的 API 函数、Service 层**推荐直接传 snake_case** 参数，减少转换链路
- 类型定义若使用 camelCase，应由请求拦截器负责转换，不要与接口文档字段名冲突

## 五、管理端前端（pet-app-admin-web）规则

### 5.1 响应字段消费

- 响应拦截器 `toCamelCase()` 保留不变，作为唯一的 snake_case → camelCase 转换点
- 视图、组件、Store 层**只使用 camelCase** 字段访问响应数据
- 类型定义（`src/types/*.ts`、`src/api/*.ts` 接口）：
  - **请求**侧类型（如 `OrderListQuery`）可使用 snake_case（`page_size`），与接口文档一致
  - **响应**侧类型（如 `OrderListItem`）统一使用 **snake_case**，因为 `toCamelCase` 会递归转换所有响应对象，所以前端实际拿到的是 camelCase 对象；类型定义建议用 **camelCase**，与运行时实际结构一致
  - 若响应类型与请求类型共用同一接口，使用 snake_case 并在 Service 层做转换

### 5.2 TypeScript 类型推荐实践

```typescript
// src/types/order.ts
// ✅ 推荐：响应类型使用 camelCase（与拦截器转换后的实际结构一致）
export interface OrderListItem {
  orderCode: string
  userCode: string
  createTime: string
  lastModifiedTime: string
  payAmount: number
}

// ✅ 推荐：请求 Query 类型与后端接口文档保持 snake_case
export interface OrderListQuery {
  page?: number
  page_size?: number
  order_status?: string
  keyword?: string
}
```

## 六、跨项目对齐规则

| 检查项 | 规则 |
|--------|------|
| API 规范文档（pet-app-common/docs/api-specs/） | 请求参数、响应字段全部 snake_case |
| 后端 Model / SQL / Sequelize | 全部 snake_case（已有规则） |
| 后端 API 响应 JSON | 全部 snake_case（本章核心规则） |
| 前端响应拦截器 | 各端 1 个，统一转换为 camelCase，保留不动 |
| 前端页面/组件代码 | 全部 camelCase 访问 |
| 后端自测脚本（verify-*.js） | 断言 snake_case，可兼容写法（先 snake 后 camel） |

## 七、违规修复流程

1. **发现违规**（代码审查、grep 自检）：后端响应对象中出现 camelCase 键名
2. **修复步骤**：
   - 后端：把 camelCase 键名改为 snake_case（如 `avatarUrl` → `avatar_url`）
   - grep 确认无其他引用：在 `controllers/services/utils/routes` 范围内搜索旧键名
   - 重新运行 `npm run verify:auth / verify:admin`，确保脚本通过
3. **确认兼容性**：前端响应拦截器会自动转换，前端代码无需变更
4. **同步记录**：写入对应项目 `docs/yjm_daily/` 当日记录

## 八、版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-08-18 | 终态首版。命名规范过渡期收敛 Phase 1-5 全部完成后发布，固化 1 层转换链路。 |
| v2.0 | 2026-08-24 | 基于数据库表重构与数据服务优化需求修订：主键字段从 `{实体}_id` 改为业务主键 `{实体}_code`；时间字段从 `created_at`/`updated_at` 改为 `create_time`/`last_modified_time`；外键引用从 `{实体}_id` 改为 `{实体}_code`；更新常见错误对照、自检清单、前端消费示例、TypeScript 类型示例。 |
