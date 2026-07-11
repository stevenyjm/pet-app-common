# C端API响应字段文档

## 1. 概述

本文档整理了宠物生鲜电商后端（微信小程序）所有C端接口的响应字段，包括字段含义、类型、来源及重复/无用字段分析。

## 2. 统一响应格式

所有API响应遵循统一格式：

```json
{
  "code": 0,
  "message": "success",
  "data": {}
}
```

| code | 含义 |
|------|------|
| 0 | 成功 |
| 40001 | 参数错误 |
| 40101 | 未登录/Token失效 |
| 40301 | 无权访问 |
| 404xx | 资源不存在 |
| 422xx | 业务规则拒绝 |
| 50000 | 系统繁忙 |
| 50001 | 服务器错误 |

---

## 3. 认证模块 (auth)

### 3.1 POST /auth/wechat/login - 微信登录

**响应数据**：

| 字段 | 类型 | 含义 | 来源 |
|------|------|------|------|
| accessToken | string | 访问令牌 | JWT生成 |
| refreshToken | string | 刷新令牌 | 随机生成 |
| expiresIn | number | accessToken过期时间（秒） | 配置值 |
| user | object | 用户信息 | UserProfile |
| user.id | string | 用户ID（UUID格式） | 格式化user_id |
| user.userId | number | 用户ID（原始数字） | user_account.user_id |
| user.profileId | number | 资料ID | user_profile.profile_id |
| user.nickname | string | 昵称 | user_profile.nickname |
| user.avatarUrl | string | 头像URL | user_profile.avatar_url |
| user.gender | string | 性别 | user_profile.gender |
| user.phone | string | 手机号（脱敏） | user_profile.phone |
| user.email | string | 邮箱（脱敏） | user_profile.email |
| user.bio | string | 个人简介 | user_profile.bio |
| user.inviteCode | string | 邀请码 | user_profile.invite_code |
| user.invitedByCode | string | 邀请人邀请码 | 查询inviter_user_id对应的邀请码 |
| isNewUser | boolean | 是否新用户 | 注册逻辑判断 |

### 3.2 POST /auth/token/refresh - 刷新令牌

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| accessToken | string | 新的访问令牌 |
| refreshToken | string | 新的刷新令牌 |
| expiresIn | number | 过期时间（秒） |

### 3.3 POST /auth/logout - 退出登录

**响应数据**：`null`

### 3.4 POST /auth/qrcode/create - 创建扫码会话

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| sceneId | string | 会话场景ID |
| expiresAt | string | 过期时间（ISO格式） |
| qrImageUrl | string | 二维码图片URL |
| status | string | 状态（pending/scanned/confirmed/cancelled/expired） |

### 3.5 GET /auth/qrcode/poll - 轮询扫码状态

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| sceneId | string | 会话场景ID |
| status | string | 状态 |
| tokenPayload | object | 扫码成功后的令牌数据（确认后返回） |

---

## 4. 用户模块 (user)

### 4.1 GET /user/profile - 获取用户资料

**响应数据**（同登录响应的user字段）：

| 字段 | 类型 | 含义 |
|------|------|------|
| profileId | number | 资料ID |
| userId | number | 用户ID |
| id | string | 用户ID（UUID格式） |
| nickname | string | 昵称 |
| avatarUrl | string | 头像URL |
| gender | string | 性别 |
| phone | string | 手机号（脱敏） |
| email | string | 邮箱（脱敏） |
| bio | string | 个人简介 |
| inviteCode | string | 邀请码 |
| invitedByCode | string | 邀请人邀请码 |

### 4.2 PUT /user/profile - 更新用户资料

**响应数据**（同上）

### 4.3 GET /user/preferences - 获取用户偏好

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| notification_enabled | boolean | 通知开关 |
| reminder_enabled | boolean | 提醒开关 |
| default_address_id | number | 默认地址ID |
| language | string | 语言偏好 |
| theme | string | 主题偏好 |
| update_time | string | 更新时间（ISO格式，北京时间） |

### 4.4 PUT /user/preferences - 更新用户偏好

**响应数据**（同上）

---

## 5. 宠物模块 (pets)

### 5.1 GET /pets - 获取宠物列表

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| list | array | 宠物列表 |
| list[].pet_id | number | 宠物ID |
| list[].name | string | 宠物名称 |
| list[].avatar_url | string | 头像URL |
| list[].breed | string | 品种 |
| list[].gender | number | 性别（0-未知，1-雄性，2-雌性） |
| list[].birthday | string | 生日（YYYY-MM-DD） |
| list[].latest_mood | string | 最新心情 |
| list[].latest_mood_update_time | string | 最新心情更新时间（ISO格式） |
| list[].latest_weight_kg | number | 最新体重（kg） |
| list[].latest_weight_update_time | string | 最新体重更新时间（ISO格式） |
| list[].latest_stool_health | string | 最新便便健康 |
| list[].latest_stool_update_time | string | 最新便便健康更新时间（ISO格式） |
| list[].latest_physical_state | string | 最近身体状态描述 |
| list[].latest_physical_state_update_time | string | 最近身体状态更新时间（ISO格式） |
| list[].latest_record_date | string | 最新记录日期 |
| list[].status | number | 状态 |
| list[].create_time | string | 创建时间（ISO格式） |
| list[].update_time | string | 更新时间（ISO格式） |
| list[].family_group_id | number | 家庭组ID（如有） |
| total | number | 总数 |

