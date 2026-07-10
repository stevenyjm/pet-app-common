# 商品数据字段分析 - C端与Web管理端对比

> **文档类型**：商品字段分析报告
> **文档版本**：v1.2
> **更新日期**：2026-06-26
> **目的**：分析小程序C端商品数据字段与Web管理端商品编辑能力的差距，为后续扩展编辑功能提供参考

---

## 1. 小程序商城页面 - 商品列表字段

小程序端 `GET /products` 返回的商品列表数据包含以下字段：

| 序号 | 字段名 | 类型 | 说明 | 前端展示用途 |
|------|--------|------|------|-------------|
| 1 | `product_id` | number | 商品唯一标识 | 跳转详情、加购 |
| 2 | `product_name` | string | 商品名称 | 商品标题 |
| 3 | `thumbnail` | string \| null | 缩略图URL | 商品封面图 |
| 4 | `intro` | string \| null | 商品简介 | 简短描述 |
| 5 | `badge` | string \| null | 角标文字 | 如"限时"、"热销" |
| 6 | `img_theme` | string \| null | 主题色 | 商品卡片主题色（green/blue/orange） |
| 7 | `cold_chain` | number | 是否冷链 | 冷链标识展示 |
| 8 | `status` | number | 上架状态 | 控制是否展示/可购买 |
| 9 | `category_id` | number | 分类ID | 分类筛选 |
| 10 | `category_slug` | string \| null | 分类slug | 分类标识 |
| 11 | `sold_count` | number | 销量 | 销量展示 |
| 12 | `original_price` | number \| null | 原价 | 划线价展示 |
| 13 | `skus` | ProductSku[] | SKU列表 | 价格区间、规格选择 |
| 14 | `is_hidden` | boolean | 是否隐藏 | 管理端控制可见性 |

### 1.1 SKU 列表字段（嵌套在商品列表中）

| 序号 | 字段名 | 类型 | 说明 | 前端展示用途 |
|------|--------|------|------|-------------|
| 1 | `sku_id` | number | SKU唯一标识 | 加购选择 |
| 2 | `spec_label` | string | 规格标签 | 规格展示（如"500g装"） |
| 3 | `sku_name` | string \| null | SKU名称 | 规格名称 |
| 4 | `price` | number | 售价 | 价格展示 |
| 5 | `original_price` | number \| null | 原价 | 划线价 |
| 6 | `stock` | number | 库存 | 库存提示 |
| 7 | `status` | number | 在售状态 | 控制是否可选 |
| 8 | `nutrition_json` | NutritionJsonV1 \| null | 营养数据 | 餐食扫码预填 |

---

## 2. 小程序商品详情页面 - 详情字段

小程序端 `GET /products/{product_id}` 返回的商品详情数据包含以下字段：

| 序号 | 字段名 | 类型 | 说明 | 前端展示用途 |
|------|--------|------|------|-------------|
| 1 | `product_id` | number | 商品唯一标识 | 页面标识 |
| 2 | `product_name` | string | 商品名称 | 详情页标题 |
| 3 | `thumbnail` | string \| null | 缩略图URL | 主图展示 |
| 4 | `intro` | string \| null | 商品简介 | 详情页简介 |
| 5 | `detail_desc` | string \| null | 详情描述 | 详情页富文本/HTML内容 |
| 6 | `badge` | string \| null | 角标文字 | 促销标识 |
| 7 | `img_theme` | string \| null | 主题色 | 页面主题色 |
| 8 | `cold_chain` | number | 是否冷链 | 冷链标识 |
| 9 | `status` | number | 上架状态 | 购买按钮状态 |
| 10 | `category_id` | number | 分类ID | 面包屑导航 |
| 11 | `category_slug` | string \| null | 分类slug | 分类标识 |
| 12 | `sold_count` | number | 销量 | 销量展示 |
| 13 | `original_price` | number \| null | 原价 | 划线价 |
| 14 | `extra_json` | string \| null | 扩展JSON | 餐食扫码等扩展配置 |
| 15 | `extra` | Record<string, unknown> \| null | 解析后的扩展数据 | 扩展字段解析 |
| 16 | `skus` | ProductSku[] | SKU列表 | 规格选择器 |
| 17 | `is_hidden` | boolean | 是否隐藏 | 管理端控制 |

### 2.1 `extra_json` 扩展字段（商品详情特有）

根据后端文档，`extra_json` 可包含以下扩展配置：

| 配置项 | 说明 | 对应业务 |
|--------|------|----------|
| `meal_scan` | 餐食扫码配置 | 扫码预填食物信息 |
| `gallery` | 商品图片画廊 | 多图展示 |
| `tags` | 商品标签 | 标签展示 |
| `intro_points` | 卖点列表 | 详情页卖点 |

