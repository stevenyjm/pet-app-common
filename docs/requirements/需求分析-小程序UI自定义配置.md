# 小程序 UI 自定义配置需求方案

## 1. 需求概述

### 1.1 需求背景

商户希望能够在 Web 管理端对小程序的界面 UI 进行个性化调整，包括页面背景图片、按钮图标等视觉元素，以满足不同品牌风格和运营需求。

### 1.2 配置目标

| 序号 | 配置项 | 说明 |
|------|--------|------|
| 1 | 五个标签页背景图片 | 首页、商城、宠物、订单、我的页面背景 |
| 2 | 按钮图标 | 购物车、消息通知、设置按钮的图标 |
| 3 | InfoMiniCard 卡片背景图片 | 每个 InfoMiniCard 卡片独立配置背景（手机号、邮箱、邀请码等） |

### 1.3 多端协同关系

```
┌─────────────────┐     API     ┌─────────────────┐    配置数据    ┌─────────────────┐
│   Web管理端     │ ──────────> │   后端服务      │ ─────────────> │   小程序前端    │
│ (配置编辑/发布) │             │ (/mini-app-config)│               │ (动态加载/渲染) │
└─────────────────┘             └─────────────────┘               └─────────────────┘
```

---

## 2. 技术方案设计

### 2.1 架构设计

#### 2.1.1 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                     配置数据流转                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Web端编辑  →  后端存储  →  API(/mini-app-config/public)        │
│                                      │                          │
│                                      ▼                          │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │               小程序前端配置管理                         │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │    │
│  │  │  dockConfig │  │themeAssets  │  │  shopConfig │     │    │
│  │  │ (底部导航)   │  │ (UI资源)    │  │ (店铺配置)   │     │    │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘     │    │
│  │         │               │               │              │    │
│  │         ▼               ▼               ▼              │    │
│  │  ┌─────────────────────────────────────────────────┐   │    │
│  │  │              页面与组件渲染                      │   │    │
│  │  │  BottomDock │ TabNavBar │ InfoMiniCard │ 页面   │   │    │
│  │  └─────────────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### 2.1.2 设计原则

1. **复用现有模式**：参考 `dockConfig` 的成熟实现，包括：
   - bundled default 内置默认配置
   - 远程配置拉取与缓存机制
   - 订阅发布模式实现响应式更新
   - 配置合并策略（远程覆盖默认）

2. **渐进式降级**：任何远程配置加载失败时，自动回退到内置默认样式

3. **统一 API 入口**：复用 `/mini-app-config/public` 接口，新增 `themeAssets` 配置段

### 2.2 API 接口设计

#### 2.2.1 接口说明

复用现有接口：`GET /api/v1/mini-app-config/public`

**请求参数扩展**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| keys | string | 否 | 配置项 key 列表，新增 `themeAssets` |

**响应结构扩展**：

```json
{
  "code": 0,
  "data": {
    "dock": { ... },
    "themeAssets": {
      "version": 1,
      "pages": {
        "home": { "background": "https://..." },
        "mall": { "background": "https://..." },
        "pet": { "background": "https://..." },
        "order": { "background": "https://..." },
        "mine": { "background": "https://..." }
      },
      "buttons": {
        "cart": { "icon": "https://..." },
        "message": { "icon": "https://..." },
        "settings": { "icon": "https://..." }
      },
      "components": {
        "infoMiniCard": {
          "phone": { "background": "https://..." },
          "email": { "background": "https://..." },
          "inviteCode": { "background": "https://..." },
          "fillInviteCode": { "background": "https://..." },
          "dailyReminder": { "background": "https://..." },
          "onlineService": { "background": "https://..." },
          "address": { "background": "https://..." }
        }
      }
    }
  }
}
```

#### 2.2.2 配置项数据结构

##### 2.2.2.1 页面背景配置 (`pages`)

| 字段 | 类型 | 说明 |
|------|------|------|
| pageId | string | 页面标识：`home` / `mall` / `pet` / `order` / `mine` |
| background | string | 背景图片 URL（必须是 HTTP/HTTPS） |

##### 2.2.2.2 按钮图标配置 (`buttons`)

| 字段 | 类型 | 说明 |
|------|------|------|
| buttonId | string | 按钮标识：`cart` / `message` / `settings` |
| icon | string | 图标图片 URL |

##### 2.2.2.3 组件背景配置 (`components`)

**InfoMiniCard 卡片背景**（支持每个卡片独立配置）：

| 卡片标识 | 说明 | 所在页面 |
|---------|------|---------|
| phone | 手机号卡片 | 我的 |
| email | 邮箱卡片 | 我的 |
| inviteCode | 邀请码卡片 | 我的 |
| fillInviteCode | 填写邀请码卡片 | 我的 |
| dailyReminder | 日常提醒卡片 | 我的 |
| onlineService | 在线客服卡片 | 我的 |
| address | 收货地址卡片 | 我的 |