### 5.2 POST /pets - 创建宠物

**响应数据**（详情格式）：

| 字段 | 类型 | 含义 |
|------|------|------|
| pet_id | number | 宠物ID |
| name | string | 宠物名称 |
| avatar_url | string | 头像URL |
| breed | string | 品种 |
| gender | number | 性别（0-未知，1-雄性，2-雌性） |
| birthday | string | 生日 |
| default_owner_user_id | number | 默认主人ID |
| preferences | string | 偏好设置（JSON字符串） |
| medical_history | string | 医疗史（JSON字符串） |
| allergens | string | 过敏原（JSON字符串） |
| latest_mood | string | 最新心情 |
| latest_mood_update_time | string | 最新心情更新时间（ISO格式） |
| latest_weight_kg | number | 最新体重 |
| latest_weight_update_time | string | 最新体重更新时间（ISO格式） |
| latest_stool_health | string | 最新便便健康 |
| latest_stool_update_time | string | 最新便便健康更新时间（ISO格式） |
| latest_physical_state | string | 最近身体状态描述 |
| latest_physical_state_update_time | string | 最近身体状态更新时间（ISO格式） |
| latest_record_date | string | 最新记录日期 |
| status | number | 状态 |
| create_time | string | 创建时间 |
| update_time | string | 更新时间 |
| license | object | 证件信息 |
| license.license_no | string | 证件编号 |
| license.license_type | string | 证件类型 |
| license.valid_from | string | 有效期起始 |
| license.valid_to | string | 有效期结束 |
| license.adoption_date | string | 领养日期 |
| license.license_status | string | 证件状态 |
| license.photo_url | string | 照片URL |
| license.cert_status | string | 认证状态（missing/pending/reviewing/approved/rejected） |
| license.body_photo_urls | array | 身体照片URL列表 |
| license.license_photo_url | string | 证件照片URL |
| license.reject_reason | string | 拒绝原因 |
| license.submitted_at | string | 提交时间 |
| license.reviewed_at | string | 审核时间 |
| companionship | object | 陪伴信息 |
| assessment | object | 评估信息 |
| assessment.morph_type | string | 体型类型 |
| assessment.morph_level | string | 体型等级 |
| assessment.assessed_at | string | 评估时间 |

### 5.3 GET /pets/:pet_id - 获取宠物详情

**响应数据**（同创建响应）

### 5.4 PUT /pets/:pet_id - 更新宠物

**响应数据**（同创建响应）

### 5.5 DELETE /pets/:pet_id - 删除宠物

**响应数据**（同列表项）

### 5.6 GET /pets/:pet_id/daily-records - 获取日记录列表

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| list | array | 日记录列表 |
| list[].daily_record_id | number | 记录ID |
| list[].record_date | string | 记录日期（YYYY-MM-DD） |
| list[].daily_mood | string | 心情 |
| list[].daily_weight_kg | number | 体重（kg） |
| list[].daily_stool_health | string | 便便健康 |
| total | number | 总数 |

### 5.7 GET /pets/:pet_id/daily-records/:record_date - 获取日记录详情

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| daily_record_id | number | 记录ID |
| record_date | string | 记录日期 |
| daily_mood | string | 心情 |
| daily_weight_kg | number | 体重 |
| daily_stool_health | string | 便便健康 |
| meal_sessions | array | 用餐会话列表 |
| meal_sessions[].session_id | number | 会话ID |
| meal_sessions[].period | string | 时段（morning/noon/evening） |
| meal_sessions[].meal_time | string | 用餐时间 |
| meal_sessions[].session_time | string | 会话时间（HH:mm） |
| meal_sessions[].post_meal_status | string | 餐后状态 |
| meal_sessions[].nutrition_reference | string | 营养参考 |
| meal_sessions[].remark | string | 备注 |
| meal_sessions[].sort_order | number | 排序 |
| meal_sessions[].items | array | 餐食项 |
| meal_sessions[].items[].item_id | number | 项ID |
| meal_sessions[].items[].food_name | string | 食物名称 |
| meal_sessions[].items[].amount | string | 分量 |
| meal_sessions[].items[].nutrition_ref | string | 营养参考ID |
| meal_sessions[].items[].nutrition_json | object | 营养数据 |
| meal_sessions[].items[].nutrition_source | string | 营养来源 |
| exercise_sessions | array | 运动会话列表 |
| exercise_sessions[].session_id | number | 会话ID |
| exercise_sessions[].location | string | 运动地点 |
| exercise_sessions[].start_time | string | 开始时间（HH:mm） |
| exercise_sessions[].duration_minutes | number | 时长（分钟） |
| exercise_sessions[].assessment | string | 运动评价 |
| exercise_summary | object | 运动汇总 |

---

## 6. 商品模块 (product)

### 6.1 GET /categories - 获取分类列表

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| list | array | 分类列表 |
| list[].id | string | 分类ID（slug格式） |
| list[].name | string | 分类名称 |
| list[].icon | string | 图标emoji |
| list[].sort_order | number | 排序 |
| total | number | 总数 |