**`meal_scan` Schema 示例**：
```json
{
  "meal_scan": {
    "default_sku_id": "sku-1-500",
    "skus": {
      "sku-1-500": {
        "food_name": "鸡肉鲜食",
        "weight_g": 500,
        "nutrition_ref": "粗蛋白 65.6g · 粗脂肪 0.8g"
      }
    }
  }
}
```

---

## 3. Web管理端商品编辑页面 - 可编辑字段

当前 Web 管理端 `src/views/product/Edit.vue` 支持编辑的字段：

### 3.1 基础信息

| 序号 | 字段名 | 表单组件 | 编辑状态 |
|------|--------|----------|----------|
| 1 | `category_id` | el-select | ✅ 可编辑 |
| 2 | `product_name` | el-input | ✅ 可编辑 |
| 3 | `intro` | el-input(textarea) | ✅ 可编辑 |
| 4 | `detail_desc` | el-input(textarea) | ✅ 可编辑 |
| 5 | `thumbnail` | ImageUpload | ✅ 可编辑 |
| 6 | `badge` | el-input | ✅ 可编辑 |
| 7 | `img_theme` | el-select | ✅ 可编辑 |
| 8 | `cold_chain` | el-switch | ✅ 可编辑 |
| 9 | `status` | el-radio-group | ✅ 可编辑 |

### 3.2 SKU 管理

| 序号 | 字段名 | 表单组件 | 编辑状态 |
|------|--------|----------|----------|
| 1 | `spec_label` | el-input | ✅ 可编辑 |
| 2 | `price` | el-input-number | ✅ 可编辑 |
| 3 | `original_price` | el-input-number | ✅ 可编辑 |
| 4 | `stock` | el-input-number | ✅ 可编辑 |
| 5 | `sku_image` | ImageUpload | ✅ 可编辑 |
| 6 | `status` | el-radio-group | ✅ 可编辑 |
| 7 | `nutrition_json` | SkuNutritionEditor | ✅ 可编辑 |

---

## 4. 字段对比分析

### 4.1 已有编辑能力（✅ 已覆盖）

| 字段 | C端列表 | C端详情 | Web编辑 | 说明 |
|------|---------|---------|---------|------|
| `product_id` | ✅ | ✅ | ⏭️ | 只读，自动生成 |
| `product_name` | ✅ | ✅ | ✅ | 商品名称 |
| `category_id` | ✅ | ✅ | ✅ | 分类选择 |
| `category_slug` | ✅ | ✅ | ⏭️ | 从分类派生，无需编辑 |
| `thumbnail` | ✅ | ✅ | ✅ | 缩略图上传 |
| `intro` | ✅ | ✅ | ✅ | 简介 |
| `detail_desc` | ❌ | ✅ | ✅ | 详情描述 |
| `badge` | ✅ | ✅ | ✅ | 角标 |
| `img_theme` | ✅ | ✅ | ✅ | 主题色 |
| `cold_chain` | ✅ | ✅ | ✅ | 冷链开关 |
| `status` | ✅ | ✅ | ✅ | 上下架 |
| `is_hidden` | ❌ | ❌ | ⏭️ | 管理端独立控制 |

### 4.2 缺失编辑能力（❌ 待扩展）

> **更新说明**：以下 P0/P1 优先级字段已在 Web 前端实现编辑功能（2026-06-26），详见 §5.1 优先级排序。

| 字段 | C端列表 | C端详情 | Web编辑 | 说明 | 扩展建议 | 实现状态 |
|------|---------|---------|---------|------|----------|----------|
| `slug` | ❌ | ❌ | ✅ | 商品别名/Slug | 添加输入框，用于URL友好标识 | **已实现** |
| `original_price` | ✅ | ✅ | ✅ | 商品原价（非SKU级别） | 添加价格输入框，与SKU原价区分 | **已实现** |
| `sold_count` | ✅ | ✅ | ✅ | 销量 | 可提供手动修正功能（后台调整） | **已实现** |
| `extra_json.gallery` | ❌ | ✅ | ✅ | 商品详情页轮播图 | 开发多图上传组件，支持图片排序、删除 | **已实现** |
| `extra_json.tags` | ❌ | ✅ | ✅ | 商品标签 | 开发标签管理组件（新增/删除/排序） | **已实现** |
| `extra_json.intro_points` | ❌ | ✅ | ✅ | 详情页卖点列表 | 开发卖点条目管理组件 | **已实现** |
| `extra_json.meal_scan` | ❌ | ✅ | ❌ | 餐食扫码配置 | 开发可视化餐食扫码配置编辑器 | 待开发（P2） |
| `extra` | ❌ | ✅ | ❌ | 解析后的扩展数据 | 同 extra_json | - |

