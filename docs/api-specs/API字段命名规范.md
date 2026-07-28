# API 字段命名规范

> **版本**：v1.0  
> **日期**：2026-07-28  
> **关联规范**：`.trae/rules/naming-conventions.md`（三端）  
> **跨项目影响**：pet-app-backend、pet-app-admin-web、pet-app（小程序）

---

## 一、现状说明

### 1.1 当前实践

当前三端 API 响应字段大量使用 snake_case 命名（如 `page_size`、`created_at`、`is_favorited`），与命名规范要求的 camelCase 不一致。

### 1.2 历史原因

- 后端数据库字段使用 snake_case，API 响应直接透传数据库字段
- 三端开发者已形成 snake_case 开发习惯
- 早期规范未明确 API 层字段命名要求

---

## 二、整改策略

### 2.1 后端改造（已完成）

后端通过 `camelizeResponse` 中间件实现自动转换，支持 `use_camel_case` 查询参数控制：

| 参数值 | 响应格式 | 适用场景 |
|--------|----------|----------|
| `use_camel_case=true` | camelCase（如 `pageSize`、`createdAt`） | 新接口或迁移完成的接口 |
| `use_camel_case=false` 或不传 | snake_case（如 `page_size`、`created_at`） | 向后兼容旧前端 |

**新增文件**：
- `utils/snakeToCamel.js`：递归转换 snake_case → camelCase 的工具函数
- `middleware/camelizeResponse.js`：全局响应中间件

### 2.2 前端改造（小程序 · 已完成）

小程序前端通过 HTTP 拦截器实现自动转换，无需修改业务代码：

| 阶段 | 转换方向 | 实现方式 |
|------|----------|----------|
| 请求阶段 | camelCase → snake_case | `deepToSnakeCase()` 自动转换 |
| 响应阶段 | snake_case → camelCase | `deepToCamelCase()` 自动转换 |

**关键文件**：
- `src/utils/caseConvert.js`：case 转换工具函数
- `src/utils/request.js`：HTTP 请求拦截器

### 2.3 前端改造（管理端 · 已完成）

管理端采用相同的 HTTP 拦截器方案。

---

## 三、前端开发规范

### 3.1 API 层（`src/api/*.js`）

- **参数命名**：统一使用 camelCase（如 `pageSize`、`orderId`）
- **响应处理**：直接透传，Service 层处理转换
- **JSDoc 注释**：使用 camelCase 标注参数和响应字段

```javascript
// ✅ 正确
export function getOrders(params = {}) {
  // params: { status?, page?, pageSize? }
  return get('/orders', params)
}
```

### 3.2 Service 层（`src/services/*.js`）

- **数据转换**：拦截器已自动转换，无需手动处理
- **响应字段**：直接使用 camelCase（如 `res.data.pageSize`）
- **状态映射**：使用 camelCase 字段名

```javascript
// ✅ 正确
export async function fetchMessages(params = {}) {
  const res = await messageApi.getMessages(params)
  return {
    list: res.data?.list ?? [],
    pagination: {
      page: res.data?.pagination?.page ?? 1,
      pageSize: res.data?.pagination?.pageSize ?? 20,
      total: res.data?.pagination?.total ?? 0,
    },
  }
}
```

### 3.3 页面层（`.vue`）

- **参数传递**：统一使用 camelCase
- **数据绑定**：使用 camelCase 字段名
- **URL 参数**：使用 camelCase（如 `?cartIds=1,2,3`）

### 3.4 特殊说明：`use_camel_case` 参数

小程序前端已在 `request.js` 中自动添加 `use_camel_case=true` 参数：

```javascript
export function get(url, data, opts = {}) {
  const queryData = { ...data, useCamelCase: true }
  return request({ url, method: 'GET', data: queryData, ...opts })
}
```

前端开发者**无需手动添加**此参数。

---

## 四、字段映射对照表

### 4.1 常用字段映射