### 6.2 GET /products - 获取商品列表

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| list | array | 商品列表 |
| list[].id | number | 商品ID |
| list[].product_id | number | 商品ID（重复） |
| list[].category_id | string | 分类ID（slug格式） |
| list[].name | string | 商品名称 |
| list[].title | string | 商品名称（重复） |
| list[].price | number | 最低价格 |
| list[].min_price | number | 最低价格（重复） |
| list[].original_price | number | 原价 |
| list[].desc | string | 简介 |
| list[].description | string | 简介（重复） |
| list[].cover_url | string | 封面图URL |
| list[].emoji | string | 图标emoji |
| list[].cover_emoji | string | 图标emoji（重复） |
| list[].badge | string | 徽章文本 |
| list[].badge_text | string | 徽章文本（重复） |
| list[].img_theme | string | 图片主题颜色 |
| list[].cold_chain | boolean | 是否冷链 |
| total | number | 总数 |
| page | number | 当前页码 |
| page_size | number | 每页数量 |

### 6.3 GET /products/:product_id - 获取商品详情

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| id | number | 商品ID |
| product_id | number | 商品ID（重复） |
| category_id | string | 分类ID |
| name | string | 商品名称 |
| title | string | 商品名称（重复） |
| price | number | 最低价格 |
| min_price | number | 最低价格（重复） |
| original_price | number | 原价 |
| desc | string | 简介 |
| description | string | 简介（重复） |
| cover_url | string | 封面图URL |
| emoji | string | 图标emoji |
| cover_emoji | string | 图标emoji（重复） |
| badge | string | 徽章文本 |
| badge_text | string | 徽章文本（重复） |
| img_theme | string | 图片主题颜色 |
| cold_chain | boolean | 是否冷链 |
| sub_title | string | 副标题 |
| sold_count | number | 销量 |
| discount_label | string | 折扣标签 |
| tags | array | 标签列表 |
| gallery | array | 图片画廊 |
| gallery[].url | string | 图片URL |
| gallery[].caption | string | 图片说明 |
| gallery[].theme | string | 主题颜色 |
| gallery[].emoji | string | 图标 |
| skus | array | SKU列表 |
| skus[].id | number | SKU ID |
| skus[].sku_id | number | SKU ID（重复） |
| skus[].label | string | 规格标签 |
| skus[].name | string | 规格名称（重复） |
| skus[].price | number | 价格 |
| skus[].original_price | number | 原价 |
| skus[].stock | number | 库存 |
| intro_points | array | 介绍要点 |
| intro_highlight | string | 高亮介绍 |
| detail_blocks | array | 详情块 |
| detail_params | array | 详情参数 |
| detail_html | string | 详情HTML |
| related_entry | object | 相关入口 |

### 6.4 GET /products/:product_id/meal-scan-payload - 获取餐食扫码载荷

**响应数据**（较复杂，包含营养信息）：

| 字段 | 类型 | 含义 |
|------|------|------|
| product_id | number | 商品ID |
| sku_id | number | SKU ID |
| product_name | string | 商品名称 |
| sku_name | string | 规格名称 |
| weight_g | number | 重量（克） |
| nutrition_json | object | 营养数据JSON |
| nutrition_json.schema_version | number | 版本号 |
| nutrition_json.basis | string | 营养基准（as_feed/dry_matter） |
| nutrition_json.basic | object | 基础营养成分 |
| nutrition_json.basic.crude_protein | number | 粗蛋白 |
| nutrition_json.basic.crude_fat | number | 粗脂肪 |
| nutrition_json.basic.crude_fiber | number | 粗纤维 |
| nutrition_json.basic.crude_ash | number | 粗灰分 |
| nutrition_json.basic.moisture | number | 水分 |
| nutrition_json.updated_at | string | 更新时间 |

### 6.5 GET /skus/:sku_id - 获取SKU详情

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| sku | object | 当前SKU |
| sku.sku_id | number | SKU ID |
| sku.product_id | number | 商品ID |
| sku.name | string | 商品名称 |
| sku.spec | string | 规格 |
| sku.sku_name | string | 规格名称（重复） |
| sku.price | number | 价格 |
| sku.unit_price | number | 单价（重复） |
| sku.original_price | number | 原价 |
| sku.quantity | number | 数量 |
| sku.qty | number | 数量（重复） |
| sku.cold_chain | boolean | 是否冷链 |
| sku.emoji | string | 图标 |
| sku.thumb_theme | string | 缩略图主题 |
| sku.cover_url | string | 封面URL |
| product | object | 商品信息（同商品列表项） |
| skus | array | 同款所有SKU列表 |
| invalid | boolean | 是否失效 |

---

## 7. 购物车模块 (cart)

### 7.1 GET /cart - 获取购物车

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| list | array | 购物车列表 |
| list[].id | number | 购物车项ID |
| list[].cart_id | number | 购物车项ID（重复） |
| list[].sku_id | number | SKU ID |
| list[].product_id | number | 商品ID |
| list[].name | string | 商品名称 |
| list[].spec | string | 规格 |
| list[].sku_name | string | 规格名称（重复） |
| list[].price | number | 价格 |
| list[].unit_price | number | 单价（重复） |
| list[].original_price | number | 原价 |
| list[].quantity | number | 数量 |
| list[].qty | number | 数量（重复） |
| list[].cold_chain | boolean | 是否冷链 |
| list[].emoji | string | 图标 |
| list[].thumb_theme | string | 缩略图主题 |
| list[].cover_url | string | 封面URL |
| list[].checked | number | 是否勾选（0/1） |
| list[].selected | boolean | 是否勾选（重复） |
| list[].invalid | boolean | 是否失效 |
| list[].is_invalid | boolean | 是否失效（重复） |
| total | number | 总数 |