#### 4.2.1 轮播图（gallery）编辑方案

**现状**：根据后端文档，商品详情页轮播图 `gallery` 字段暂存于 `product.detail_desc` JSON 中，也可存储于 `product.extra_json` 中。

**技术方案**：

| 维度 | 方案 |
|------|------|
| **数据存储** | 使用 `product.extra_json` 的 `gallery` 字段，类型为 `string[]`（图片URL数组） |
| **存储结构** | `{ "gallery": ["https://.../img1.jpg", "https://.../img2.jpg"] }` |
| **编辑组件** | 开发 `MultiImageUpload` 组件，支持多图上传、拖拽排序、删除单张 |
| **前端预览** | 在商品预览面板中添加轮播图预览区域 |
| **后端兼容** | 后端已支持 `extra_json` 的读写，无需额外API开发 |

**MultiImageUpload 组件设计**：

| 功能 | 说明 |
|------|------|
| 多图上传 | 支持点击或拖拽上传多张图片 |
| 图片预览 | 网格展示已上传图片 |
| 拖拽排序 | 支持拖拽调整图片顺序（影响轮播顺序） |
| 删除单张 | 每张图片右上角显示删除按钮 |
| 图片数量限制 | 建议上限 6-8 张 |
| 图片尺寸建议 | 建议正方形（如 800×800）或 3:4 比例 |

#### 4.2.2 商品标签（tags）编辑方案

| 维度 | 方案 |
|------|------|
| **数据存储** | `product.extra_json.tags`，类型为 `string[]` |
| **存储结构** | `{ "tags": ["热销", "限时", "新品"] }` |
| **编辑组件** | 使用 Element Plus 的 `el-tag` 配合输入框，支持新增标签、点击删除 |
| **标签样式** | 可配置标签颜色（默认主题色） |

#### 4.2.3 卖点列表（intro_points）编辑方案

| 维度 | 方案 |
|------|------|
| **数据存储** | `product.extra_json.intro_points`，类型为 `string[]` |
| **存储结构** | `{ "intro_points": ["天然食材", "冷链配送", "当日新鲜"] }` |
| **编辑组件** | 列表式编辑器，支持新增、编辑、删除、排序 |

#### 4.2.4 餐食扫码（meal_scan）编辑方案

| 维度 | 方案 |
|------|------|
| **数据存储** | `product.extra_json.meal_scan`，结构见 §2.1 |
| **编辑组件** | 专用编辑器，支持配置默认SKU、各SKU的食物名称/重量/营养参考 |
| **关联字段** | 与SKU的 `nutrition_json` 联动，优先使用结构化营养数据 |

#### 4.2.5 商品原价（original_price）编辑方案

| 维度 | 方案 |
|------|------|
| **数据存储** | `product.original_price`，类型为 `number \| null` |
| **编辑组件** | `el-input-number`，与SKU原价区分 |
| **业务规则** | 可为空（表示不显示原价）；需大于0；建议不低于商品最低SKU售价 |

#### 4.2.6 销量修正（sold_count）编辑方案

| 维度 | 方案 |
|------|------|
| **数据存储** | `product.sold_count`，类型为 `number` |
| **编辑组件** | `el-input-number`，仅管理员可见 |
| **业务规则** | 需 ≥ 0；谨慎使用，建议添加操作日志 |

#### 4.2.7 商品别名（slug）编辑方案

| 维度 | 方案 |
|------|------|
| **数据存储** | `product.slug`，类型为 `string \| null` |
| **编辑组件** | `el-input`，自动转为小写、移除特殊字符 |
| **业务规则** | 可选字段；用于生成友好URL；需唯一 |

### 4.3 SKU 字段对比

| SKU字段 | C端列表 | C端详情 | Web编辑 | 说明 | 扩展建议 | 实现状态 |
|---------|---------|---------|---------|------|----------|----------|
| `sku_id` | ✅ | ✅ | ⏭️ | 只读 | - | - |
| `product_id` | ✅ | ✅ | ⏭️ | 关联商品，只读 | - | - |
| `spec_json` | ✅ | ✅ | ❌ | 规格JSON | 开发规格可视化编辑器，支持键值对配置 | 待开发（P3） |
| `spec_label` | ✅ | ✅ | ✅ | 规格标签 | - | - |
| `sku_name` | ✅ | ✅ | ❌ | SKU名称 | 添加 `el-input` 编辑框 | 待开发（P2） |
| `price` | ✅ | ✅ | ✅ | 售价 | - | - |
| `original_price` | ✅ | ✅ | ✅ | 原价 | - | - |
| `stock` | ✅ | ✅ | ✅ | 库存 | - | - |
| `sku_image` | ✅ | ✅ | ✅ | SKU图片 | - | - |
| `status` | ✅ | ✅ | ✅ | 在售状态 | - | - |
| `nutrition_json` | ✅ | ✅ | ✅ | 营养数据 | - | - |