| snake_case（后端数据库） | camelCase（前端使用） | 使用场景 |
|--------------------------|-----------------------|----------|
| `page_size` | `pageSize` | 分页参数/响应 |
| `is_favorited` | `isFavorited` | 收藏状态 |
| `is_read` | `isRead` | 已读状态 |
| `is_hidden` | `isHidden` | 隐藏状态 |
| `is_active` | `isActive` | 启用状态 |
| `created_at` | `createdAt` | 创建时间 |
| `updated_at` | `updatedAt` | 更新时间 |
| `order_status` | `orderStatus` | 订单状态 |
| `total_amount` | `totalAmount` | 订单金额 |
| `pay_amount` / `paid_amount` | `payAmount` | 支付金额 |
| `order_id` | `orderId` | 订单 ID |
| `user_id` | `userId` | 用户 ID |
| `product_id` | `productId` | 商品 ID |
| `sku_id` | `skuId` | SKU ID |
| `address_id` | `addressId` | 地址 ID |
| `contact_phone` | `contactPhone` | 联系电话 |
| `image_urls` | `imageUrls` | 图片 URL 列表 |
| `buyer_message` | `buyerMessage` | 买家留言 |
| `pay_channel` | `payChannel` | 支付渠道 |
| `cart_ids` | `cartIds` | 购物车 ID 列表 |
| `refund_amount` | `refundAmount` | 退款金额 |
| `warehouse_id` | `warehouseId` | 仓库 ID |
| `location_id` | `locationId` | 库位 ID |
| `batch_id` | `batchId` | 批次 ID |
| `supplier_id` | `supplierId` | 供应商 ID |
| `category_id` | `categoryId` | 分类 ID |
| `notification_type` | `notificationType` | 通知类型 |
| `warning_level` | `warningLevel` | 预警级别 |
| `triggered_count` | `triggeredCount` | 触发次数 |
| `processed_by` | `processedBy` | 处理人 |

### 4.2 特殊字段处理

| 字段 | 说明 |
|------|------|
| `id` | 保持不变，无需转换 |
| `type` | 保持不变，无需转换 |
| `name` | 保持不变，无需转换 |
| `code` | 保持不变，无需转换 |

---

## 五、过渡期注意事项

### 5.1 向后兼容

- 后端默认返回 snake_case，确保旧版本前端正常运行
- 前端通过 `use_camel_case=true` 参数主动请求 camelCase 响应

### 5.2 开发环境验证

| 验证项 | 说明 |
|--------|------|
| 不传参数 | 返回 snake_case（兼容旧前端） |
| `?use_camel_case=true` | 返回 camelCase（新前端） |
| `?use_camel_case=false` | 返回 snake_case（强制兼容） |

### 5.3 调试技巧

- 使用浏览器开发者工具查看实际请求参数和响应格式
- 检查 `use_camel_case` 参数是否正确传递
- 验证响应字段是否为 camelCase 格式

---

## 六、三端协同状态

| 项目 | 状态 | 说明 |
|------|------|------|
| **pet-app-backend** | ✅ 已完成 | 提供 `use_camel_case` 参数，支持双模式 |
| **pet-app-admin-web** | ✅ 已完成 | HTTP 拦截器自动转换 |
| **pet-app**（小程序） | ✅ 已完成 | HTTP 拦截器自动转换 + `use_camel_case=true` |

---

## 七、整改完成说明

> **完成日期**：2026-07-28  
> **整改策略**：后端支持双模式 + 前端拦截器自动转换

### 已完成整改项

1. **后端**：实现 `camelizeResponse` 中间件 + `use_camel_case` 参数
2. **小程序前端**：创建 `caseConvert.js` 工具函数
3. **小程序前端**：修改 `request.js` 添加 HTTP 拦截器
4. **小程序前端**：API 层统一使用 camelCase 参数
5. **小程序前端**：Service 层统一使用 camelCase
6. **小程序前端**：页面层统一使用 camelCase
7. **小程序前端**：自动添加 `use_camel_case=true` 参数
8. **管理端前端**：同步实现 HTTP 拦截器方案

### 验证结果

- ✅ 小程序 H5 构建成功
- ✅ 小程序微信构建成功
- ✅ 单元测试全部通过

---

*文档生成时间：2026-07-28*  
*关联规则：`.trae/rules/naming-conventions.md`（三端）*