### 7.2 POST /cart - 添加购物车

**响应数据**（同列表项）

### 7.3 PUT /cart/:cart_id - 更新购物车

**响应数据**（同列表项）

### 7.4 DELETE /cart/:cart_id - 删除购物车

**响应数据**：`null`

---

## 8. 订单模块 (orders)

### 8.1 POST /orders - 创建订单

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| order_id | string | 订单号 |
| id | string | 订单号（重复） |
| pay_amount | number | 支付金额 |
| expire_at | string | 过期时间（YYYY-MM-DD HH:mm:ss） |
| status | string | 状态（pending） |

### 8.2 GET /orders - 获取订单列表

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| list | array | 订单列表 |
| list[].id | string | 订单号 |
| list[].order_id | string | 订单号（重复） |
| list[].status | string | 状态（pending/ship/receive/done/cancel/aftersale） |
| list[].cold_chain | boolean | 是否冷链 |
| list[].created_at | string | 创建时间（YYYY-MM-DD HH:mm:ss） |
| list[].total | number | 支付金额 |
| list[].pay_amount | number | 支付金额（重复） |
| list[].reviewed | boolean | 是否已评价 |
| list[].has_review | boolean | 是否已评价（重复） |
| list[].items | array | 订单商品列表 |
| list[].items[].order_item_id | number | 订单项ID |
| list[].items[].product_id | number | 商品ID |
| list[].items[].sku_id | number | SKU ID |
| list[].items[].name | string | 商品名称 |
| list[].items[].spec | string | 规格 |
| list[].items[].price | number | 单价 |
| list[].items[].qty | number | 数量 |
| list[].items[].emoji | string | 图标 |
| list[].items[].thumb_theme | string | 缩略图主题 |
| list[].actions | array | 可执行操作 |
| total | number | 总数 |
| page | number | 当前页码 |
| page_size | number | 每页数量 |

### 8.3 GET /orders/:order_id - 获取订单详情

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| id | string | 订单号 |
| order_id | string | 订单号（重复） |
| status | string | 状态 |
| cold_chain | boolean | 是否冷链 |
| created_at | string | 创建时间 |
| expire_at | string | 过期时间 |
| shipped_at | string | 发货时间 |
| delivery_method | string | 配送方式 |
| buyer_message | string | 买家留言 |
| address | object | 收货地址 |
| address.name | string | 收货人 |
| address.phone | string | 手机号（脱敏） |
| address.province | string | 省份 |
| address.city | string | 城市 |
| address.district | string | 区 |
| address.detail | string | 详细地址 |
| items | array | 订单商品列表（同列表） |
| subtotal | number | 商品小计 |
| shipping_fee | number | 运费 |
| coupon_discount | number | 优惠券折扣 |
| member_discount | number | 会员折扣 |
| total | number | 支付金额 |
| total_amount | number | 总金额（重复） |
| pay_amount | number | 支付金额（重复） |
| payment_method | string | 支付方式 |
| paid_at | string | 支付时间 |
| transaction_id | string | 交易ID |
| logistics | object | 物流信息 |
| logistics.company | string | 快递公司 |
| logistics.tracking_no | string | 运单号 |
| logistics.phone | string | 客服电话 |
| logistics.timeline | array | 物流轨迹 |
| reviewed | boolean | 是否已评价 |
| has_review | boolean | 是否已评价（重复） |
| actions | array | 可执行操作 |

### 8.4 POST /orders/:order_id/cancel - 取消订单

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| order_id | string | 订单号 |
| status | string | 状态（cancel） |

### 8.5 POST /orders/:order_id/confirm - 确认收货

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| order_id | string | 订单号 |
| status | string | 状态（done） |

### 8.6 PUT /orders/:order_id/address - 修改地址

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| order_id | string | 订单号 |
| address | object | 收货地址（完整格式，含手机号） |
| updated_at | string | 更新时间 |

---

## 9. 地址模块 (addresses)

### 9.1 GET /addresses - 获取地址列表

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| list | array | 地址列表 |
| list[].id | number | 地址ID |
| list[].address_id | number | 地址ID（重复） |
| list[].receiver_name | string | 收货人 |
| list[].receiver_phone | string | 手机号 |
| list[].province | string | 省份 |
| list[].city | string | 城市 |
| list[].district | string | 区 |
| list[].detail_address | string | 详细地址 |
| list[].is_default | number | 是否默认（0/1） |
| total | number | 总数 |

### 9.2 POST /addresses - 创建地址

**响应数据**（同列表项）

### 9.3 PUT /addresses/:address_id - 更新地址

**响应数据**（同列表项）

### 9.4 DELETE /addresses/:address_id - 删除地址

**响应数据**：`null`

### 9.5 POST /addresses/:address_id/default - 设置默认

**响应数据**（同列表项）

---

## 10. 资讯模块 (articles)

### 10.1 GET /articles - 获取资讯列表

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| list | array | 资讯列表 |
| list[].id | number | 资讯ID |
| list[].type | string | 类型 |
| list[].tag | string | 标签 |
| list[].title | string | 标题 |
| list[].desc | string | 摘要 |
| list[].date | string | 发布日期（YYYY-MM-DD） |
| list[].icon | string | 图标 |
| list[].cover_emoji | string | 封面图标（重复） |
| list[].cover_theme | string | 封面主题 |
| total | number | 总数 |
| page | number | 当前页码 |
| page_size | number | 每页数量 |