#### 4.3.1 SKU 名称（sku_name）编辑方案

| 维度 | 方案 |
|------|------|
| **数据存储** | `product_sku.sku_name`，类型为 `string \| null` |
| **编辑组件** | 在SKU编辑弹窗中添加 `el-input` |
| **业务规则** | 可选字段；若未填写，前端展示时回退到 `spec_label` |

#### 4.3.2 SKU 规格JSON（spec_json）可视化编辑方案

| 维度 | 方案 |
|------|------|
| **数据存储** | `product_sku.spec_json`，类型为 `string`（JSON字符串） |
| **存储结构** | `{ "label": "500g/袋", "specs": { "weight": "500g", "unit": "袋" } }` |
| **编辑组件** | 开发规格键值对编辑器，支持动态添加/删除规格项 |
| **业务规则** | `label` 字段为必填（即当前的 `spec_label`），`specs` 为可选扩展 |

---

## 5. 扩展开发建议

### 5.1 优先级排序

| 优先级 | 字段 | 开发工作量 | 业务价值 | 说明 |
|--------|------|------------|----------|------|
| P0 | `extra_json.gallery`（轮播图） | 中 | **高** | **用户明确需求**：允许在商品编辑页面编辑C端商品详情页顶部轮播图 |
| P0 | `original_price`（商品级） | 小 | 高 | C端已展示原价，管理端应支持编辑 |
| P0 | `sold_count`（修正） | 小 | 中 | 支持后台修正销量数据 |
| P1 | `slug` | 小 | 中 | SEO友好，URL标识 |
| P1 | `extra_json.tags`（商品标签） | 小 | 中 | 丰富商品展示信息 |
| P1 | `extra_json.intro_points`（卖点） | 小 | 中 | 提升商品转化率 |
| P2 | `extra_json.meal_scan`（餐食扫码） | 大 | 高 | 支持餐食扫码预填功能 |
| P2 | SKU `sku_name` | 小 | 低 | 补充规格名称 |
| P3 | SKU `spec_json` 可视化编辑 | 中 | 中 | 规格结构可视化配置 |

### 5.2 `extra_json` 扩展配置编辑器设计

建议开发一个通用的扩展配置编辑器，支持以下配置项：

| 配置模块 | 字段 | 编辑组件 | 说明 |
|----------|------|----------|------|
| 餐食扫码 | `meal_scan.default_sku_id` | el-select | 默认SKU |
| 餐食扫码 | `meal_scan.skus[*].food_name` | el-input | 食物名称 |
| 餐食扫码 | `meal_scan.skus[*].weight_g` | el-input-number | 重量(g) |
| 餐食扫码 | `meal_scan.skus[*].nutrition_ref` | el-input(textarea) | 营养参考文案 |
| 图片画廊 | `gallery[*]` | ImageUpload(multiple) | 多图上传 |
| 商品标签 | `tags[*]` | el-tag + el-input | 标签列表 |
| 卖点列表 | `intro_points[*]` | el-input(textarea) | 卖点条目 |

### 5.3 类型定义更新建议

在 `src/types/product.ts` 中添加 `extra_json` 的具体类型定义：

```typescript
export interface ProductExtraMealScanSku {
  food_name: string
  weight_g: number
  nutrition_ref: string
}

export interface ProductExtraMealScan {
  default_sku_id: string
  skus: Record<string, ProductExtraMealScanSku>
}

export interface ProductExtra {
  meal_scan?: ProductExtraMealScan
  gallery?: string[]
  tags?: string[]
  intro_points?: string[]
}
```

---

## 7. 修订记录

| 版本 | 日期 | 修改人 | 摘要 |
|------|------|--------|------|
| v1.2 | 2026-06-26 | yjm | **Web前端实现**：新增 `MultiImageUpload` 组件；商品编辑页面新增轮播图(gallery)、商品原价(original_price)、销量修正(sold_count)、商品标签(tags)、卖点列表(intro_points)、商品别名(slug)等编辑功能；商品预览面板支持轮播图预览、标签预览、卖点预览；更新章节4.2和4.3的实现状态 |
| v1.1 | 2026-06-26 | yjm | 细化章节4.2和4.3的解决方案：新增轮播图(gallery)、商品标签(tags)、卖点(intro_points)、餐食扫码(meal_scan)、商品原价、销量修正、slug等7个子方案；SKU名称和规格JSON编辑方案；将轮播图提升至P0优先级 |
| v1.0 | 2026-06-26 | yjm | 初始版本：分析C端商品列表/详情字段与Web管理端编辑能力差距 |