每个卡片配置结构：

| 字段 | 类型 | 说明 |
|------|------|------|
| cardId | string | 卡片标识（见上表） |
| background | string | 背景图片 URL |

### 2.3 前端实现方案

#### 2.3.1 新增配置 Store

**文件路径**：`src/stores/themeAssets.js`

参考 [dockConfig.js](file:///C:/Users/22188/Desktop/宠物生鲜电商开发项目/pet-app/src/stores/dockConfig.js) 实现，包含：

- `getEffectiveThemeAssets()` - 获取当前生效配置
- `subscribeThemeAssets(listener)` - 订阅配置更新
- `initThemeAssetsFromCache()` - 从缓存初始化
- `refreshThemeAssetsFromRemote()` - 从远程刷新
- `bootstrapThemeAssets()` - 启动时初始化

#### 2.3.2 新增配置常量

**文件路径**：`src/constants/miniAppConfig.js`

```javascript
export const MINI_APP_CONFIG_STORAGE_KEYS = {
  DOCK: 'mini_app_config_dock',
  THEME_ASSETS: 'mini_app_config_theme_assets',
}

export const MINI_APP_CONFIG_PUBLIC_KEYS = {
  DOCK: 'dock',
  THEME_ASSETS: 'themeAssets',
}

export const INFO_MINI_CARD_IDS = {
  PHONE: 'phone',
  EMAIL: 'email',
  INVITE_CODE: 'inviteCode',
  FILL_INVITE_CODE: 'fillInviteCode',
  DAILY_REMINDER: 'dailyReminder',
  ONLINE_SERVICE: 'onlineService',
  ADDRESS: 'address',
}
```

#### 2.3.3 新增服务层

**文件路径**：`src/services/themeAssets.js`

参考 [miniAppConfig.js](file:///C:/Users/22188/Desktop/宠物生鲜电商开发项目/pet-app/src/services/miniAppConfig.js) 实现：

```javascript
export function normalizeThemeAssets(raw) { ... }
export function fetchPublicThemeAssets() { ... }
```

#### 2.3.4 页面改造方案

##### 首页 (`pages/index/index.vue`)

**改造内容**：
- 引入 `themeAssets` store
- 将硬编码的 `$page-bg` 改为动态绑定
- 添加图片加载失败回退逻辑

**关键代码示例**：

```vue
<template>
  <view class="page" :style="pageStyle">
    <!-- ... -->
  </view>
</template>

<script>
import { getEffectiveThemeAssets, subscribeThemeAssets } from '@/stores/themeAssets'

export default {
  data() {
    return {
      themeAssets: getEffectiveThemeAssets(),
      bgLoadFailed: false,
      unsubscribeThemeAssets: null,
    }
  },
  computed: {
    pageStyle() {
      if (this.bgLoadFailed) {
        return { background: '#f5f6f8' }
      }
      const bg = this.themeAssets?.pages?.home?.background
      if (bg && /^https?:\/\//i.test(bg)) {
        return { backgroundImage: `url(${bg})`, backgroundSize: 'cover' }
      }
      return { background: '#f5f6f8' }
    },
  },
  mounted() {
    this.unsubscribeThemeAssets = subscribeThemeAssets((config) => {
      this.themeAssets = config
      this.bgLoadFailed = false
    })
  },
  beforeUnmount() {
    this.unsubscribeThemeAssets?.()
  },
}
</script>
```

##### 商城 (`pages/mall/mall.vue`)

**改造内容**：同上

##### 宠物 (`pages/pet/pet.vue`)

**改造内容**：同上，注意个人信息区域的渐变背景也需要支持配置

##### 订单 (`pages/order/order.vue`)

**改造内容**：同上

##### 我的 (`pages/mine/mine.vue`)

**改造内容**：
- 页面背景动态绑定
- 购物车、消息通知、设置按钮图标动态绑定
- InfoMiniCard 每个卡片独立背景动态绑定

**按钮图标改造示例**：

```vue
<view class="fab-cart" @tap="goCart">
  <image v-if="cartIcon" :src="cartIcon" mode="aspectFit" @error="onCartIconError" />
  <text v-else class="fab-icon">🛒</text>
</view>
```

**InfoMiniCard 改造示例**：

```vue
<InfoMiniCard
  icon="📱"
  title="手机号"
  :bg-image="themeAssets?.components?.infoMiniCard?.phone?.background"
  @tap="goBindPhone"
/>
```

#### 2.3.5 组件改造方案

##### InfoMiniCard (`components/info-mini-card/InfoMiniCard.vue`)

**改造内容**：
- 支持动态背景图片
- 添加 `bgImage` prop
- 图片加载失败回退到默认白色背景

**关键代码示例**：

```vue
<template>
  <view class="mini-card" :style="cardStyle" @tap="onTap">
    <!-- ... -->
  </view>
</template>

<script>
export default {
  props: {
    bgImage: { type: String, default: '' },
    // ...其他 props
  },
  data() {
    return {
      bgLoadFailed: false,
    }
  },
  computed: {
    cardStyle() {
      if (this.bgLoadFailed) return {}
      if (this.bgImage && /^https?:\/\//i.test(this.bgImage)) {
        return {
          backgroundImage: `url(${this.bgImage})`,
          backgroundSize: 'cover',
        }
      }
      return {}
    },
  },
  methods: {
    onBgError() {
      this.bgLoadFailed = true
    },
  },
}
</script>
```

##### TabNavBar (`components/tab-nav-bar/TabNavBar.vue`)

**改造内容**：
- 支持动态背景图片（可选，根据需求决定是否需要）
- 暂时保持现有主题模式，后续可扩展

### 2.4 Fallback 方案

#### 2.4.1 图片加载失败 Fallback

| 场景 | Fallback 策略 |
|------|--------------|
| 页面背景图片加载失败 | 回退到硬编码的默认背景色 `#f5f6f8` |
| 按钮图标加载失败 | 回退到硬编码的 Emoji 图标 |
| InfoMiniCard 单卡片背景加载失败 | 回退到默认白色背景 |

#### 2.4.2 API 请求失败 Fallback

| 场景 | Fallback 策略 |
|------|--------------|
| 远程配置请求失败 | 使用本地缓存（有效期7天），缓存也无则使用内置默认 |
| 配置数据结构异常 | 使用内置默认配置 |

#### 2.4.3 配置未发布 Fallback

当商户未发布任何配置时，小程序使用以下内置默认值：

```json
{
  "pages": {
    "home": { "background": "" },
    "mall": { "background": "" },
    "pet": { "background": "" },
    "order": { "background": "" },
    "mine": { "background": "" }
  },
  "buttons": {
    "cart": { "icon": "" },
    "message": { "icon": "" },
    "settings": { "icon": "" }
  },
  "components": {
    "infoMiniCard": {
      "phone": { "background": "" },
      "email": { "background": "" },
      "inviteCode": { "background": "" },
      "fillInviteCode": { "background": "" },
      "dailyReminder": { "background": "" },
      "onlineService": { "background": "" },
      "address": { "background": "" }
    }
  }
}
```

---

## 3. Web 管理端实现方案

### 3.1 配置管理界面

#### 3.1.1 页面结构

Web 管理端「小程序配置」页面采用选项卡（Tab）形式，在页面顶部设置配置分类选项卡，与现有的「底部导航」选项卡并列：

```
┌─────────────────────────────────────────────────────────────────────┐
│                        管理端左侧菜单                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  小程序配置                                                  │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                           ↓ 点击进入右侧页面
┌─────────────────────────────────────────────────────────────────────┐
│                        小程序配置页面                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌────────────┬────────────┬────────────┬────────────┐             │
│  │ 底部导航   │ 页面背景   │ 按钮图标   │ 组件背景   │             │
│  └─────┬──────┴─────┬──────┴─────┬──────┴─────┬──────┘             │
│        │            │            │            │                    │
│        ↓            ↓            ↓            ↓                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    配置内容区域                              │   │
│  │                                                             │   │
│  │   ┌──────────────┬──────────────────────────────────────┐   │   │
│  │   │              │                                      │   │   │
│  │   │   配置区     │              预览区                   │   │   │
│  │   │              │                                      │   │   │
│  │   │  上传/编辑   │     手机模拟器实时预览                 │   │   │
│  │   │              │                                      │   │   │
│  │   └──────────────┴──────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              [保存草稿]              [发布配置]              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**选项卡说明**：

| 选项卡名称 | 说明 |
|-----------|------|
| 底部导航 | 现有功能，配置底部导航栏样式和图标 |
| 页面背景 | 新增，配置五个标签页的背景图片 |
| 按钮图标 | 新增，配置购物车、消息通知、设置按钮图标 |
| 组件背景 | 新增，配置 InfoMiniCard 各卡片的背景图片 |

#### 3.1.2 页面背景配置选项卡

**配置区域**：

```
┌──────────────────────────────────────────────────────────┐
│                    页面背景配置                            │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────┬────────────┬────────────┬────────────┐   │
│  │            │            │            │            │   │
│  │    首页    │    商城    │    宠物    │    订单    │   │
│  │ [上传图片] │ [上传图片] │ [上传图片] │ [上传图片] │   │
│  │   [预览]   │   [预览]   │   [预览]   │   [预览]   │   │
│  │            │            │            │            │   │
│  └────────────┴────────────┴────────────┴────────────┘   │
│                                                          │
│  ┌────────────┐                                          │
│  │            │                                          │
│  │    我的    │                                          │
│  │ [上传图片] │                                          │
│  │   [预览]   │                                          │
│  │            │                                          │
│  └────────────┘                                          │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**预览区功能**：
- 模拟手机屏幕（375px 宽度）
- 实时显示选中页面的背景图片效果
- 页面切换时预览区同步更新

#### 3.1.3 按钮图标配置选项卡

**配置区域**：

```
┌──────────────────────────────────────────────────────────┐
│                    按钮图标配置                            │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────┬────────────┬────────────┐                │
│  │            │            │            │                │
│  │  购物车    │  消息通知  │   设置    │                │
│  │ [上传图标] │ [上传图标] │ [上传图标] │                │
│  │   [预览]   │   [预览]   │   [预览]   │                │
│  │            │            │            │                │
│  └────────────┴────────────┴────────────┘                │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**预览区功能**：
- 模拟手机屏幕显示按钮图标位置
- 实时预览上传的图标效果（替换原有 Emoji）
- 图标大小、位置与小程序实际效果一致

#### 3.1.4 组件背景配置选项卡

**配置区域**：

```
┌──────────────────────────────────────────────────────────┐
│                    组件背景配置                            │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────┬────────────┬────────────┐                │
│  │            │            │            │                │
│  │  手机号    │   邮箱     │  邀请码    │                │
│  │ [上传图片] │ [上传图片] │ [上传图片] │                │
│  │   [预览]   │   [预览]   │   [预览]   │                │
│  │            │            │            │                │
│  └────────────┴────────────┴────────────┘                │
│                                                          │
│  ┌────────────┬────────────┬────────────┐                │
│  │            │            │            │                │
│  │ 填写邀请码 │  日常提醒  │ 在线客服   │                │
│  │ [上传图片] │ [上传图片] │ [上传图片] │                │
│  │   [预览]   │   [预览]   │   [预览]   │                │
│  │            │            │            │                │
│  └────────────┴────────────┴────────────┘                │
│                                                          │
│  ┌────────────┐                                          │
│  │            │                                          │
│  │  收货地址  │                                          │
│  │ [上传图片] │                                          │
│  │   [预览]   │                                          │
│  │            │                                          │
│  └────────────┘                                          │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**预览区功能**：
- 模拟「我的」页面卡片布局
- 实时预览每个卡片的背景图片效果
- 卡片顺序、尺寸与小程序实际效果一致

### 3.2 图片上传要求

| 配置项 | 推荐尺寸 | 格式 | 大小限制 |
|--------|---------|------|---------|
| 页面背景 | 750×1334 px | PNG/JPG | 2MB |
| 按钮图标 | 64×64 px | PNG（透明背景） | 100KB |
| InfoMiniCard 卡片背景 | 350×160 px | PNG/JPG | 500KB |

### 3.3 预览区实现方案

**技术方案**：

| 方案 | 说明 | 优点 | 缺点 |
|------|------|------|------|
| 静态模拟预览 | 使用 CSS 模拟手机界面，展示配置效果 | 实现简单，响应快速 | 与实际小程序存在一定差异 |
| iframe 嵌入预览 | 嵌入小程序 Web 预览版本 | 真实效果 | 需要额外部署 Web 版本 |
| 图片合成预览 | 将配置图片合成到手机模板图上 | 视觉效果好 | 交互性差 |

**推荐方案**：静态模拟预览

- 使用 CSS 绘制手机边框（圆角、阴影）
- 内部区域模拟小程序页面布局
- 根据选中的配置项动态更新预览内容
- 支持实时切换不同页面/卡片预览

---

## 4. 后端实现方案

### 4.1 数据库设计

#### 4.1.1 配置表结构

**表名**：复用现有 `mini_app_config` 表（Schema v15）

> 注：后端实现时复用了现有的 `mini_app_config` 表，通过 `config_key='themeAssets'` 区分配置项，无需新建表。该表已支持草稿/发布/归档状态管理和版本控制。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| config_key | varchar(64) | 配置项键，固定为 `themeAssets` |
| status | varchar(16) | 状态：`draft`-草稿，`published`-已发布，`archived`-已归档 |
| version | int | 版本号（草稿为0，发布后递增） |
| config_json | json | 配置内容（见 2.2.2） |
| published_at | datetime | 发布时间 |
| created_by | bigint | 创建人（Admin ID） |
| created_at | datetime | 创建时间 |
| updated_at | datetime | 更新时间 |

> 备注：当前系统为单商户架构，`merchant_id` 字段未在表中体现，配置为全局生效。

### 4.2 API 接口

#### 4.2.1 商户端接口（需鉴权）

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/v1/admin/mini-app-config/theme-assets` | GET | 获取当前商户的配置（含草稿） |
| `/api/v1/admin/mini-app-config/theme-assets/draft` | PUT | 更新配置（保存草稿） |
| `/api/v1/admin/mini-app-config/theme-assets/publish` | POST | 发布配置 |
| `/api/v1/admin/mini-app-config/theme-assets/publish/rollback` | POST | 回滚到历史版本 |
| `/api/v1/admin/mini-app-config/theme-assets/validate` | POST | 校验配置格式 |

#### 4.2.2 小程序端接口（无需鉴权）

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/v1/mini-app-config/public?keys=themeAssets` | GET | 获取已发布的配置 |

---

## 5. 实施计划

### 5.1 阶段一：后端接口开发（1-2天）

| 任务编号 | 任务名称 | 详细描述 | 负责人 | 依赖 | 状态 |
|---------|---------|---------|--------|------|------|
| B-01 | 创建数据库表 | **复用现有 `mini_app_config` 表**（Schema v15），通过 `config_key='themeAssets'` 区分配置项，无需新建表 | 后端开发 | 无 | ✅ 已完成 |
| B-02 | 开发商户端查询接口 | 实现 `GET /api/v1/admin/mini-app-config/theme-assets`，支持获取当前商户的配置（含草稿） | 后端开发 | B-01 | ✅ 已完成 |
| B-03 | 开发商户端更新接口 | 实现 `PUT /api/v1/admin/mini-app-config/theme-assets/draft`，支持更新配置（保存草稿） | 后端开发 | B-01 | ✅ 已完成 |
| B-04 | 开发商户端发布接口 | 实现 `POST /api/v1/admin/mini-app-config/theme-assets/publish`，支持发布配置 | 后端开发 | B-01 | ✅ 已完成 |
| B-05 | 开发小程序端公开接口 | 复用现有 `GET /api/v1/mini-app-config/public?keys=themeAssets`，无需鉴权，返回已发布配置 | 后端开发 | B-01 | ✅ 已完成 |
| B-06 | 接口联调测试 | 与前端联调，验证接口正确性 | 后端开发 | B-02~B-05 | ✅ 已完成 |

### 5.2 阶段二：小程序前端改造（2-3天）

| 任务编号 | 任务名称 | 详细描述 | 负责人 | 依赖 | 状态 |
|---------|---------|---------|--------|------|------|
| F-01 | 创建 themeAssets store | 参考 dockConfig 实现，包含 getEffectiveThemeAssets、subscribeThemeAssets、initThemeAssetsFromCache、refreshThemeAssetsFromRemote、bootstrapThemeAssets 方法 | 小程序前端开发 | 无 | ✅ 已完成 |
| F-02 | 创建 themeAssets service | 实现 normalizeThemeAssets 和 fetchPublicThemeAssets 方法 | 小程序前端开发 | F-01 | ✅ 已完成 |
| F-03 | 更新配置常量 | 在 constants/miniAppConfig.js 中添加 THEME_ASSETS 相关常量和 INFO_MINI_CARD_IDS 枚举 | 小程序前端开发 | 无 | ✅ 已完成 |
| F-04 | 首页动态背景 | 修改 pages/index/index.vue，引入 themeAssets store，实现动态背景绑定和加载失败回退 | 小程序前端开发 | F-01 | ✅ 已完成 |
| F-05 | 商城动态背景 | 修改 pages/mall/mall.vue，实现动态背景绑定和加载失败回退 | 小程序前端开发 | F-01 | ✅ 已完成 |
| F-06 | 宠物动态背景 | 修改 pages/pet/pet.vue，实现动态背景绑定和加载失败回退 | 小程序前端开发 | F-01 | ✅ 已完成 |
| F-07 | 订单动态背景 | 修改 pages/order/order.vue，实现动态背景绑定和加载失败回退 | 小程序前端开发 | F-01 | ✅ 已完成 |
| F-08 | 我的页动态背景 | 修改 pages/mine/mine.vue，实现动态背景绑定和加载失败回退 | 小程序前端开发 | F-01 | ✅ 已完成 |
| F-09 | 我的页按钮图标 | 修改 pages/mine/mine.vue，实现购物车、消息通知、设置按钮的动态图标绑定和回退 | 小程序前端开发 | F-01 | ✅ 已完成 |
| F-10 | 我的页卡片背景 | 修改 pages/mine/mine.vue，为每个 InfoMiniCard 组件传入对应的 card-id 属性 | 小程序前端开发 | F-01, F-11 | ✅ 已完成 |
| F-11 | InfoMiniCard 组件改造 | 修改 components/info-mini-card/InfoMiniCard.vue，添加 card-id prop，支持动态背景和加载失败回退 | 小程序前端开发 | 无 | ✅ 已完成 |
| F-12 | 启动时配置加载 | 在小程序启动时调用 bootstrapThemeAssets，确保配置优先加载 | 小程序前端开发 | F-01 | ✅ 已完成 |
| F-13 | Fallback 机制测试 | 验证图片加载失败、API 请求失败、未发布配置时的回退行为 | 小程序前端开发 | F-04~F-12 | ✅ 已完成 |

### 5.3 阶段三：Web 管理端开发（2-3天）

| 任务编号 | 任务名称 | 详细描述 | 负责人 | 依赖 | 状态 |
|---------|---------|---------|--------|------|------|
| W-01 | 新增选项卡布局 | 在「小程序配置」页面顶部新增选项卡（页面背景、按钮图标、组件背景），与底部导航选项卡并列 | Web 前端开发 | 无 | ✅ 已完成 |
| W-02 | 页面背景配置选项卡 | 实现五个页面（首页、商城、宠物、订单、我的）的背景图片上传和预览功能 | Web 前端开发 | W-01, W-05, W-06 | ✅ 已完成 |
| W-03 | 按钮图标配置选项卡 | 实现购物车、消息通知、设置按钮图标的上传和预览功能 | Web 前端开发 | W-01, W-05, W-06 | ✅ 已完成 |
| W-04 | 组件背景配置选项卡 | 实现七个 InfoMiniCard 卡片（手机号、邮箱、邀请码等）的背景图片上传和预览功能 | Web 前端开发 | W-01, W-05, W-06 | ✅ 已完成 |
| W-05 | 手机预览组件 | 创建模拟手机屏幕的预览组件，支持动态展示配置效果 | Web 前端开发 | 无 | ✅ 已完成 |
| W-06 | 图片上传组件 | 创建或复用图片上传组件，支持图片选择、预览、上传、删除操作 | Web 前端开发 | 无 | ✅ 已完成（复用 ImageUpload） |
| W-07 | 配置获取逻辑 | 实现从后端获取 themeAssets 配置的逻辑 | Web 前端开发 | B-02 | ✅ 已完成 |
| W-08 | 保存草稿功能 | 实现配置变更保存到后端草稿的逻辑 | Web 前端开发 | B-03 | ✅ 已完成 |
| W-09 | 发布配置功能 | 实现配置发布功能，将草稿状态改为已发布 | Web 前端开发 | B-04 | ✅ 已完成 |
| W-10 | 配置预览交互 | 实现选项卡切换时预览区同步更新，实时预览配置效果 | Web 前端开发 | W-02~W-05 | ✅ 已完成 |
| W-11 | 联调测试 | 与后端联调，验证配置的获取、保存、发布流程 | Web 前端开发 | W-07~W-10 | ✅ 已完成 |

### 5.4 阶段四：联调测试（1天）

| 任务编号 | 任务名称 | 详细描述 | 负责人 | 依赖 |
|---------|---------|---------|--------|------|
| T-01 | 前后端联调 | 小程序前端与后端接口联调，验证配置获取和应用流程 | 全端协作 | B-05, F-01~F-12 | ✅ 已完成 |
| T-02 | Web端与后端联调 | Web 管理端与后端接口联调，验证配置编辑和发布流程 | 全端协作 | B-02~B-04, W-07~W-11 | ✅ 已完成 |
| T-03 | 兼容性测试 | 在不同微信版本、不同设备上测试配置功能的兼容性 | 测试人员 | T-01 | ✅ 已完成 |
| T-04 | Fallback 机制测试 | 测试图片加载失败、网络异常、未发布配置等场景下的回退行为 | 测试人员 | T-01 | ✅ 已完成 |
| T-05 | 功能验收测试 | 商户端验收所有配置功能，确保符合需求 | 产品/商户 | T-01~T-04 | ⚠️ 验收通过但需扩展，详见 V2 需求文档 |

---

## 6. 三端任务清单汇总

### 6.1 后端任务清单

| 序号 | 任务 | 状态 | 优先级 | 备注 |
|------|------|------|--------|------|
| 1 | 创建数据库表 `mini_app_theme_assets` | ✅ 已完成 | P0 | **复用**现有 `mini_app_config` 表，通过 `config_key='themeAssets'` 区分 |
| 2 | 开发商户端查询接口 `GET /theme-assets` | ✅ 已完成 | P0 | 路径：`/admin/mini-app-config/theme-assets` |
| 3 | 开发商户端更新接口 `PUT /theme-assets` | ✅ 已完成 | P0 | 路径：`/admin/mini-app-config/theme-assets/draft` |
| 4 | 开发商户端发布接口 `POST /theme-assets/publish` | ✅ 已完成 | P0 | 路径：`/admin/mini-app-config/theme-assets/publish` |
| 5 | 开发小程序端公开接口 `GET /public?keys=themeAssets` | ✅ 已完成 | P0 | 复用现有接口，自动支持 `themeAssets` |
| 6 | 接口联调测试 | ✅ 已完成 | P1 | 等待小程序前端与 Web 管理端联调 |

### 6.2 小程序前端任务清单

| 序号 | 任务 | 状态 | 优先级 | 备注 |
|------|------|------|--------|------|
| 1 | 创建 themeAssets store | ✅ 已完成 | P0 | 包含 getEffectiveThemeAssets、subscribeThemeAssets、initThemeAssetsFromCache、refreshThemeAssetsFromRemote、bootstrapThemeAssets 方法 |
| 2 | 创建 themeAssets service | ✅ 已完成 | P0 | 实现 normalizeThemeAssets 和 fetchPublicThemeAssets 方法 |
| 3 | 更新配置常量 | ✅ 已完成 | P0 | 在 constants/miniAppConfig.js 中添加 THEME_ASSETS 相关常量和 INFO_MINI_CARD_IDS 枚举 |
| 4 | 首页动态背景改造 | ✅ 已完成 | P0 | 修改 pages/index/index.vue，引入 themeAssets store，实现动态背景绑定和加载失败回退 |
| 5 | 商城动态背景改造 | ✅ 已完成 | P0 | 修改 pages/mall/mall.vue，实现动态背景绑定和加载失败回退 |
| 6 | 宠物动态背景改造 | ✅ 已完成 | P0 | 修改 pages/pet/pet.vue，实现动态背景绑定和加载失败回退 |
| 7 | 订单动态背景改造 | ✅ 已完成 | P0 | 修改 pages/order/order.vue，实现动态背景绑定和加载失败回退 |
| 8 | 我的页动态背景改造 | ✅ 已完成 | P0 | 修改 pages/mine/mine.vue，实现动态背景绑定和加载失败回退 |
| 9 | 我的页按钮图标改造 | ✅ 已完成 | P0 | 修改 pages/mine/mine.vue，实现消息通知、设置按钮的动态图标绑定和回退 |
| 10 | 我的页卡片背景改造 | ✅ 已完成 | P0 | 修改 pages/mine/mine.vue，为每个 InfoMiniCard 组件传入 card-id 属性 |
| 11 | InfoMiniCard 组件改造 | ✅ 已完成 | P0 | 修改 components/info-mini-card/InfoMiniCard.vue，添加 card-id prop，支持动态背景和加载失败回退 |
| 12 | 启动时配置加载 | ✅ 已完成 | P1 | 在 app.vue onLaunch 中调用 bootstrapThemeAssets，onShow 中调用 refreshThemeAssetsFromRemote |
| 13 | Fallback 机制测试 | ✅ 已完成 | P1 | 需联调后验证图片加载失败、API 请求失败、未发布配置时的回退行为 |

### 6.3 Web 管理端任务清单

| 序号 | 任务 | 状态 | 优先级 | 备注 |
|------|------|------|--------|------|
| 1 | 新增选项卡布局 | ✅ 已完成 | P0 | 页面背景、按钮图标、组件背景三个选项卡 |
| 2 | 页面背景配置选项卡 | ✅ 已完成 | P0 | 五个页面背景图片上传 |
| 3 | 按钮图标配置选项卡 | ✅ 已完成 | P0 | 购物车、消息通知、设置按钮图标 |
| 4 | 组件背景配置选项卡 | ✅ 已完成 | P0 | 七个 InfoMiniCard 卡片背景 |
| 5 | 手机预览组件 | ✅ 已完成 | P0 | ThemeAssetsPreviewPanel 组件 |
| 6 | 图片上传组件 | ✅ 已完成 | P0 | 复用 ImageUpload 组件 |
| 7 | 配置获取逻辑 | ✅ 已完成 | P0 | getThemeAssetsConfigDetail API |
| 8 | 保存草稿功能 | ✅ 已完成 | P0 | saveThemeAssetsDraft API |
| 9 | 发布配置功能 | ✅ 已完成 | P0 | publishThemeAssetsConfig API |
| 10 | 配置预览交互 | ✅ 已完成 | P1 | 选项卡切换实时预览 |
| 11 | 联调测试 | ✅ 已完成 | P1 | 等待后端联调验证 |

### 6.4 依赖关系图

```
后端开发 ✅ 已完成
    ├─ B-01 创建数据库表 ✅（复用现有 mini_app_config 表）
    │   ├─ B-02 查询接口 ✅
    │   ├─ B-03 更新接口 ✅
    │   ├─ B-04 发布接口 ✅
    │   └─ B-05 公开接口 ✅
    │
小程序前端开发          Web 管理端开发
    │                         │
    ├─ F-01~F-03 创建Store   ├─ W-01 新增选项卡
    │   与Service            │
    ├─ F-04~F-08 页面改造    ├─ W-05 预览组件
    │   (动态背景)           │   W-06 上传组件
    ├─ F-09 按钮图标         ├─ W-02~W-04 各选项卡
    ├─ F-10~F-11 卡片背景    │   (依赖 W-05, W-06)
    │   与组件改造           │
    └─ F-12 启动加载         ├─ W-07~W-09 配置CRUD
                                (依赖 B-02~B-04) ✅ 后端就绪
                            └─ W-10 预览交互
```

---

## 7. 注意事项

### 7.1 微信小程序限制

1. **图片域名白名单**：所有远程图片 URL 的域名必须在微信公众平台配置「downloadFile 合法域名」
2. **图片格式限制**：支持 JPG、PNG 格式，不支持 SVG
3. **图片大小限制**：单个图片建议不超过 2MB

### 7.2 性能优化

1. **缓存策略**：配置数据本地缓存 7 天，减少网络请求
2. **图片懒加载**：页面背景图片可考虑懒加载
3. **占位符**：图片加载过程中显示默认背景色或骨架屏

### 7.3 版本管理

1. 配置版本号自增，小程序通过版本号判断是否需要更新
2. 支持配置回滚（可回退到历史版本）

### 7.4 配置一致性

1. 各子页面的配置变更统一保存到同一份配置数据中
2. 发布操作对所有配置项生效，确保配置一致性

---

## 7. 附录

### 7.1 现有配置体系参考

| 配置项 | Store | API | 缓存 Key |
|--------|-------|-----|----------|
| Dock 导航 | [dockConfig.js](file:///C:/Users/22188/Desktop/宠物生鲜电商开发项目/pet-app/src/stores/dockConfig.js) | `/mini-app-config/public?keys=dock` | `mini_app_config_dock` |
| Shop 配置 | shopConfig service | `/shop-config/public` | 无 |

### 7.2 涉及文件清单

**小程序端**：

| 文件 | 操作 | 说明 |
|------|------|------|
| `src/stores/themeAssets.js` | 新建 | 配置状态管理 |
| `src/services/themeAssets.js` | 新建 | 配置服务层 |
| `src/constants/miniAppConfig.js` | 修改 | 添加常量定义 |
| `src/pages/index/index.vue` | 修改 | 支持动态背景 |
| `src/pages/mall/mall.vue` | 修改 | 支持动态背景 |
| `src/pages/pet/pet.vue` | 修改 | 支持动态背景 |
| `src/pages/order/order.vue` | 修改 | 支持动态背景 |
| `src/pages/mine/mine.vue` | 修改 | 支持动态背景、按钮图标、卡片背景 |
| `src/components/info-mini-card/InfoMiniCard.vue` | 修改 | 支持动态背景 |

**后端**：

| 文件 | 操作 | 说明 |
|------|------|------|
| `mini_app_theme_assets` 表 | 新建 | 配置存储 |
| 主题资产 API 控制器 | 新建 | 接口实现 |

**Web 管理端**：

| 文件 | 操作 | 说明 |
|------|------|------|
| 小程序配置页面 | 修改 | 在现有页面中新增选项卡和配置区域 |
| 页面背景配置选项卡 | 新建 | 页面背景配置编辑区域 |
| 按钮图标配置选项卡 | 新建 | 按钮图标配置编辑区域 |
| 组件背景配置选项卡 | 新建 | 组件背景配置编辑区域 |
| 手机预览组件 | 新建 | 模拟手机预览窗口组件 |
| 图片上传组件 | 新建/复用 | 图片上传功能 |

---

## 8. 文档版本说明

### 8.1 版本变更记录

| 版本 | 状态 | 说明 |
|------|------|------|
| V1 | ✅ 已完成 | 基础配置功能：五个页面背景、按钮图标、InfoMiniCard 卡片背景 |
| V2 | ✅ 已完成 | 扩展配置功能：页面内各功能区域的精细配置，详见 [需求分析-小程序UI自定义配置-V2.md](file:///C:/Users/22188/Desktop/宠物生鲜电商开发项目/pet-app-common/docs/requirements/需求分析-小程序UI自定义配置-V2.md) |
| V3 | ✅ 已完成 | 扩展配置功能：版权信息文本配置、宠物选项区收起状态背景、所有子页面背景配置，详见 [需求分析-小程序UI自定义配置-V3.md](file:///C:/Users/22188/Desktop/宠物生鲜电商开发项目/pet-app-common/docs/requirements/需求分析-小程序UI自定义配置-V3.md) |

### 8.2 关联文档

- **需求分析-小程序UI自定义配置-V2.md**：基于 T-05 验收反馈的扩展需求，包含首页资讯卡片、商城分类区/商品区、宠物选项区/状态栏/日记录卡片、订单筛选区/卡片、我的个人介绍卡片等新增配置项
- **需求分析-小程序UI自定义配置-V3.md**：进一步扩展配置功能，包含版权信息文本配置、宠物选项区收起状态背景、所有子页面背景配置（✅ 已完成）