### 10.2 GET /articles/:article_id - 获取资讯详情

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| id | number | 资讯ID |
| type | string | 类型 |
| tag | string | 标签 |
| title | string | 标题 |
| desc | string | 摘要 |
| summary | string | 摘要（重复） |
| date | string | 发布日期 |
| published_at | string | 发布时间（ISO格式，北京时间） |
| views | number | 阅读数 |
| source | string | 来源 |
| cover_url | string | 封面图URL |
| cover_theme | string | 封面主题 |
| cover_emoji | string | 封面图标 |
| paragraphs | array | 正文段落 |
| quote | string | 引用 |
| tail | string | 尾部内容 |
| internal_link | string | 内部链接 |
| external_link | string | 外部链接 |
| cta | string | 行动号召 |
| inline_url | string | 内联链接 |
| position | string | 位置 |
| sort_order | number | 排序 |

---

## 11. 支付模块 (payment)

### 11.1 POST /payment/prepay - 预支付

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| appId | string | 小程序AppID |
| timeStamp | string | 时间戳 |
| nonceStr | string | 随机字符串 |
| package | string | 统一下单返回的prepay_id |
| signType | string | 签名类型（MD5） |
| paySign | string | 支付签名 |

### 11.2 GET /payment/orders/:order_id - 查询支付状态

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| order_id | string | 订单号 |
| status | string | 支付状态（pending/success/failed/refunded） |
| paid_at | string | 支付时间 |
| transaction_id | string | 微信交易ID |

---

## 12. 物流模块 (logistics)

### 12.1 GET /logistics/orders/:order_id - 获取物流信息

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| order_id | string | 订单号 |
| express_company | string | 快递公司 |
| express_no | string | 运单号 |
| logistics_trace | array | 物流轨迹 |
| logistics_trace[].time | string | 时间 |
| logistics_trace[].status | string | 状态描述 |
| logistics_trace[].location | string | 地点 |

---

## 13. 售后模块 (after-sales)

### 13.1 POST /after-sales - 创建售后

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| after_sales_id | number | 售后ID |
| order_id | string | 订单号 |
| status | string | 状态（pending/reviewing/approved/rejected） |
| reason | string | 售后原因 |
| refund_amount | number | 退款金额 |
| create_time | string | 创建时间 |

### 13.2 GET /after-sales/orders/:order_id - 获取售后详情

**响应数据**（同创建响应，含审核信息）

---

## 14. 家庭组模块 (family-groups)

### 14.1 GET /family-groups - 获取家庭组列表

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| list | array | 家庭组列表 |
| list[].family_group_id | number | 家庭组ID |
| list[].name | string | 家庭组名称 |
| list[].creator_user_id | number | 创建者ID |
| list[].status | number | 状态 |
| list[].create_time | string | 创建时间 |
| list[].update_time | string | 更新时间 |
| total | number | 总数 |

### 14.2 GET /family-groups/:family_group_id - 获取家庭组详情

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| family_group_id | number | 家庭组ID |
| name | string | 名称 |
| creator_user_id | number | 创建者ID |
| status | number | 状态 |
| create_time | string | 创建时间 |
| update_time | string | 更新时间 |
| dissolved_at | string | 解散时间 |
| members | array | 成员列表 |
| members[].member_id | number | 成员ID |
| members[].user_id | number | 用户ID |
| members[].member_role | string | 角色 |
| members[].status | number | 状态 |
| members[].can_invite | boolean | 可邀请 |
| members[].can_remove_member | boolean | 可移除成员 |
| members[].can_manage_member_permission | boolean | 可管理权限 |
| members[].join_time | string | 加入时间 |
| members[].leave_time | string | 离开时间 |
| pets | array | 宠物列表 |

---

## 15. 消息模块 (messages)

### 15.1 GET /messages - 获取消息列表

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| list | array | 消息列表 |
| list[].message_id | number | 消息ID |
| list[].category | number | 分类（1订单/2系统/3活动/4其他） |
| list[].message_type | string | 消息类型 |
| list[].title | string | 标题 |
| list[].content | string | 内容 |
| list[].is_read | number | 是否已读（0/1） |
| list[].create_time | string | 创建时间 |
| list[].action_path | string | 跳转路径 |
| pagination | object | 分页信息 |
| pagination.page | number | 当前页码 |
| pagination.page_size | number | 每页数量 |
| pagination.total | number | 总数 |
| pagination.has_more | boolean | 是否有更多 |

### 15.2 GET /messages/unread-count - 获取未读数量

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| total | number | 未读总数 |
| by_category | object | 按分类统计 |
| by_category.1 | number | 订单消息未读数 |
| by_category.2 | number | 系统消息未读数 |
| by_category.3 | number | 活动消息未读数 |
| by_category.4 | number | 其他消息未读数 |

### 15.3 GET /messages/:message_id - 获取消息详情

**响应数据**（同列表项，含extra字段）

---

## 16. 评价模块 (reviews)

### 16.1 POST /orders/:order_id/review - 提交评价

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| review_id | number | 评价ID |
| order_id | string | 订单号 |
| rating | number | 评分（1-5） |
| content | string | 评价内容 |
| images | array | 图片URL列表 |
| create_time | string | 创建时间 |

