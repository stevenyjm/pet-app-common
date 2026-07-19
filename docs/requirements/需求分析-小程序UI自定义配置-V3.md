# 小程序 UI 自定义配置需求方案（V3）

## 1. 需求概述

### 1.1 需求背景

基于 V2 功能的实施和商户反馈，需要进一步扩展 UI 配置能力：
1. 新增文本内容配置（版权信息），支持商户自定义品牌文案
2. 优化宠物选项区背景配置，区分日记录收起/展开状态
3. 扩展子页面背景配置，覆盖所有二级页面

### 1.2 配置目标扩展

在原有 V1/V2 配置基础上，新增以下配置项：

| 序号 | 配置项类别 | 说明 | 涉及页面 |
|------|-----------|------|---------|
| 1 | 版权信息文本 | 首页底部版权信息可配置，设置页面底部版本信息下方添加版权信息，两处共用同一段配置内容 | 首页、设置 |
| 2 | 宠物选项区收起状态背景 | 当日记录处于收起状态时，宠物选项展示区使用独立背景图片，与展开状态区分 | 宠物 |
| 3 | 子页面背景图片 | 所有二级页面的背景图片配置，包括资讯详情、商品详情、购物车、设置等 | 全部子页面 |

### 1.3 文档引用

本需求文档是对 [需求分析-小程序UI自定义配置.md](file:///C:/Users/22188/Desktop/宠物生鲜电商开发项目/pet-app-common/docs/requirements/需求分析-小程序UI自定义配置.md) 和 [需求分析-小程序UI自定义配置-V2.md](file:///C:/Users/22188/Desktop/宠物生鲜电商开发项目/pet-app-common/docs/requirements/需求分析-小程序UI自定义配置-V2.md) 的补充和扩展，原文档中已完成的配置项继续有效。

---

## 2. 详细需求说明

### 2.1 版权信息配置

#### 2.1.1 首页底部版权信息

| 配置项 | 说明 | 附图参考 |
|--------|------|---------|
| 版权信息文本 | 需要新增对首页底部版权信息一行内容的配置，支持自定义文本 | 首页底部 |

**配置标识**：`texts.copyright.content`

**配置类型**：文本字符串

**Fallback 策略**：当未配置时，显示默认文本「© 2024 宠物生鲜电商」

#### 2.1.2 设置页面底部版权信息

| 配置项 | 说明 | 附图参考 |
|--------|------|---------|
| 版权信息文本 | 设置页面底部版本信息下方添加版权信息，使用与首页相同的配置项 | 设置页面底部 |

**配置标识**：`texts.copyright.content`（与首页共用）

**说明**：两处版权信息使用同一配置项，确保品牌一致性

---

### 2.2 宠物标签页配置扩展

#### 2.2.1 宠物选项区收起状态背景图片

| 配置项 | 说明 | 附图参考 |
|--------|------|---------|
| 宠物选项区收起状态背景 | 当日记录处于收起状态时，宠物选项展示区使用独立的背景图片，与展开状态区分 | 宠物标签页 |

**配置标识**：`pages.pet.petSelector.collapsedBackground`

**与 V2 兼容性说明**：
- V2 已定义 `pages.pet.petSelector.background`，用于日记录展开状态
- V3 新增 `pages.pet.petSelector.collapsedBackground`，用于日记录收起状态
- 当 `collapsedBackground` 未配置时，回退使用 `background` 配置

**Fallback 策略**：当收起状态背景未配置时，回退使用展开状态背景配置；若展开状态也未配置，则使用默认样式

---

### 2.3 子页面背景配置

#### 2.3.1 子页面背景配置列表

| 配置项 | 说明 | 配置标识 |
|--------|------|---------|
| 资讯详情页面背景 | 资讯文章详情页面的背景图片 | `subPages.articleDetail.background` |
| 商品详情页面背景 | 商品详情页面的背景图片 | `subPages.productDetail.background` |
| 购物车页面背景 | 购物车页面的背景图片 | `subPages.cart.background` |
| 订单详情页面背景 | 订单详情页面的背景图片 | `subPages.orderDetail.background` |
| 下单页面背景 | 提交订单页面的背景图片 | `subPages.checkout.background` |
| 收货地址页面背景 | 收货地址列表页面的背景图片 | `subPages.address.background` |
| 在线客服页面背景 | 在线客服聊天页面的背景图片 | `subPages.onlineService.background` |
| 日常提醒页面背景 | 日常提醒列表页面的背景图片 | `subPages.dailyReminder.background` |
| 提醒编辑页面背景 | 日常提醒编辑页面的背景图片 | `subPages.reminderEdit.background` |
| 宠物档案页面（只读模式）背景 | 宠物档案页面只读模式下的背景图片 | `subPages.petProfile.readonly.background` |
| 宠物档案页面（编辑模式）背景 | 宠物档案页面编辑模式下的背景图片 | `subPages.petProfile.edit.background` |
| 消息中心页面背景 | 消息中心页面的背景图片 | `subPages.message.background` |
| 用户反馈页面背景 | 用户反馈提交页面的背景图片 | `subPages.feedback.background` |
| 我的反馈页面背景 | 我的反馈列表页面的背景图片 | `subPages.myFeedback.background` |
| 关于我们页面背景 | 关于我们页面的背景图片 | `subPages.about.background` |

#### 2.3.2 宠物档案页面特殊说明

| 配置项 | 说明 |
|--------|------|
| 只读模式背景 | 宠物档案页面默认显示状态使用的背景图片 |
| 编辑模式背景 | 宠物档案页面进入编辑状态后使用的背景图片 |

**说明**：宠物档案页面的两种模式使用不同的背景配置，允许商户为不同状态设置差异化视觉效果

#### 2.3.3 子页面背景通用配置标识

所有子页面背景配置统一位于 `subPages` 配置段下，采用以下命名规则：

```
subPages.{pageKey}.background
```

其中 `{pageKey}` 为页面标识，对应关系如下：

| pageKey | 页面名称 | 页面路径 |
|---------|---------|---------|
| articleDetail | 资讯详情 | subpk-news/detail.vue |
| productDetail | 商品详情 | pages/product/detail.vue |
| cart | 购物车 | pages/cart/cart.vue |
| orderDetail | 订单详情 | subpk-order/detail.vue |
| checkout | 下单 | subpk-order/checkout.vue |
| address | 收货地址 | subpk-address/list.vue |
| onlineService | 在线客服 | subpk-service/chat.vue |
| dailyReminder | 日常提醒 | subpk-reminder/list.vue |
| reminderEdit | 提醒编辑 | subpk-reminder/edit.vue |
| petProfile.readonly | 宠物档案（只读） | subpk-pet/info.vue |
| petProfile.edit | 宠物档案（编辑） | subpk-pet/info.vue（编辑模式） |
| message | 消息中心 | pages/message/index.vue |
| feedback | 用户反馈 | subpk-feedback/index.vue |
| myFeedback | 我的反馈 | subpk-feedback/history.vue |
| about | 设置 | subpk-settings/index.vue |

#### 2.3.4 子页面背景 Fallback 策略

| 场景 | Fallback 策略 |
|------|--------------|
| 背景图片未配置 | 回退到所属标签页的页面背景配置 |
| 背景图片加载失败 | 回退到硬编码的默认背景色 `#f5f6f8` |
| 所属标签页背景也未配置 | 回退到硬编码的默认背景色 `#f5f6f8` |

**示例**：购物车页面（`subPages.cart.background`）的 fallback 链：
1. 使用 `subPages.cart.background`
2. 若未配置，回退使用 `pages.mall.background`（商城标签页背景）
3. 若仍未配置，回退使用默认背景色 `#f5f6f8`

---

## 3. API 接口扩展设计

### 3.1 响应结构扩展

在原有 `themeAssets` 配置结构基础上，新增 `texts` 和 `subPages` 配置段：

```json
{
  "themeAssets": {
    "version": 3,
    "pages": {
      "home": {
        "background": "https://..."
      },
      "mall": {
        "background": "https://...",
        "categoryArea": { "background": "https://..." },
        "categoryItem": { "background": "https://..." },
        "productArea": { "background": "https://..." },
        "productCard": { "background": "https://..." }
      },
      "pet": {
        "background": "https://...",
        "petSelector": {
          "background": "https://...",
          "collapsedBackground": "https://..."
        },
        "dailyStatus": { "background": "https://..." },
        "recordCard": {
          "meal": { "background": "https://..." },
          "exercise": { "background": "https://..." }
        }
      },
      "order": {
        "background": "https://...",
        "filterBar": { "background": "https://..." },
        "orderCard": { "background": "https://..." }
      },
      "mine": {
        "background": "https://...",
        "introCard": { "background": "https://..." }
      }
    },
    "subPages": {
      "articleDetail": { "background": "https://..." },
      "productDetail": { "background": "https://..." },
      "cart": { "background": "https://..." },
      "orderDetail": { "background": "https://..." },
      "checkout": { "background": "https://..." },
      "address": { "background": "https://..." },
      "onlineService": { "background": "https://..." },
      "dailyReminder": { "background": "https://..." },
      "reminderEdit": { "background": "https://..." },
      "petProfile": {
        "readonly": { "background": "https://..." },
        "edit": { "background": "https://..." }
      },
      "message": { "background": "https://..." },
      "feedback": { "background": "https://..." },
      "myFeedback": { "background": "https://..." },
      "about": { "background": "https://..." }
    },
    "buttons": {
      "cart": { "icon": "https://..." },
      "message": { "icon": "https://..." },
      "settings": { "icon": "https://..." },
      "scan": { "icon": "https://..." }
    },
    "texts": {
      "copyright": {
        "content": "© 2024 宠物生鲜电商"
      }
    },
    "components": { ... }
  }
}
```

### 3.2 新增配置项数据结构

#### 3.2.1 文本配置 (`texts`)

| 字段 | 类型 | 说明 |
|------|------|------|
| texts.copyright.content | string | 版权信息文本，用于首页和关于我们页面底部 |

#### 3.2.2 宠物配置扩展 (`pages.pet`)

| 字段 | 类型 | 说明 |
|------|------|------|
| petSelector.collapsedBackground | string | 宠物选项区收起状态背景图片 URL |

#### 3.2.3 子页面配置 (`subPages`)

| 字段 | 类型 | 说明 |
|------|------|------|
| articleDetail.background | string | 资讯详情页面背景图片 URL |
| productDetail.background | string | 商品详情页面背景图片 URL |
| cart.background | string | 购物车页面背景图片 URL |
| orderDetail.background | string | 订单详情页面背景图片 URL |
| checkout.background | string | 下单页面背景图片 URL |
| address.background | string | 收货地址页面背景图片 URL |
| onlineService.background | string | 在线客服页面背景图片 URL |
| dailyReminder.background | string | 日常提醒页面背景图片 URL |
| reminderEdit.background | string | 提醒编辑页面背景图片 URL |
| petProfile.readonly.background | string | 宠物档案页面（只读模式）背景图片 URL |
| petProfile.edit.background | string | 宠物档案页面（编辑模式）背景图片 URL |
| message.background | string | 消息中心页面背景图片 URL |
| feedback.background | string | 用户反馈页面背景图片 URL |
| myFeedback.background | string | 我的反馈页面背景图片 URL |
| about.background | string | 关于我们页面背景图片 URL |

---

## 4. 前端实现方案

### 4.1 页面改造方案

#### 4.1.1 首页 (`pages/index/index.vue`)

**改造内容**：
- 底部版权信息文本动态绑定，使用 `texts.copyright.content`

#### 4.1.2 设置页面 (`subpk-settings/index.vue`)

**改造内容**：
- 底部版本信息下方添加版权信息文本，动态绑定 `texts.copyright.content`
- 页面背景动态绑定，使用 `subPages.about.background`
- fallback 回退到我的页面背景配置

#### 4.1.3 宠物 (`pages/pet/pet.vue`)

**改造内容**：
- 根据日记录展开/收起状态，动态切换宠物选项区背景图片
- 收起状态使用 `pages.pet.petSelector.collapsedBackground`
- 展开状态使用 `pages.pet.petSelector.background`

#### 4.1.4 资讯详情页面 (`subpk-news/detail.vue`)

**改造内容**：
- 页面背景动态绑定，使用 `subPages.articleDetail.background`
- fallback 回退到首页背景配置

#### 4.1.5 商品详情页面 (`pages/product/detail.vue`)

**改造内容**：
- 页面背景动态绑定，使用 `subPages.productDetail.background`
- fallback 回退到商城页面背景配置

#### 4.1.6 购物车页面 (`pages/cart/cart.vue`)

**改造内容**：
- 页面背景动态绑定，使用 `subPages.cart.background`
- fallback 回退到商城页面背景配置

#### 4.1.7 订单详情页面 (`subpk-order/detail.vue`)

**改造内容**：
- 页面背景动态绑定，使用 `subPages.orderDetail.background`
- fallback 回退到订单页面背景配置

#### 4.1.8 下单页面 (`subpk-order/checkout.vue`)

**改造内容**：
- 页面背景动态绑定，使用 `subPages.checkout.background`
- fallback 回退到订单页面背景配置

#### 4.1.9 收货地址页面 (`subpk-address/list.vue`)

**改造内容**：
- 页面背景动态绑定，使用 `subPages.address.background`
- fallback 回退到我的页面背景配置

#### 4.1.10 在线客服页面 (`subpk-service/chat.vue`)

**改造内容**：
- 页面背景动态绑定，使用 `subPages.onlineService.background`
- fallback 回退到我的页面背景配置

#### 4.1.11 日常提醒页面 (`subpk-reminder/list.vue`)

**改造内容**：
- 页面背景动态绑定，使用 `subPages.dailyReminder.background`
- fallback 回退到宠物页面背景配置

#### 4.1.12 提醒编辑页面 (`subpk-reminder/edit.vue`)

**改造内容**：
- 页面背景动态绑定，使用 `subPages.reminderEdit.background`
- fallback 回退到宠物页面背景配置

#### 4.1.13 宠物档案页面（只读模式） (`subpk-pet/info.vue`)

**改造内容**：
- 页面背景动态绑定，使用 `subPages.petProfile.readonly.background`
- fallback 回退到宠物页面背景配置

#### 4.1.14 宠物档案页面（编辑模式） (`subpk-pet/info.vue`)

**改造内容**：
- 页面背景动态绑定，使用 `subPages.petProfile.edit.background`
- fallback 回退到宠物页面背景配置

#### 4.1.15 消息中心页面 (`pages/message/index.vue`)

**改造内容**：
- 页面背景动态绑定，使用 `subPages.message.background`
- fallback 回退到我的页面背景配置

#### 4.1.16 用户反馈页面 (`subpk-feedback/index.vue`)

**改造内容**：
- 页面背景动态绑定，使用 `subPages.feedback.background`
- fallback 回退到我的页面背景配置

#### 4.1.17 我的反馈页面 (`subpk-feedback/history.vue`)

**改造内容**：
- 页面背景动态绑定，使用 `subPages.myFeedback.background`
- fallback 回退到我的页面背景配置

### 4.2 配置常量更新

需要在 `src/constants/miniAppConfig.js` 中新增以下常量：

```javascript
export const SUB_PAGE_IDS = {
  ARTICLE_DETAIL: 'articleDetail',
  PRODUCT_DETAIL: 'productDetail',
  CART: 'cart',
  ORDER_DETAIL: 'orderDetail',
  CHECKOUT: 'checkout',
  ADDRESS: 'address',
  ONLINE_SERVICE: 'onlineService',
  DAILY_REMINDER: 'dailyReminder',
  REMINDER_EDIT: 'reminderEdit',
  PET_PROFILE_READONLY: 'petProfile.readonly',
  PET_PROFILE_EDIT: 'petProfile.edit',
  MESSAGE: 'message',
  FEEDBACK: 'feedback',
  MY_FEEDBACK: 'myFeedback',
  ABOUT: 'about',
}

export const TEXT_CONFIG_IDS = {
  COPYRIGHT: 'copyright',
}
```

### 4.3 Fallback 方案

#### 4.3.1 文本配置 Fallback

| 配置项 | Fallback 策略 |
|--------|--------------|
| texts.copyright.content | 回退到默认文本「© 2024 宠物生鲜电商」 |

#### 4.3.2 宠物选项区背景 Fallback

| 配置项 | Fallback 策略 |
|--------|--------------|
| petSelector.collapsedBackground | 回退到 petSelector.background；若仍未配置，使用默认样式 |

#### 4.3.3 子页面背景 Fallback

| 子页面 | 所属标签页 | Fallback 链 |
|--------|-----------|-------------|
| articleDetail | 首页 | subPages.articleDetail → pages.home → 默认 #f5f6f8 |
| productDetail | 商城 | subPages.productDetail → pages.mall → 默认 #f5f6f8 |
| cart | 商城 | subPages.cart → pages.mall → 默认 #f5f6f8 |
| orderDetail | 订单 | subPages.orderDetail → pages.order → 默认 #f5f6f8 |
| checkout | 订单 | subPages.checkout → pages.order → 默认 #f5f6f8 |
| address | 我的 | subPages.address → pages.mine → 默认 #f5f6f8 |
| onlineService | 我的 | subPages.onlineService → pages.mine → 默认 #f5f6f8 |
| dailyReminder | 宠物 | subPages.dailyReminder → pages.pet → 默认 #f5f6f8 |
| reminderEdit | 宠物 | subPages.reminderEdit → pages.pet → 默认 #f5f6f8 |
| petProfile.readonly | 宠物 | subPages.petProfile.readonly → pages.pet → 默认 #f5f6f8 |
| petProfile.edit | 宠物 | subPages.petProfile.edit → pages.pet → 默认 #f5f6f8 |
| message | 我的 | subPages.message → pages.mine → 默认 #f5f6f8 |
| feedback | 我的 | subPages.feedback → pages.mine → 默认 #f5f6f8 |
| myFeedback | 我的 | subPages.myFeedback → pages.mine → 默认 #f5f6f8 |
| about | 我的 | subPages.about → pages.mine → 默认 #f5f6f8 |

---

## 5. Web 管理端实现方案

### 5.1 配置管理界面扩展

#### 5.1.1 新增选项卡

在现有「小程序配置」页面中，新增以下选项卡：

| 选项卡名称 | 说明 |
|-----------|------|
| 文本配置 | 配置版权信息等文本内容 |
| 子页面背景 | 配置所有二级页面的背景图片 |

#### 5.1.2 文本配置选项卡

**配置区域**：

```
┌──────────────────────────────────────────────────────────┐
│                    文本配置                               │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │ 版权信息                                            │  │
│  │                                                    │  │
│  │  标签：[首页 / 设置]                               │  │
│  │  内容：[__________________________]                │  │
│  │                                                    │  │
│  │  说明：配置首页和设置页面底部显示的版权信息文本       │  │
│  │  字数限制：50 字符以内                              │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**预览区功能**：

```
┌──────────────────────────────────────────────────────────┐
│                    预览区                                │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  ┌─────────────────────────────────────────────┐   │  │
│  │  │  ⬜ 手机模拟器 375px × 812px               │   │  │
│  │  │                                           │   │  │
│  │  │        [页面内容区域]                      │   │  │
│  │  │                                           │   │  │
│  │  │        © 2024 宠物生鲜电商                  │   │  │
│  │  │                                           │   │  │
│  │  └─────────────────────────────────────────────┘   │  │
│  │                                                    │  │
│  │  [实时预览：输入内容同步显示在模拟器底部版权区域]      │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**预览区实现细节**：
- 使用 CSS 绘制手机边框（圆角 20px、阴影效果），模拟 iPhone 设备外观
- 内部区域模拟小程序页面布局，底部显示版权信息区域
- 实时预览输入的文本内容，字体、颜色、位置与小程序实际效果一致
- 版权信息区域样式：12px 灰色字体，居中对齐，与版本信息保持合理间距

#### 5.1.3 子页面背景配置选项卡

**配置区域**：

```
┌──────────────────────────────────────────────────────────┐
│                    子页面背景配置                          │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────┬────────────┬────────────┐                │
│  │  资讯详情  │  商品详情  │   购物车   │                │
│  │ [上传图片] │ [上传图片] │ [上传图片] │                │
│  │ 750×1334  │ 750×1334  │ 750×1334   │                │
│  │   [预览]   │   [预览]   │   [预览]   │                │
│  └────────────┴────────────┴────────────┘                │
│                                                          │
│  ┌────────────┬────────────┬────────────┐                │
│  │  订单详情  │   下单页   │  收货地址  │                │
│  │ [上传图片] │ [上传图片] │ [上传图片] │                │
│  │ 750×1334  │ 750×1334  │ 750×1334   │                │
│  │   [预览]   │   [预览]   │   [预览]   │                │
│  └────────────┴────────────┴────────────┘                │
│                                                          │
│  ┌────────────┬────────────┬────────────┐                │
│  │  在线客服  │  日常提醒  │  提醒编辑  │                │
│  │ [上传图片] │ [上传图片] │ [上传图片] │                │
│  │ 750×1334  │ 750×1334  │ 750×1334   │                │
│  │   [预览]   │   [预览]   │   [预览]   │                │
│  └────────────┴────────────┴────────────┘                │
│                                                          │
│  ┌────────────┬────────────┬────────────┐                │
│  │宠物档案(只读)│宠物档案(编辑)│  消息中心  │              │
│  │ [上传图片] │ [上传图片] │ [上传图片] │                │
│  │ 750×1334  │ 750×1334  │ 750×1334   │                │
│  │   [预览]   │   [预览]   │   [预览]   │                │
│  └────────────┴────────────┴────────────┘                │
│                                                          │
│  ┌────────────┬────────────┬────────────┐                │
│  │  用户反馈  │  我的反馈  │    设置    │                │
│  │ [上传图片] │ [上传图片] │ [上传图片] │                │
│  │ 750×1334  │ 750×1334  │ 750×1334   │                │
│  │   [预览]   │   [预览]   │   [预览]   │                │
│  └────────────┴────────────┴────────────┘                │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**预览区功能**：

```
┌──────────────────────────────────────────────────────────┐
│                    预览区                                │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  ┌─────────────────────────────────────────────┐   │  │
│  │  │  ⬜ 手机模拟器 375px × 812px               │   │  │
│  │  │                                           │   │  │
│  │  │     ┌─────────────────────────────────┐   │   │  │
│  │  │     │         导航栏                   │   │   │  │
│  │  │     └─────────────────────────────────┘   │   │  │
│  │  │                                           │   │  │
│  │  │     ┌─────────────────────────────────┐   │   │  │
│  │  │     │         页面内容区域             │   │   │  │
│  │  │     │                                 │   │   │  │
│  │  │     │     [根据选中页面显示对应布局]    │   │   │  │
│  │  │     │                                 │   │   │  │
│  │  │     └─────────────────────────────────┘   │   │  │
│  │  │                                           │   │  │
│  │  └─────────────────────────────────────────────┘   │  │
│  │                                                    │  │
│  │  当前页面：[资讯详情 ▼]                              │  │
│  │  [实时预览：上传图片同步显示为页面背景]                │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**预览区实现细节**：
- 使用 CSS 绘制手机边框（圆角 20px、阴影效果），模拟 iPhone 设备外观
- 内部区域模拟小程序子页面布局，包含导航栏和内容区域
- 根据选中的子页面类型，展示对应的页面结构（如商品详情页展示商品图片区域、购物车页展示购物车列表区域等）
- 上传的背景图片使用 `background-size: cover` 和 `background-position: center` 渲染，与小程序实际效果一致
- 支持下拉选择不同子页面进行预览切换
- 背景图片加载过程中显示默认背景色 `#f5f6f8` 作为占位

**图片上传区域设计**：
- 上传按钮：矩形区域（200px × 120px），带虚线边框和上传图标
- 悬停效果：边框颜色变亮，显示提示文字
- 尺寸提示：上传按钮下方显示推荐尺寸（如 `750×1334 px`）
- 上传限制：仅支持 PNG/JPG 格式，单张图片不超过 2MB
- 预览按钮：上传成功后显示，点击在预览区查看效果
- 移除按钮：上传成功后显示，支持删除已上传图片

#### 5.1.4 页面背景配置选项卡扩展

在宠物页面背景配置区域，增加收起状态背景配置：

| 配置项 | 说明 |
|--------|------|
| 宠物选项区（展开状态）背景 | 现有配置，日记录展开时使用 |
| 宠物选项区（收起状态）背景 | 新增配置，日记录收起时使用 |

#### 5.1.5 图片上传要求

| 配置项 | 推荐尺寸 | 格式 | 大小限制 | 说明 |
|--------|---------|------|---------|------|
| 子页面背景（资讯详情） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，适配 iPhone X 及以上机型 |
| 子页面背景（商品详情） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，含顶部商品图片区域 |
| 子页面背景（购物车） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，含底部结算区域 |
| 子页面背景（订单详情） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，含订单信息区域 |
| 子页面背景（下单页面） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，含收货地址和商品列表区域 |
| 子页面背景（收货地址） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，含地址列表区域 |
| 子页面背景（在线客服） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，含聊天区域 |
| 子页面背景（日常提醒） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，含提醒列表区域 |
| 子页面背景（提醒编辑） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，含表单区域 |
| 子页面背景（宠物档案-只读） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，含宠物信息展示区域 |
| 子页面背景（宠物档案-编辑） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，含表单编辑区域 |
| 子页面背景（消息中心） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，含消息列表区域 |
| 子页面背景（用户反馈） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，含反馈表单区域 |
| 子页面背景（我的反馈） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，含反馈列表区域 |
| 子页面背景（设置） | 750×1334 px | PNG/JPG | 2MB | 完整页面背景，含设置项列表区域 |
| 宠物选项区收起状态背景 | 350×380 px | PNG/JPG | 500KB | 收起状态下宠物垂直列表区域背景，包含宠物卡片和添加按钮区域 |

**尺寸说明**：
- 子页面背景推荐尺寸 `750×1334 px` 基于 iPhone X 屏幕分辨率（375×812 pt，2x 倍率）
- 实际渲染时使用 `background-size: cover`，确保适配不同屏幕尺寸
- 宠物选项区展开状态背景尺寸为 `350×80 px`（水平芯片式布局），收起状态背景尺寸为 `350×380 px`（垂直列表布局），两者尺寸不同以适配各自布局形式

---

## 6. 实施计划

### 6.1 小程序前端任务

| 任务编号 | 任务名称 | 详细描述 | 负责人 | 依赖 | 状态 |
|---------|---------|---------|--------|------|------|
| F-V3-01 | 首页版权信息配置 | 修改 pages/index/index.vue，底部版权信息动态绑定 texts.copyright.content | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-02 | 设置页面版权信息配置 | 修改 subpk-settings/index.vue，底部版本信息下方添加版权信息，动态绑定 texts.copyright.content | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-03 | 设置页面背景 | 修改 subpk-settings/index.vue，添加动态背景绑定，使用 subPages.about.background，fallback 回退到我的背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-04 | 宠物选项区收起状态背景 | 修改 pages/pet/pet.vue，根据日记录展开/收起状态切换背景，收起状态使用 collapsedBackground | 小程序前端开发 | F-01, F-V2-03 | 📋 待开发 |
| F-V3-05 | 资讯详情页面背景 | 修改 subpk-news/detail.vue，添加动态背景绑定，fallback 回退到首页背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-06 | 商品详情页面背景 | 修改 pages/product/detail.vue，添加动态背景绑定，fallback 回退到商城背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-07 | 购物车页面背景 | 修改 pages/cart/cart.vue，添加动态背景绑定，fallback 回退到商城背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-08 | 订单详情页面背景 | 修改 subpk-order/detail.vue，添加动态背景绑定，fallback 回退到订单背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-09 | 下单页面背景 | 修改 subpk-order/checkout.vue，添加动态背景绑定，fallback 回退到订单背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-10 | 收货地址页面背景 | 修改 subpk-address/list.vue，添加动态背景绑定，fallback 回退到我的背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-11 | 在线客服页面背景 | 修改 subpk-service/chat.vue，添加动态背景绑定，fallback 回退到我的背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-12 | 日常提醒页面背景 | 修改 subpk-reminder/list.vue，添加动态背景绑定，fallback 回退到宠物背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-13 | 提醒编辑页面背景 | 修改 subpk-reminder/edit.vue，添加动态背景绑定，fallback 回退到宠物背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-14 | 宠物档案页面背景（只读） | 修改 subpk-pet/info.vue，只读模式下使用 subPages.petProfile.readonly.background，fallback 回退到宠物背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-15 | 宠物档案页面背景（编辑） | 修改 subpk-pet/info.vue，编辑模式下使用 subPages.petProfile.edit.background，fallback 回退到宠物背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-16 | 消息中心页面背景 | 修改 pages/message/index.vue，添加动态背景绑定，fallback 回退到我的背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-17 | 用户反馈页面背景 | 修改 subpk-feedback/index.vue，添加动态背景绑定，fallback 回退到我的背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-18 | 我的反馈页面背景 | 修改 subpk-feedback/history.vue，添加动态背景绑定，fallback 回退到我的背景 | 小程序前端开发 | F-01 | 📋 待开发 |
| F-V3-19 | 配置常量更新 | 更新 constants/miniAppConfig.js，添加 SUB_PAGE_IDS 和 TEXT_CONFIG_IDS 常量 | 小程序前端开发 | 无 | 📋 待开发 |
| F-V3-20 | 工具层更新 | 更新 utils/themeAssets.js，扩展默认配置和合并逻辑支持 texts 和 subPages 配置项 | 小程序前端开发 | F-V3-19 | 📋 待开发 |

### 6.2 Web 管理端任务

| 任务编号 | 任务名称 | 详细描述 | 负责人 | 依赖 | 状态 |
|---------|---------|---------|--------|------|------|
| W-V3-01 | 文本配置选项卡 | 在小程序配置页面新增「文本配置」选项卡，支持版权信息编辑 | Web 前端开发 | W-01 | 📋 待开发 |
| W-V3-02 | 子页面背景配置选项卡 | 在小程序配置页面新增「子页面背景」选项卡，支持14个子页面背景配置 | Web 前端开发 | W-01, W-05 | 📋 待开发 |
| W-V3-03 | 宠物选项区背景扩展 | 在页面背景选项卡中，为宠物页面增加收起状态背景配置 | Web 前端开发 | W-V2-01 | 📋 待开发 |
| W-V3-04 | 预览区扩展 | 更新手机预览组件，支持展示文本配置和子页面背景效果 | Web 前端开发 | W-05 | 📋 待开发 |
| W-V3-05 | 配置获取逻辑更新 | 更新配置获取逻辑，支持获取 texts 和 subPages 配置项 | Web 前端开发 | B-V3-01 | 📋 待开发 |
| W-V3-06 | 保存草稿逻辑更新 | 更新保存草稿逻辑，支持保存 texts 和 subPages 配置项 | Web 前端开发 | B-V3-02 | 📋 待开发 |
| W-V3-07 | 联调测试 | 与后端联调，验证新增配置项的获取、保存、发布流程 | Web 前端开发 | W-V3-01~W-V3-06 | 📋 待测试 |

### 6.3 后端任务

| 任务编号 | 任务名称 | 详细描述 | 负责人 | 依赖 | 状态 |
|---------|---------|---------|--------|------|------|
| B-V3-01 | 配置结构扩展 | 更新配置 JSON 结构，支持 texts 和 subPages 配置段，扩展 petSelector.collapsedBackground | 后端开发 | B-01 | 📋 待开发 |
| B-V3-02 | 配置校验逻辑更新 | 更新配置校验逻辑，支持 texts 和 subPages 配置项格式校验 | 后端开发 | B-V3-01 | 📋 待开发 |
| B-V3-03 | 联调测试 | 与前端联调，验证新增配置项的接口正确性 | 后端开发 | B-V3-01~B-V3-02 | 📋 待测试 |

---

## 7. 注意事项

### 7.1 与原配置的兼容性

新增配置项为可选配置，不影响原有配置项的使用：
- 当新增配置项未设置时，使用默认样式或 fallback 链
- 配置版本号从 v3 开始，向后兼容 v1、v2 配置
- V2 的 `pages.pet.petSelector.background` 继续作为展开状态背景使用

### 7.2 文本配置特殊说明

版权信息是首个非图片类型的配置项：
- 配置值为纯文本字符串，无需图片上传
- 首页和关于我们页面共用同一配置项，确保品牌一致性
- 文本内容长度建议不超过 50 个字符

### 7.3 微信小程序限制

与原需求文档一致，所有远程图片 URL 的域名必须在微信公众平台配置「downloadFile 合法域名」。

### 7.4 性能优化

新增配置项较多，需注意：
- 配置数据缓存策略保持不变（7天）
- 子页面背景按需加载，避免页面加载时并发请求过多图片资源
- 文本配置无需额外图片加载，性能影响较小

---

## 8. 附录

### 8.1 附图索引

| 附图编号 | 页面 | 说明 |
|---------|------|------|
| 第一张附图 | 首页 | 展示底部版权信息位置 |
| 第二张附图 | 关于我们 | 展示底部版权信息位置 |
| 第三张附图 | 宠物 | 展示宠物选项区收起/展开状态 |

### 8.2 文档关系

```
需求分析-小程序UI自定义配置.md（V1，已完成）
        ↓ 引用
需求分析-小程序UI自定义配置-V2.md（V2，已完成）
        ↓ 引用
需求分析-小程序UI自定义配置-V3.md（V3，待开发）
```

### 8.3 配置项汇总表

| 配置段 | 配置项 | 类型 | 说明 |
|--------|--------|------|------|
| texts | copyright.content | string | 版权信息文本（首页/设置页面共用） |
| pages.pet | petSelector.collapsedBackground | string | 宠物选项区收起状态背景 |
| subPages | articleDetail.background | string | 资讯详情页面背景 |
| subPages | productDetail.background | string | 商品详情页面背景 |
| subPages | cart.background | string | 购物车页面背景 |
| subPages | orderDetail.background | string | 订单详情页面背景 |
| subPages | checkout.background | string | 下单页面背景 |
| subPages | address.background | string | 收货地址页面背景 |
| subPages | onlineService.background | string | 在线客服页面背景 |
| subPages | dailyReminder.background | string | 日常提醒页面背景 |
| subPages | reminderEdit.background | string | 提醒编辑页面背景 |
| subPages | petProfile.readonly.background | string | 宠物档案（只读）背景 |
| subPages | petProfile.edit.background | string | 宠物档案（编辑）背景 |
| subPages | message.background | string | 消息中心页面背景 |
| subPages | feedback.background | string | 用户反馈页面背景 |
| subPages | myFeedback.background | string | 我的反馈页面背景 |
| subPages | about.background | string | 设置页面背景 |