### 16.2 GET /orders/:order_id/review/draft - 获取评价草稿

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| rating | number | 评分 |
| content | string | 评价内容 |
| images | array | 图片列表 |

### 16.3 PUT /orders/:order_id/review/draft - 保存评价草稿

**响应数据**（同获取草稿）

---

## 17. 存储模块 (storage)

### 17.1 POST /storage/upload - 上传文件

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| key | string | 文件存储key |
| url | string | 文件访问URL |
| size | number | 文件大小（字节） |
| mimeType | string | MIME类型 |

---

## 18. 餐食模板模块 (meal-templates)

### 18.1 GET /meal-templates - 获取餐食模板列表

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| list | array | 模板列表 |
| list[].template_id | string | 模板ID（tpl_前缀） |
| list[].template_name | string | 模板名称 |
| list[].session_slot | string | 时段 |
| list[].session_time | string | 时间（HH:mm） |
| list[].post_meal_status | string | 餐后状态 |
| list[].item_count | number | 餐食项数量 |
| list[].updated_at | string | 更新时间 |
| list[].preview_items | array | 预览餐食项（前2项） |
| list[].preview_text | string | 预览文本 |
| list[].items | array | 全部餐食项 |

### 18.2 GET /meal-templates/:template_id - 获取餐食模板详情

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| template_id | string | 模板ID |
| template_name | string | 模板名称 |
| session_slot | string | 时段 |
| session_time | string | 时间 |
| post_meal_status | string | 餐后状态 |
| nutrition_ref | string | 营养参考 |
| items | array | 餐食项列表 |

---

## 19. 运动模板模块 (exercise-templates)

### 19.1 GET /exercise-templates - 获取运动模板列表

**响应数据**：

| 字段 | 类型 | 含义 |
|------|------|------|
| list | array | 模板列表 |
| list[].template_id | string | 模板ID（extpl_前缀） |
| list[].template_name | string | 模板名称 |
| list[].location | string | 运动地点 |
| list[].start_time | string | 开始时间（HH:mm） |
| list[].duration_minutes | number | 时长（分钟） |
| list[].assessment | string | 评价 |
| list[].preview_text | string | 预览文本 |
| list[].updated_at | string | 更新时间 |

### 19.2 GET /exercise-templates/:template_id - 获取运动模板详情

**响应数据**（同列表项，无updated_at）

---

## 20. 重复/无用字段审查

### 20.1 严重重复字段（建议删除一个）

| 模块 | 重复字段对 | 现状 | 建议保留 | 建议删除 |
|------|-----------|------|----------|----------|
| 订单 | `id` / `order_id` | 列表和详情都重复 | `order_id` | `id` |
| 订单 | `total` / `pay_amount` | 列表中重复 | `pay_amount` | `total` |
| 订单 | `reviewed` / `has_review` | 列表和详情都重复 | `reviewed` | `has_review` |
| 订单 | `total_amount` / `pay_amount` | 详情中重复 | `pay_amount` | `total_amount` |
| 购物车 | `id` / `cart_id` | 所有接口重复 | `cart_id` | `id` |
| 购物车 | `checked` / `selected` | 数字0/1 vs 布尔值 | `selected` | `checked` |
| 购物车 | `invalid` / `is_invalid` | 布尔值重复 | `invalid` | `is_invalid` |
| 商品 | `id` / `product_id` | 列表和详情重复 | `product_id` | `id` |
| 商品 | `name` / `title` | 列表和详情重复 | `name` | `title` |
| 商品 | `price` / `min_price` | 列表和详情重复 | `price` | `min_price` |
| 商品 | `desc` / `description` | 列表和详情重复 | `desc` | `description` |
| 商品 | `emoji` / `cover_emoji` | 列表和详情重复 | `emoji` | `cover_emoji` |
| 商品 | `badge` / `badge_text` | 列表和详情重复 | `badge` | `badge_text` |
| SKU | `id` / `sku_id` | 商品详情的SKU列表 | `sku_id` | `id` |
| SKU | `label` / `name` | 商品详情的SKU列表 | `label` | `name` |
| SKU | `price` / `unit_price` | 购物车和SKU详情 | `price` | `unit_price` |
| SKU | `quantity` / `qty` | 购物车和SKU详情 | `quantity` | `qty` |
| SKU | `spec` / `sku_name` | 购物车 | `spec` | `sku_name` |
| 地址 | `id` / `address_id` | 所有接口重复 | `address_id` | `id` |
| 资讯 | `desc` / `summary` | 详情中重复 | `desc` | `summary` |
| 资讯列表 | `icon` / `cover_emoji` | 列表中重复 | `cover_emoji` | `icon` |

### 20.2 冗余字段（建议删除）

| 模块 | 字段 | 原因 | 前端决策 |
|------|------|------|----------|
| 订单列表 | `items` | 列表页通常不需要完整商品列表，增加响应体积 | 列表页下还需显示订单的所有商品，暂时保留 |
| 餐食模板列表 | `items` | 列表页已有preview_items，完整items冗余 | 删除 |
| 用户资料 | `userId` | 已有`id`（UUID格式）和`profileId`，原始数字ID对前端无用 | 暂时保留（前端 mine.vue 页面显示用户ID使用） |
| 宠物详情 | `default_owner_user_id` | 前端不关心内部用户ID | 删除 |
| 宠物详情 | `preferences` / `medical_history` / `allergens` | JSON字符串，前端需解析，建议直接返回解析后的对象 | 目前小程序前端仍需该字符串字段，暂时保留 |

### 20.3 潜在问题字段

| 模块 | 字段 | 问题 | 建议 | 前端决策 |
|------|------|------|------|----------|
| 用户资料 | `phone` / `email` | 已脱敏，如需完整值需额外接口 | 确认前端是否需要完整值 | 保持现状 |
| 订单地址 | `phone` | 列表页脱敏，详情页完整 | 保持现状 | 保持现状 |
| 宠物详情 | `create_time` / `update_time` | ISO格式但无时区标记 | 统一使用北京时间ISO格式 | 统一使用北京时间ISO格式 |
| 日记录 | `record_date` | YYYY-MM-DD字符串 | 前端可直接解析 | 保持现状 |
| 餐食会话 | `meal_time` / `session_time` | meal_time完整时间，session_time仅HH:mm | 统一格式 | 删除 `meal_time` 字段（前端仅使用 session_time，格式为 HH:mm） |

### 20.4 前端冲突决策与修改事项

以下字段精简方案与小程序前端现有实现存在冲突，经决策确认按文档建议执行：

| 冲突项 | 文档建议 | 前端现状 | 决策 | 前端修改事项 |
|--------|----------|----------|------|--------------|
| 购物车勾选状态 | 保留 `selected`（布尔值），删除 `checked`（数字0/1） | 前端 `cart.js` 以 `checked` 为主，页面使用 `item.checked` | **按文档建议执行** | 详见下方修改清单 |
| 唯一标识字段 | 保留 `xxx_id`（`order_id`/`cart_id`/`product_id`/`address_id`/`sku_id`），删除 `id` | 前端 normalize 函数优先取 `id`，`xxx_id` 作为后备 | **按文档建议执行** | 详见下方修改清单 |
| 订单金额字段 | 保留 `pay_amount`，删除 `total`（列表）和 `total_amount`（详情） | 前端 `order.js` 优先取 `total` | **按文档建议执行** | 详见下方修改清单 |
| 数量字段 | 保留 `quantity`，删除 `qty` | 前端普遍使用 `qty`（cart.vue、order.vue、checkout.vue、detail.vue 等） | **按文档建议执行** | 详见下方修改清单 |

#### 前端修改清单（按模块分类）

**1. 购物车模块 - `checked` → `selected`**

修改文件：
- `src/services/cart.js`（第 53 行）：将 `checked: Boolean(raw.checked ?? raw.selected ?? raw.is_checked ?? true)` 改为优先取 `selected`
- `src/services/cart.js`（第 122 行）：将 `checked: true` 改为 `selected: true`
- `src/services/cart.js`（第 144-154 行）：`updateCartItemChecked` 函数参数和请求 payload 需改用 `selected`
- `src/pages/cart/cart.vue`：所有 `item.checked` 改为 `item.selected`（约 15 处）
- `src/subpk-order/services/checkout.js`（第 33 行）：将 `item.checked` 改为 `item.selected`

**2. 唯一标识字段 - `id` → `xxx_id`**

修改文件：
- `src/services/cart.js`（第 45 行）：将 `id: Number(raw.id ?? raw.cart_id ?? 0)` 改为 `id: Number(raw.cart_id ?? raw.id ?? 0)`
- `src/services/product.js`（第 40 行）：将 `id: raw.id ?? raw.product_id` 改为 `id: raw.product_id ?? raw.id`
- `src/services/order.js`（第 48 行）：将 `id: raw.id ?? raw.order_id ?? ''` 改为 `id: raw.order_id ?? raw.id ?? ''`
- `src/services/address.js`：调整 `id` 取值优先级为 `address_id` 优先
- `src/services/product.js`（第 123 行）：SKU 的 `id` 取值优先级调整为 `sku_id` 优先

**3. 订单金额字段 - `total` → `pay_amount`**

修改文件：
- `src/services/order.js`（第 49 行）：将 `total: Number(raw.total ?? raw.total_amount ?? raw.pay_amount ?? 0)` 改为 `total: Number(raw.pay_amount ?? raw.total ?? 0)`
- `src/services/order.js`（详情响应）：移除对 `total_amount` 的 fallback

**4. 数量字段 - `qty` → `quantity`**

修改文件：
- `src/services/cart.js`（第 52 行）：将 `qty: Number(raw.qty ?? raw.quantity ?? 1)` 改为 `qty: Number(raw.quantity ?? raw.qty ?? 1)`
- `src/services/cart.js`（第 94、104、111、163、194 行）：所有 `qty` 变量改用 `quantity`
- `src/services/order.js`（第 31、154 行）：将 `qty` 改为优先取 `quantity`
- `src/pages/cart/cart.vue`：所有 `item.qty` 改为 `item.quantity`（约 8 处）
- `src/pages/order/order.vue`（第 90、225 行）：将 `item.qty` 改为 `item.quantity`
- `src/subpk-order/checkout.vue`（第 86、270 行）：将 `item.qty` 改为 `item.quantity`
- `src/subpk-order/detail.vue`（第 99 行）：将 `item.qty` 改为 `item.quantity`
- `src/services/trade.js`（第 256 行）：将 `item.qty` 改为 `item.quantity`

#### 修改实施说明

以上修改事项需在后端完成字段精简后，由前端同步执行。建议按以下顺序实施：
1. 后端先保留重复字段，前端完成代码修改并测试
2. 前端修改完成后，后端逐步删除冗余字段
3. 前端需注意 fallback 逻辑保留，确保过渡期兼容性

---

## 21. 字段精简建议

### 21.1 优先级 - 高（强烈建议）

删除以下重复字段对中的后者：
- `id` → 使用 `order_id` / `cart_id` / `product_id` / `address_id` / `sku_id`
- `total` → 使用 `pay_amount`
- `has_review` → 使用 `reviewed`
- `total_amount` → 使用 `pay_amount`
- `checked` → 使用 `selected`（统一布尔值）
- `is_invalid` → 使用 `invalid`
- `title` → 使用 `name`
- `min_price` → 使用 `price`
- `description` → 使用 `desc`
- `cover_emoji` → 使用 `emoji`
- `badge_text` → 使用 `badge`
- `name`（SKU）→ 使用 `label`
- `unit_price` → 使用 `price`
- `qty` → 使用 `quantity`
- `sku_name` → 使用 `spec`
- `summary` → 使用 `desc`
- `icon`（资讯）→ 使用 `cover_emoji`

### 21.2 优先级 - 中（建议）

- 订单列表移除 `items` 字段，详情页保留
- 餐食模板列表移除 `items` 字段，仅保留 `preview_items`
- 用户资料移除 `userId` 字段（原始数字ID）
- 宠物详情移除 `default_owner_user_id` 字段

### 21.3 优先级 - 低（可选）

- 宠物详情的JSON字符串字段转为对象格式
- 统一所有时间字段格式为北京时间ISO格式

---

## 22. 字段命名规范建议

为保持一致性，建议遵循以下命名规范：

| 类别 | 规范 | 示例 |
|------|------|------|
| 唯一标识 | 使用 `xxx_id` 格式 | `order_id`, `product_id`, `sku_id` |
| 布尔值 | 使用 `is_xxx` 或直接使用语义化名称 | `is_default`, `selected`, `invalid` |
| 数量 | 使用 `quantity` | `quantity` |
| 价格 | 使用 `price` | `price`, `original_price` |
| 金额 | 使用 `amount` | `pay_amount`, `total_amount` |
| 时间 | 使用 `xxx_time` 或 `xxx_at` | `create_time`, `paid_at` |
| 文本描述 | 使用语义化名称 | `name`, `desc`, `content` |

---

## 23. 总结

本项目C端API响应存在较多重复字段，主要集中在：
1. 唯一标识字段（`id` / `xxx_id`）
2. 名称字段（`name` / `title` / `label`）
3. 布尔值字段（`checked` / `selected`，`invalid` / `is_invalid`）
4. 价格/金额字段（`price` / `unit_price`，`total` / `pay_amount`）

建议按优先级逐步清理，先处理高优先级的严重重复字段，可显著减少响应体积并提升前端解析效率。

---

## 24. 修改记录（2026-07-01）

### 24.1 已完成的字段精简

以下字段已从响应中移除（前端已同步确认）：

**商品模块（productHelpers.js）**

| 文件 | 函数 | 删除字段 | 保留字段 |
|------|------|----------|----------|
| productHelpers.js | formatProductListItem | `id`, `title`, `min_price`, `description`, `cover_emoji`, `badge_text` | `product_id`, `name`, `price`, `desc`, `emoji`, `badge` |
| productHelpers.js | formatProductDetail > SKU | `id`, `name` | `sku_id`, `label` |
| productHelpers.js | formatSkuLine | `sku_name`, `unit_price`, `qty` | `spec`, `price`, `quantity` |

**购物车模块（cartController.js）**

| 文件 | 函数 | 删除字段 | 保留字段 |
|------|------|----------|----------|
| cartController.js | formatCartItem | `id`, `checked`, `is_invalid` | `cart_id`, `selected`, `invalid` |

**订单模块（orderController.js）**

| 文件 | 函数 | 删除字段 | 保留字段 |
|------|------|----------|----------|
| orderController.js | buildOrderSummary | `id`, `total`, `total_amount`, `has_review` | `order_id`, `pay_amount`, `reviewed` |
| orderController.js | listOrders > 列表项 | `id`, `total`, `has_review` | `order_id`, `pay_amount`, `reviewed` |
| orderController.js | createOrder | `id` | `order_id` |

**地址模块（addressController.js）**

| 文件 | 函数 | 删除字段 | 保留字段 |
|------|------|----------|----------|
| addressController.js | formatAddress | `id`, `is_default`（数字0/1） | `address_id`, `is_default`（布尔值） |

**资讯模块（articleController.js）**

| 文件 | 函数 | 删除字段 | 保留字段 |
|------|------|----------|----------|
| articleController.js | formatArticleRow | `summary` | `desc` |
| articleController.js | formatArticleSummary | `icon` | `cover_emoji` |

### 24.2 修改实施说明

1. **后端修改时间**：2026-07-01
2. **前端同步要求**：前端需按「20.4 前端冲突决策与修改事项」中的修改清单完成对应调整
3. **兼容性处理**：前端需保留 fallback 逻辑（如 `raw.xxx_id ?? raw.id`），确保过渡期兼容性
4. **测试验证**：修改完成后需运行 `npm run verify:auth`、`npm run verify:storage` 等验证脚本确保接口正常