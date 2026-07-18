# Web 管理端预览渲染对齐小程序需求方案（V1）

## 1. 需求概述

### 1.1 需求背景

商户在使用 Web 管理端进行内容编辑时，反馈现有的右侧预览渲染效果与小程序端的实际展示存在较大差异，且部分场景缺少关键的预览形态（如商品卡片、资讯卡片）。这导致商户在管理端配置完成后，无法准确预估小程序端的真实呈现效果，需要反复在小程序端验证，降低了配置效率。

本次需求旨在：

1. **补齐缺失的预览形态**：商品卡片预览、资讯卡片预览
2. **对齐已有预览的视觉效果**：商品详情、资讯详情、CMS 页面（用户协议/隐私政策/关于我们/商家资质）、UI 主题资产
3. **统一品牌视觉语言**：将预览组件的配色、间距、字号与小程序端实际样式保持一致

### 1.2 需求范围

| 序号 | 涉及页面 | 预览类型 | 改造类型 |
|------|---------|---------|---------|
| 1 | 商品编辑页 | 商城标签页下的商品卡片预览 | 新增 |
| 2 | 商品编辑页 | 商品详情页预览 | 调整对齐 |
| 3 | 资讯编辑页 | 资讯详情页预览 | 调整对齐 |
| 4 | 资讯编辑页 | 首页/商城标签页下的资讯卡片预览 | 新增 |
| 5 | CMS 页面管理 | 用户协议/隐私政策/关于我们/商家资质预览 | 调整对齐 |
| 6 | UI 主题资产 | 右侧各页面预览 | 调整对齐 |

### 1.3 文档引用

- 小程序端实际页面实现位于 `../pet-app/src/` 目录下
- Web 管理端预览组件位于 `../pet-app-admin-web/src/components/` 目录下
- 各编辑页面位于 `../pet-app-admin-web/src/views/` 目录下
- 本需求仅涉及 Web 管理端改造，不修改小程序端代码

### 1.4 总体原则

1. **视觉一致**：预览组件的配色、字号、间距、圆角、阴影需与小程序端实际渲染保持一致
2. **结构对齐**：预览组件的 DOM 结构应映射小程序端的页面结构（如顶栏、轮播、内容卡、底栏）
3. **单位换算**：小程序端使用 `rpx`（750rpx = 375px），Web 预览需按 `1rpx ≈ 0.5px` 换算或直接使用 px 等比缩放
4. **品牌色统一**：使用小程序端定义的品牌色变量
   - `$brand`: `#07c160`（主品牌色，绿色）
   - `$brand-dark`: `#06ad56`（深品牌色）
   - `$brand-light`: `#e8f8ef`（浅品牌色）
   - `$accent`: `#ff6b35`（强调色，橙色，用于价格）
   - `$text`: `#1a1a1a`（主文本）
   - `$sub`: `#8a8a8a`（次文本）
   - `$muted`: `#b0b0b0`（弱化文本）
   - `$border`: `#efefef`（边框）
   - `$page-bg`: `#f5f6f8`（页面背景）
5. **预览模式切换**：商品编辑页和资讯编辑页需支持"卡片预览"与"详情预览"两种模式切换
6. **保留交互提示**：所有预览中的按钮、链接点击时仍触发 `showWarning('预览模式：xxx不可操作')` 提示，不执行实际业务

---

## 2. 详细需求说明

### 2.1 商品编辑页 - 新增商品卡片预览

#### 2.1.1 需求描述

在商品编辑页右侧预览区，新增"商品卡片"预览模式，模拟小程序端商城标签页（`pages/mall/mall.vue`）下商品卡片的展示效果。商户可在"商品卡片"与"商品详情"两种预览模式间切换。

#### 2.1.2 预览模式切换

- 在预览面板顶部增加模式切换控件（SegmentedControl 或 RadioGroup）
- 模式选项：「商品卡片」「商品详情」
- 默认显示「商品详情」模式（保持现有行为兼容）

#### 2.1.3 商品卡片预览规格

参考小程序端 `pages/mall/mall.vue` 第 86-114 行的商品卡片结构：

| 元素 | 样式规格 | 数据来源 |
|------|---------|---------|
| 卡片容器 | 宽度 48%（双列网格），背景 `#fff`，圆角 12px，阴影 `0 1px 6px rgba(0,0,0,0.04)` | - |
| 图片区 | 高度 72px，居中显示，根据 `img_theme` 显示对应渐变背景 | `thumbnail` / `img_theme` |
| 图片主题渐变 | green: `linear-gradient(145deg, #e8f8ef, #c8eed9)`<br>orange: `linear-gradient(145deg, #fff3e6, #ffe0c2)`<br>blue: `linear-gradient(145deg, #eef3ff, #d4e4ff)`<br>pink: `linear-gradient(145deg, #fce8f0, #f5c6dc)` | `img_theme` |
| 徽章 | 左上角，圆角 4px，字号 8px，颜色 `#fff`<br>hot: `#ff6b35` / new: `#07c160` / sale: `#e74c3c` | `badge` |
| 商品图片 | `object-fit: contain`，覆盖整个图片区 | `thumbnail` |
| 占位 Emoji | 当无图片时显示，字号 36px | `img_theme` 推断或默认 `🛒` |
| 信息区 | padding: 7px 8px 8px | - |
| 商品名称 | 字号 11px，字重 600，颜色 `#1a1a1a`，2 行截断 | `product_name` |
| 价格 | 字号 13px，字重 700，颜色 `#ff6b35` | SKU 最低价 |
| 原价 | 字号 9px，颜色 `#b0b0b0`，删除线 | `original_price` |
| 加购按钮 | 圆形 24px，背景 `#07c160`，白色 `+` 号 | - |

#### 2.1.4 卡片网格布局

为更真实地模拟商城页面效果，商品卡片预览应展示为**双列网格**（至少展示 2 张卡片：1 张当前商品卡片 + 1 张占位卡片），并在网格上方显示分类标题栏（含分类名和商品数量）。

---

### 2.2 商品编辑页 - 调整商品详情预览

#### 2.2.1 需求描述

调整 `ProductPreviewPanel.vue` 组件，使其视觉与结构与小程序端 `pages/product/detail.vue` 的实际展示对齐。

#### 2.2.2 现状差异分析

| 维度 | 当前 Web 预览 | 小程序实际 | 调整方向 |
|------|-------------|-----------|---------|
| 价格颜色 | `#ee0a24` 红色 | `#ff6b35` 橙色 | 改为橙色 |
| 价格字号 | 22px | 26px（52rpx） | 调大 |
| 轮播图背景 | 纯色或无 | 主题色渐变（green/leaf/blue/orange/pink） | 增加主题渐变 |
| 轮播图高度 | 200px | 240px（480rpx） | 调高 |
| 顶栏 | 无 | 沉浸式浮动顶栏（返回按钮） | 增加 |
| 内容区顶部 | 无圆角覆盖 | 圆角 16px 向上覆盖轮播图 | 增加 |
| 标签颜色 | `#07c160` 绿色 | 按类型区分（cold 蓝/ship 绿/hot 橙） | 按类型区分 |
| 卖点图标 | 文字 Emoji | 圆角背景方块 + Emoji | 调整 |
| 详情内容 | 纯段落 | 详情块（detail-block）+ 参数表 | 增加 |
| 规格选择 | 圆角 pill | 圆角矩形 + 边框 | 调整 |
| 底栏 | 仅加购按钮 | 分享/客服/购物车/二维码/数量选择器/加购 | 补齐 |
| 加购按钮 | 矩形圆角 | 渐变背景 + 阴影 | 调整 |

#### 2.2.3 详细调整项

**A. 沉浸式顶栏**

- 浮动在轮播图上方，背景透明
- 左侧返回按钮（圆形，半透明黑底，白色 `‹`）
- 仅作视觉展示，点击触发预览提示

**B. 轮播图区域**

- 高度 240px
- 支持 `gallery` 数组多图切换
- 每张图片支持独立的 `theme`（green/leaf/blue/orange/pink）渐变背景
- 图片使用 `object-fit: contain`
- 支持图片说明文字（caption）
- 底部圆点指示器（active 状态为长条）
- 右下角图片索引（如 `1/3`）

**C. 信息卡片（info-card）**

- 顶部圆角 16px，向上覆盖轮播图 -14px
- 背景白色
- 价格行：橙色 `#ff6b35`，字号 26px，字重 700
- 原价：字号 12px，删除线
- 折扣标签：右侧橙色渐变背景
- 商品标题：字号 16px，字重 600
- 副标题：含 `subTitle · 已售 X 件`
- 标签行：按 `tag.key` 区分颜色（cold-蓝/ship-绿/hot-橙/default-灰）

**D. 规格选择卡（spec-card）**

- 独立白色卡片，顶部 8px 间距
- 标签："规格" 字号 12px，灰色
- 选项：圆角矩形（8px），边框 1.5px，padding 6px 14px
- 选中状态：品牌色边框 + 浅品牌色背景 + 深品牌色文字
- 禁用状态：opacity 0.45 + 删除线

**E. Tab 栏**

- 双 Tab：商品简介 / 商品详情
- 选中项底部 3px 品牌色短横线（28px 宽，居中）
- 选中项文字加粗

**F. 简介面板（intro-list）**

- 卖点项：图标（28px 圆角方块，浅品牌色背景）+ 标题 + 描述
- 高亮提示框（intro-highlight）：渐变背景，浅品牌色边框

**G. 详情面板（detail-blocks）**

- 详情块：圆角 10px，主题色渐变背景，居中文案
- 参数表：边框包裹，左标签右值，斑马纹背景

**H. 相关入口（link-section）**

- 标题左侧 3px 品牌色竖线装饰
- 内部链接卡：浅绿边框，绿色图标背景
- 外部链接卡：浅蓝边框，蓝色图标背景
- 圆角 10px，间距 8px

**I. 底部操作栏（action-bar）**

- 顶部分隔线 + 阴影
- 按钮组（从左到右）：
  - 分享（`⎙` 图标 + 文字）
  - 客服（`💬` 图标 + 文字）
  - 购物车（`🛒` 图标 + 文字 + 角标）
  - 二维码（`⎔` 图标 + 文字）
- 数量选择器：圆形 +/- 按钮，中间数字
- 加购按钮：渐变背景 `linear-gradient(135deg, #07c160, #06ad56)`，圆角 20px，阴影

---

### 2.3 资讯编辑页 - 调整资讯详情预览

#### 2.3.1 需求描述

调整 `ArticlePreviewPanel.vue` 组件，使其视觉与结构与小程序端 `subpk-news/detail.vue` 的实际展示对齐。

#### 2.3.2 现状差异分析

| 维度 | 当前 Web 预览 | 小程序实际 | 调整方向 |
|------|-------------|-----------|---------|
| 整体布局 | 单卡片 | 顶栏 + 滚动内容 + 底栏 | 重构 |
| 顶栏 | 无 | PageNavBar "资讯详情" | 增加 |
| 封面高度 | 160px | 168px（336rpx） | 调整 |
| 封面占位 | Emoji 居中 | 主题色渐变 + 半透明 Emoji | 调整 |
| 元信息 | 仅"预览·今日" | 日期 + 阅读数 + 来源 | 补齐 |
| 标题字号 | 20px | 19px（38rpx） | 调整 |
| 正文字号 | 15px | 14px（28rpx） | 调整 |
| 引语样式 | 浅绿背景 | 浅绿背景 + 左侧品牌色竖线 + 深绿文字 | 调整 |
| 内联示意图 | 无 | 支持 `inlineEmoji` / `inlineUrl` | 新增 |
| 相关入口 | 简化 | 完整卡片 + 标题竖线装饰 | 调整 |
| 底栏 | 仅 CTA | 分享/二维码/收藏/CTA | 补齐 |

#### 2.3.3 详细调整项

**A. 顶栏**

- 固定顶部，标题"资讯详情"
- 白色背景，底部 1px 边框

**B. 封面区域**

- 高度 168px
- 有图片：`aspectFill` 铺满
- 无图片：主题色渐变背景（green/blue/orange）+ 右侧半透明大 Emoji 装饰
- 右下角"🔍 点击预览"提示（仅视觉）

**C. 标题区**

- 白色背景
- 标签：圆角 6px，按类型区分颜色（science-绿/notice-蓝/promo-橙）
- 标题：字号 19px，字重 700
- 元信息行：日期（📅）· 阅读数（👁）· 来源，用圆点分隔

**D. 正文区**

- 段落：字号 14px，行高 1.75，颜色 `#444`，两端对齐
- 引语：浅绿背景 + 左侧 3px 品牌色竖线 + 深绿文字
- 内联示意图：圆角 10px，主题色渐变背景，支持 Emoji 或图片，左下角"示意图"标签

**E. 相关入口**

- 标题左侧 3px 品牌色竖线装饰
- 卡片：圆角 10px，浅灰背景，1px 边框
- 内部链接：浅绿边框
- 外部链接：浅蓝边框
- 图标：40px 圆角方块，浅色背景

**F. 底部操作栏**

- 分享按钮（`↗` 图标）
- 二维码按钮（`⎔` 图标）
- 收藏按钮（`♡`/`♥` 图标，可切换）
- CTA 按钮（渐变背景，圆角 20px）

---

### 2.4 资讯编辑页 - 新增资讯卡片预览

#### 2.4.1 需求描述

在资讯编辑页右侧预览区，新增"资讯卡片"预览模式，模拟小程序端首页资讯轮播卡片和资讯列表卡片的展示效果。

#### 2.4.2 预览模式切换

- 在预览面板顶部增加模式切换控件
- 模式选项：「资讯详情」「首页卡片」「列表卡片」
- 默认显示「资讯详情」模式

#### 2.4.3 首页资讯卡片预览规格

参考小程序端 `pages/index/index.vue` 第 50-72 行的资讯轮播卡片结构：

| 元素 | 样式规格 | 数据来源 |
|------|---------|---------|
| 卡片容器 | 白色背景，圆角 14px，阴影，padding 14px 16px | - |
| 顶部布局 | 左图标 + 右文字（标题+描述） | - |
| 图标区 | 44px 圆角方块（12px），按类型背景色 | `cover_emoji` / `type` |
| 图标背景 | science: `#e8f8ef`<br>notice: `#fff3e6`<br>promo: `#eef3ff` | `type` |
| 标签 | 字号 10px，圆角 4px，按类型颜色 | `tag` / `type` |
| 标题 | 字号 15px，字重 600，2 行截断 | `title` |
| 描述 | 字号 12px，灰色，2 行截断 | `desc` |
| 底部分隔 | 1px 虚线分隔 | - |
| 底部行 | 左日期 + 右"阅读全文 ›" | `date` |
| 日期 | 字号 11px，灰色 | - |
| 阅读全文 | 字号 12px，品牌色 | - |
| 轮播指示器 | 底部圆点，active 为长条 | - |

#### 2.4.4 列表资讯卡片预览规格

参考小程序端 `subpk-news/list.vue` 第 15-33 行的列表卡片结构：

| 元素 | 样式规格 | 数据来源 |
|------|---------|---------|
| 卡片容器 | 白色背景，圆角 12px，阴影，padding 14px，flex 布局 | - |
| 图标区 | 44px 圆角方块（10px），按类型背景色 | `cover_emoji` / `type` |
| 内容区 | flex: 1，垂直布局，间距 4px | - |
| 标签 | 字号 11px，品牌色 | `tag` |
| 标题 | 字号 15px，字重 600 | `title` |
| 描述 | 字号 12px，灰色，2 行截断 | `desc` |
| 日期 | 字号 11px，灰色 | `date` |
| 箭头 | 右侧 `›`，字号 18px，灰色 | - |

---

### 2.5 CMS 页面管理 - 调整预览对齐

#### 2.5.1 需求描述

调整 `CmsPreviewPanel.vue` 组件，使用户协议、隐私政策、关于我们、商家资质的预览与小程序端 `subpk-info/index.vue` 的实际展示对齐。

#### 2.5.2 现状差异分析

| 维度 | 当前 Web 预览 | 小程序实际 | 调整方向 |
|------|-------------|-----------|---------|
| 法律文档 - 顶栏 | 简化文字 | 完整 PageNavBar | 增加 |
| 法律文档 - 徽章 | 文字行 | 三个独立徽章（版本/日期/阅读时长） | 调整 |
| 法律文档 - 目录 | 静态列表 | 卡片式目录（可折叠，带编号） | 调整 |
| 法律文档 - 章节 | 平铺 | 卡片堆叠，带编号方块 | 调整 |
| 法律文档 - 高亮 | 简化 | 浅绿背景 + 左侧竖线 + 深绿文字 | 调整 |
| 关于我们 - Hero | 渐变背景 | 渐变背景 + Logo 卡片 + 阴影 | 调整 |
| 关于我们 - 卡片 | 简化 | 卡片堆叠 + 图标方块标题 | 调整 |
| 关于我们 - 联系行 | 简化 | 完整行 + 箭头 + 点击效果 | 调整 |
| 关于我们 - 法律链接 | 简化 | 完整行 + "查看" + 箭头 | 调整 |
| 商家资质 | 简化 | 复用法律文档布局 | 调整 |

#### 2.5.3 法律文档预览调整（用户协议/隐私政策/商家资质）

**A. 文档头部（doc-header）**

- 白色背景，底部 8px 灰色间隔
- 徽章行：三个独立徽章（圆角 10px）
  - 版本徽章：浅绿背景 + 深绿文字
  - 日期徽章：浅蓝背景 + 蓝色文字
  - 阅读时长徽章：浅橙背景 + 橙色文字
- 标题：字号 20px，字重 700
- 简介：字号 13px，灰色

**B. 章节目录卡片（toc-card）**

- 白色卡片，圆角 12px，阴影
- 头部：图标方块（22px，浅绿背景）+ "章节目录" + 折叠箭头
- 列表项：编号圆圈（20px）+ 标题
- 选中项：浅绿背景 + 品牌色编号

**C. 章节内容（doc-body）**

- 卡片堆叠布局，每章节独立白色卡片
- 章节头部：编号方块（24px，品牌色渐变背景，白色数字）+ 标题
- 段落：字号 13px，行高 1.75，颜色 `#555`
- 高亮框：浅绿背景 + 左侧 3px 品牌色竖线 + 深绿文字

#### 2.5.4 关于我们预览调整

**A. Hero 区域（about-hero）**

- 渐变背景 `linear-gradient(160deg, #059669, #07c160, #6ee7b7)`
- padding 28px 20px 32px
- Logo 容器：72px 圆角方块（20px），白色背景，阴影
- 品牌名：字号 22px，字重 700，白色
- 标语：字号 13px，半透明白色

**B. 卡片堆叠（about-cards）**

- padding 12px
- 每张卡片：白色背景，圆角 12px，阴影，padding 14px
- 卡片标题：图标方块（20px，浅绿背景）+ 标题文字
- 公司信息卡：描述文字 + 标签行
- 联系我们卡：联系行（标签 + 值 + 箭头）
- 法律与资质卡：链接行（标签 + "查看" + 箭头）

**C. 页脚**

- 版本号 + 版权信息
- 居中显示，字号 11px，灰色

---

### 2.6 UI 主题资产 - 调整预览对齐

#### 2.6.1 需求描述

调整 `ThemeAssetsPreviewPanel.vue` 组件，使各页面的预览内容与小程序端实际页面展示对齐，更真实地反映主题资产配置后的效果。

#### 2.6.2 现状差异分析

| 页面 | 当前 Web 预览 | 小程序实际 | 调整方向 |
|------|-------------|-----------|---------|
| 首页 | 简化占位 | 封面 Banner + 资讯轮播 + 底部商家信息 | 补齐 |
| 商城 | 分类 + 商品网格 | 活动轮播 + 分类栏 + 商品网格 + 购物车 FAB | 补齐 |
| 宠物 | 选择器 + 状态 + 记录卡 | 宠物选择器 + 每日状态 + 餐食/运动记录卡 | 调整 |
| 订单 | 筛选栏 + 订单卡 | 筛选栏 + 订单卡（含商品列表） | 调整 |
| 我的 | 介绍卡 + InfoMiniCard 网格 | 介绍卡 + InfoMiniCard 网格 + 底部 Dock | 补齐 |
| 通用 | 无底部 Dock | 所有 tab 页均有底部 Dock | 补齐 |

#### 2.6.3 首页预览调整

参考 `pages/index/index.vue`：

- **封面 Banner**：高度 228px，支持图片或渐变 fallback（含装饰圆 + Emoji + 标题文案）
- **资讯轮播区**：
  - 标题栏："最新资讯" + "更多 ›"
  - 卡片轮播：170px 高度，含图标+标签+标题+描述+日期+"阅读全文"
  - 圆点指示器
- **底部商家信息**：
  - 公司名 + 客服热线
  - 法律链接行（用户协议|隐私政策|商家资质|关于我们）
  - 版权信息

#### 2.6.4 商城预览调整

参考 `pages/mall/mall.vue`：

- **活动轮播**：88px 高度，渐变背景，含标签+标题+副标题+Emoji 装饰
- **分类栏**（左侧）：
  - 宽度 76px，白色背景，右侧圆角
  - 分类项：图标 + 名称，选中态左侧品牌色竖线
  - 底部购物车 FAB（48px 圆形）
- **商品区**（右侧）：
  - 标题栏："全部商品" + "共 X 件"
  - 双列商品网格
  - 商品卡：图片区（72px）+ 信息区（名称+价格+加购按钮）

#### 2.6.5 宠物页预览调整

参考 `pages/pet/pet.vue`（需进一步查看）：

- **宠物选择区**：横向滚动，当前选中宠物卡片
- **每日状态栏**：餐食/运动概览
- **记录卡片**：餐食记录 + 运动记录，支持背景图配置

#### 2.6.6 订单页预览调整

参考 `pages/order/order.vue`（需进一步查看）：

- **筛选栏**：全部/待支付/待发货/待收货/已完成
- **订单卡**：订单号 + 状态 + 商品列表 + 总价 + 操作按钮

#### 2.6.7 我的页预览调整

参考 `pages/mine/mine.vue`（需进一步查看）：

- **个人介绍卡**：头像 + 昵称 + 个人介绍
- **InfoMiniCard 网格**：双列布局，每张卡片含图标+标题+值
  - 手机号、邮箱、邀请码、填写邀请码、日常提醒、在线客服
- **底部 Dock**：5 个 tab 入口

#### 2.6.8 底部 Dock 栏

所有 tab 页面（首页/商城/宠物/订单/我的）预览底部均应显示 BottomDock 栏，与小程序端实际导航保持一致。Dock 样式参考 `components/bottom-dock/BottomDock.vue`。

---

## 3. 技术实现方案

### 3.1 组件改造策略

#### 3.1.1 预览模式切换机制

为支持商品和资讯编辑页的多模式预览，采用以下方案：

```vue
<!-- ProductPreviewPanel.vue 改造 -->
<template>
  <div class="product-preview-panel">
    <PreviewDisclaimer />
    <!-- 模式切换 -->
    <div class="preview-mode-switcher">
      <el-radio-group v-model="previewMode" size="small">
        <el-radio-button value="detail">商品详情</el-radio-button>
        <el-radio-button value="card">商品卡片</el-radio-button>
      </el-radio-group>
    </div>
    <!-- 详情预览 -->
    <ProductDetailPreview v-if="previewMode === 'detail'" v-bind="detailProps" />
    <!-- 卡片预览 -->
    <ProductCardPreview v-else v-bind="cardProps" />
  </div>
</template>
```

#### 3.1.2 组件拆分建议

为降低单个组件复杂度，建议将大型预览面板拆分为子组件：

| 父组件 | 子组件 | 职责 |
|--------|-------|------|
| ProductPreviewPanel | ProductDetailPreview | 商品详情页预览 |
| ProductPreviewPanel | ProductCardPreview | 商品卡片预览 |
| ArticlePreviewPanel | ArticleDetailPreview | 资讯详情页预览 |
| ArticlePreviewPanel | ArticleCardPreview | 资讯卡片预览（含首页/列表两种） |
| CmsPreviewPanel | CmsLegalPreview | 法律文档预览（用户协议/隐私政策/商家资质） |
| CmsPreviewPanel | CmsAboutPreview | 关于我们预览 |
| ThemeAssetsPreviewPanel | ThemeAssetsHomePreview | 首页预览 |
| ThemeAssetsPreviewPanel | ThemeAssetsMallPreview | 商城预览 |
| ThemeAssetsPreviewPanel | ThemeAssetsPetPreview | 宠物页预览 |
| ThemeAssetsPreviewPanel | ThemeAssetsOrderPreview | 订单页预览 |
| ThemeAssetsPreviewPanel | ThemeAssetsMinePreview | 我的页预览 |

#### 3.1.3 共享样式变量

在 `src/styles/preview-theme.scss` 中统一定义预览用的品牌色变量，与小程序端 `theme.scss` 保持一致：

```scss
// 预览主题变量（对齐小程序端）
$preview-brand: #07c160;
$preview-brand-dark: #06ad56;
$preview-brand-light: #e8f8ef;
$preview-accent: #ff6b35;
$preview-text: #1a1a1a;
$preview-sub: #8a8a8a;
$preview-muted: #b0b0b0;
$preview-border: #efefef;
$preview-page-bg: #f5f6f8;
```

### 3.2 rpx 到 px 的换算

小程序端使用 rpx 单位（750rpx = 375px 屏幕宽度），Web 预览按 `1rpx = 0.5px` 换算。常见换算对照表：

| 小程序 rpx | Web px |
|-----------|--------|
| 20rpx | 10px |
| 24rpx | 12px |
| 28rpx | 14px |
| 32rpx | 16px |
| 40rpx | 20px |
| 48rpx | 24px |
| 56rpx | 28px |
| 72rpx | 36px |
| 88rpx | 44px |
| 144rpx | 72px |

### 3.3 数据传递与兼容性

#### 3.3.1 商品卡片预览数据

商品卡片预览需要的数据已包含在现有 `ProductPreviewPanel` 的 props 中，无需新增 props：

- `productName` → 卡片名称
- `thumbnail` → 卡片图片
- `imgTheme` → 卡片图片渐变背景
- `badge` → 卡片徽章
- `skus` → 取最低价作为卡片价格
- `originalPrice` → 卡片原价

#### 3.3.2 资讯卡片预览数据

资讯卡片预览需要的数据已包含在现有 `ArticlePreviewPanel` 的 props 中：

- `title` → 卡片标题
- `summary` / `desc` → 卡片描述
- `coverEmoji` / `coverUrl` → 卡片图标
- `type` → 卡片标签和配色
- `tag` → 卡片标签文字

#### 3.3.3 向后兼容

- 预览模式切换默认保留原有预览（商品默认"详情"，资讯默认"详情"）
- 新增的卡片预览不影响现有 props 传递
- CMS 和主题资产的预览调整为样式优化，不改 props 接口

### 3.4 实施步骤建议

#### 阶段一：商品预览改造

1. 创建 `src/styles/preview-theme.scss` 共享变量文件
2. 拆分 `ProductPreviewPanel.vue` 为父组件 + `ProductDetailPreview` + `ProductCardPreview`
3. 改造 `ProductDetailPreview` 对齐小程序商品详情页
4. 实现 `ProductCardPreview` 双列网格卡片预览
5. 在父组件增加模式切换控件

#### 阶段二：资讯预览改造

1. 拆分 `ArticlePreviewPanel.vue` 为父组件 + `ArticleDetailPreview` + `ArticleCardPreview`
2. 改造 `ArticleDetailPreview` 对齐小程序资讯详情页
3. 实现 `ArticleCardPreview`（支持首页卡片 + 列表卡片两种子模式）
4. 在父组件增加模式切换控件

#### 阶段三：CMS 预览改造

1. 拆分 `CmsPreviewPanel.vue` 为父组件 + `CmsLegalPreview` + `CmsAboutPreview`
2. 改造 `CmsLegalPreview` 增加章节目录卡片、编号方块、高亮框样式
3. 改造 `CmsAboutPreview` 对齐小程序关于我们页

#### 阶段四：UI 主题资产预览改造

1. 拆分 `ThemeAssetsPreviewPanel.vue` 为父组件 + 5 个页面子预览组件
2. 实现首页预览（封面 Banner + 资讯轮播 + 底部信息）
3. 实现商城预览（活动轮播 + 分类栏 + 商品网格 + FAB）
4. 调整宠物页、订单页、我的页预览内容
5. 所有 tab 页预览底部增加 BottomDock 栏

---

## 4. 验收标准

### 4.1 商品编辑页验收

| 编号 | 验收项 | 验收标准 |
|------|-------|---------|
| W-P-01 | 预览模式切换 | 顶部显示「商品详情/商品卡片」切换控件，切换流畅 |
| W-P-02 | 商品卡片预览 | 双列网格布局，含图片区+徽章+名称+价格+原价+加购按钮 |
| W-P-03 | 商品卡片配色 | 价格橙色 `#ff6b35`，加购按钮绿色 `#07c160` |
| W-P-04 | 商品详情 - 顶栏 | 显示沉浸式浮动顶栏（返回按钮） |
| W-P-05 | 商品详情 - 轮播 | 高度 240px，支持主题渐变背景，圆点+索引指示器 |
| W-P-06 | 商品详情 - 信息卡 | 顶部圆角上覆盖，价格橙色，标签按类型区分颜色 |
| W-P-07 | 商品详情 - 规格卡 | 独立白色卡片，选中态品牌色边框+浅绿背景 |
| W-P-08 | 商品详情 - 底栏 | 含分享/客服/购物车/二维码/数量选择器/加购按钮 |
| W-P-09 | 商品详情 - 加购按钮 | 渐变背景 + 阴影 |
| W-P-10 | 交互提示 | 所有按钮点击触发预览模式提示 |

### 4.2 资讯编辑页验收

| 编号 | 验收项 | 验收标准 |
|------|-------|---------|
| W-A-01 | 预览模式切换 | 顶部显示「资讯详情/首页卡片/列表卡片」切换控件 |
| W-A-02 | 资讯详情 - 顶栏 | 显示"资讯详情"顶栏 |
| W-A-03 | 资讯详情 - 封面 | 高度 168px，支持渐变 fallback + Emoji 装饰 |
| W-A-04 | 资讯详情 - 元信息 | 显示日期 + 阅读数 + 来源 |
| W-A-05 | 资讯详情 - 引语 | 浅绿背景 + 左侧品牌色竖线 + 深绿文字 |
| W-A-06 | 资讯详情 - 底栏 | 含分享/二维码/收藏/CTA 按钮 |
| W-A-07 | 首页卡片预览 | 含图标+标签+标题+描述+日期+阅读全文，圆点指示器 |
| W-A-08 | 列表卡片预览 | 含图标+标签+标题+描述+日期+箭头 |
| W-A-09 | 卡片配色 | 按 type 区分标签颜色（science-绿/notice-橙/promo-蓝） |

### 4.3 CMS 页面管理验收

| 编号 | 验收项 | 验收标准 |
|------|-------|---------|
| W-C-01 | 法律文档 - 徽章 | 三个独立徽章（版本/日期/阅读时长） |
| W-C-02 | 法律文档 - 目录 | 卡片式章节目录，含编号圆圈 |
| W-C-03 | 法律文档 - 章节 | 卡片堆叠，编号方块（渐变背景）+ 标题 |
| W-C-04 | 法律文档 - 高亮 | 浅绿背景 + 左侧竖线 + 深绿文字 |
| W-C-05 | 关于我们 - Hero | 渐变背景 + Logo 卡片 + 阴影 |
| W-C-06 | 关于我们 - 卡片 | 三张卡片堆叠（公司信息/联系我们/法律与资质） |
| W-C-07 | 关于我们 - 联系行 | 标签 + 值 + 箭头 |
| W-C-08 | 商家资质 | 复用法律文档布局 |

### 4.4 UI 主题资产验收

| 编号 | 验收项 | 验收标准 |
|------|-------|---------|
| W-T-01 | 首页预览 | 含封面 Banner + 资讯轮播 + 底部商家信息 |
| W-T-02 | 商城预览 | 含活动轮播 + 分类栏 + 商品网格 + 购物车 FAB |
| W-T-03 | 宠物页预览 | 含宠物选择区 + 每日状态 + 记录卡片 |
| W-T-04 | 订单页预览 | 含筛选栏 + 订单卡 |
| W-T-05 | 我的页预览 | 含介绍卡 + InfoMiniCard 网格 |
| W-T-06 | 底部 Dock | 所有 tab 页预览底部显示 BottomDock |
| W-T-07 | 背景图应用 | 配置的背景图在各预览区正确应用 |

---

## 5. 风险与依赖

### 5.1 依赖项

| 依赖 | 说明 |
|------|------|
| 小程序端代码 | 作为视觉参考，仅读取不修改 |
| 现有 props 接口 | 改造不破坏现有数据传递 |
| EditorPreviewLayout | 现有布局组件支持预览模式切换控件的嵌入 |

### 5.2 风险评估

| 风险 | 影响 | 缓解措施 |
|------|------|---------|
| 预览组件复杂度增加 | 维护成本上升 | 拆分子组件，每个子组件职责单一 |
| rpx 到 px 换算误差 | 视觉细微差异 | 制定换算对照表，关键尺寸微调 |
| 小程序端后续样式变更 | 预览与实际再次出现差异 | 预览组件与小程序端代码保持同步 review |
| 预览性能 | 多模式预览可能影响渲染 | 使用 v-if 按需渲染，避免 v-show 内存占用 |
| 主题资产预览数据填充 | 部分页面需 mock 数据 | 使用静态占位数据，明确标注为示例 |

### 5.3 范围限定

- 本需求仅涉及 Web 管理端预览组件改造，**不修改小程序端代码**
- 不涉及后端接口变更
- 不涉及数据结构变更
- 不涉及路由变更
- 预览中的所有交互均为视觉展示，不执行实际业务逻辑

---

## 6. 任务清单

### 6.1 商品预览任务

> ✅ 本节任务已全部完成（2026-07-18，Web 前端文档 v0.30.0）
>
> 实现说明：
> - 新建 `src/styles/preview-theme.scss` 统一预览主题变量（品牌色/强调色/主题渐变/标签配色/徽章配色），所有变量加 `$preview-` 前缀避免与全局变量冲突
> - 新建 `src/types/preview.ts` 抽取预览组件共享类型，供父组件与子组件复用
> - 新建 `src/components/product/ProductDetailPreview.vue` 商品详情预览子组件，对齐小程序 `pages/product/detail.vue`（沉浸式顶栏、主题渐变轮播、圆角上覆盖信息卡、橙色价格、按 key 区分标签配色、独立规格卡、Tab 栏品牌色短横线、卖点列表、详情块、参数表、相关入口、完整底栏）
> - 新建 `src/components/product/ProductCardPreview.vue` 商品卡片预览子组件，对齐小程序 `pages/mall/mall.vue`（双列网格、主题渐变图片区、徽章颜色推断、橙色价格、圆形加购按钮、分类栏视觉占位）
> - 重构 `src/components/ProductPreviewPanel.vue` 为父组件，顶部增加「商品详情/商品卡片」模式切换控件，默认显示商品详情模式，保持原有 props 接口完全不变
> - `src/components/index.ts` 新增导出 `ProductDetailPreview` 和 `ProductCardPreview`
> - `npm run build` 构建验证通过

| 任务编号 | 任务描述 | 涉及文件 | 状态 |
|---------|---------|---------|------|
| W-PV-01 | 创建预览主题变量文件 | `src/styles/preview-theme.scss` | ✅ |
| W-PV-02 | 拆分 ProductPreviewPanel 为父组件 + 子组件 | `src/components/ProductPreviewPanel.vue` | ✅ |
| W-PV-03 | 创建 ProductDetailPreview 子组件 | `src/components/product/ProductDetailPreview.vue` | ✅ |
| W-PV-04 | 创建 ProductCardPreview 子组件 | `src/components/product/ProductCardPreview.vue` | ✅ |
| W-PV-05 | 实现商品详情预览对齐小程序 | `src/components/product/ProductDetailPreview.vue` | ✅ |
| W-PV-06 | 实现商品卡片预览（双列网格） | `src/components/product/ProductCardPreview.vue` | ✅ |
| W-PV-07 | 父组件增加预览模式切换控件 | `src/components/ProductPreviewPanel.vue` | ✅ |

### 6.2 资讯预览任务

> ✅ 本节任务已全部完成（2026-07-18，Web 前端文档 v0.31.0）
>
> 实现说明：
> - `src/styles/preview-theme.scss` 扩充资讯预览配色变量（封面 fallback 渐变、内联示意图渐变、资讯标签配色、卡片图标背景）
> - `src/types/preview.ts` 新增 `ArticlePreviewMode` 类型（`'detail' | 'home-card' | 'list-card'`）
> - 新建 `src/components/article/ArticleDetailPreview.vue` 资讯详情预览子组件，对齐小程序 `subpk-news/detail.vue`（顶栏、封面带主题渐变 fallback、完整元信息、引语、内联示意图、相关入口、底栏含分享/二维码/收藏/CTA）
> - 新建 `src/components/article/ArticleCardPreview.vue` 资讯卡片预览子组件（内部切换首页/列表两种子模式），首页卡片对齐 `pages/index/index.vue`，列表卡片对齐 `subpk-news/list.vue`
> - 重构 `src/components/ArticlePreviewPanel.vue` 为父组件，顶部增加「资讯详情/首页卡片/列表卡片」三模式切换控件，默认显示资讯详情模式，保持原有 props 接口完全不变
> - `src/components/index.ts` 新增导出 `ArticleDetailPreview` 和 `ArticleCardPreview`
> - `npm run build` 构建验证通过

| 任务编号 | 任务描述 | 涉及文件 | 状态 |
|---------|---------|---------|------|
| W-AV-01 | 拆分 ArticlePreviewPanel 为父组件 + 子组件 | `src/components/ArticlePreviewPanel.vue` | ✅ |
| W-AV-02 | 创建 ArticleDetailPreview 子组件 | `src/components/article/ArticleDetailPreview.vue` | ✅ |
| W-AV-03 | 创建 ArticleCardPreview 子组件 | `src/components/article/ArticleCardPreview.vue` | ✅ |
| W-AV-04 | 实现资讯详情预览对齐小程序 | `src/components/article/ArticleDetailPreview.vue` | ✅ |
| W-AV-05 | 实现首页资讯卡片预览 | `src/components/article/ArticleCardPreview.vue` | ✅ |
| W-AV-06 | 实现列表资讯卡片预览 | `src/components/article/ArticleCardPreview.vue` | ✅ |
| W-AV-07 | 父组件增加预览模式切换控件 | `src/components/ArticlePreviewPanel.vue` | ✅ |

### 6.3 CMS 预览任务

> ✅ 本节任务已全部完成（2026-07-18，Web 前端文档 v0.32.0）
>
> 实现说明：
> - `src/styles/preview-theme.scss` · 扩充 CMS 预览配色变量（法律文档徽章配色 ver/date/time、章节编号方块渐变、关于我们 Hero 渐变、高亮框样式）
> - `src/components/cms/CmsLegalPreview.vue` · 新建法律文档预览子组件，对齐小程序 `subpk-info/index.vue`（文档头部含三个独立徽章、可折叠章节目录卡片带编号圆圈、卡片堆叠章节布局带渐变背景编号方块、高亮框浅绿背景+左侧品牌色竖线+深绿文字），支持用户协议/隐私政策/商家资质三种页面类型
> - `src/components/cms/CmsAboutPreview.vue` · 新建关于我们预览子组件（渐变背景 Hero 区含 Logo 卡片+阴影、三张卡片堆叠布局公司信息/联系我们/法律与资质、联系行带标签+值+箭头、页脚版本号+版权信息）
> - `src/components/CmsPreviewPanel.vue` · 重构为父组件，按 `isLegal`/`isAbout` 判断条件渲染对应子组件，保持原有 props 接口完全不变
> - `src/components/index.ts` · 导出 `CmsLegalPreview` / `CmsAboutPreview`
> - `npm run build` 构建验证通过

| 任务编号 | 任务描述 | 涉及文件 | 状态 |
|---------|---------|---------|------|
| W-CV-01 | 拆分 CmsPreviewPanel 为父组件 + 子组件 | `src/components/CmsPreviewPanel.vue` | ✅ |
| W-CV-02 | 创建 CmsLegalPreview 子组件 | `src/components/cms/CmsLegalPreview.vue` | ✅ |
| W-CV-03 | 创建 CmsAboutPreview 子组件 | `src/components/cms/CmsAboutPreview.vue` | ✅ |
| W-CV-04 | 实现法律文档预览对齐小程序 | `src/components/cms/CmsLegalPreview.vue` | ✅ |
| W-CV-05 | 实现关于我们预览对齐小程序 | `src/components/cms/CmsAboutPreview.vue` | ✅ |

### 6.4 UI 主题资产预览任务 ✅

| 任务编号 | 任务描述 | 涉及文件 | 状态 |
|---------|---------|---------|------|
| W-TV-01 | 拆分 ThemeAssetsPreviewPanel 为父组件 + 子组件 | `src/components/ThemeAssetsPreviewPanel.vue` | ✅ |
| W-TV-02 | 创建首页预览子组件 | `src/components/theme-assets/HomePreview.vue` | ✅ |
| W-TV-03 | 创建商城预览子组件 | `src/components/theme-assets/MallPreview.vue` | ✅ |
| W-TV-04 | 创建宠物页预览子组件 | `src/components/theme-assets/PetPreview.vue` | ✅ |
| W-TV-05 | 创建订单页预览子组件 | `src/components/theme-assets/OrderPreview.vue` | ✅ |
| W-TV-06 | 创建我的页预览子组件 | `src/components/theme-assets/MinePreview.vue` | ✅ |
| W-TV-07 | 实现各页面预览对齐小程序 | 上述所有子组件 | ✅ |
| W-TV-08 | 所有 tab 页预览底部增加 BottomDock 栏 | 上述所有子组件 | ✅ |

**完成说明**：已完成全部 8 项任务。`ThemeAssetsPreviewPanel` 重构为父组件，根据 `activePageId` 动态渲染对应子组件。各子组件对齐小程序端实际页面：
- **HomePreview**：对齐 `pages/index/index.vue`（顶部渐变栏 + 资讯轮播 + 导航区）
- **MallPreview**：对齐 `pages/mall/mall.vue`（分类侧边栏 + 商品网格 + 活动 Banner）
- **PetPreview**：对齐 `pages/pet/pet.vue`（宠物选择条 + 每日状态卡片 + 记录列表）
- **OrderPreview**：对齐 `pages/order/order.vue`（状态筛选栏 + 订单卡片列表）
- **MinePreview**：对齐 `pages/mine/mine.vue`（个人信息区 + 个人介绍卡片 + InfoMiniCard 网格 + 菜单分组）
- **PreviewBottomDock**：底部导航栏，包含首页/商城/宠物/订单/我的五个槽位，支持激活态高亮

### 6.5 文档与测试任务 ✅

| 任务编号 | 任务描述 | 涉及文件 | 状态 |
|---------|---------|---------|------|
| W-DT-01 | 更新 Web 前端开发说明文档版本 | `docs/项目开发说明-Web前端.md` | ✅ |
| W-DT-02 | 创建每日工作记录 | `docs/yjm_daily/YYYY-MM-DD.md` | ✅ |
| W-DT-03 | 预览效果人工对比测试 | 商户验收 | 📋 |

**完成说明**：
- **W-DT-01**：项目开发说明文档已升级至 **v0.33.0**，修订记录包含 §6.1~§6.4 全部任务的完整摘要
- **W-DT-02**：今日工作记录文档 `docs/yjm_daily/2026-07-18.md` 已创建，包含商品预览（§6.1）、资讯预览（§6.2）、CMS 预览（§6.3）、UI 主题资产预览（§6.4）四个任务的详细开发记录
- **W-DT-03**：预览效果人工对比测试需商户在实际小程序端进行验收，建议对照小程序端各页面逐一验证 Web 预览的视觉一致性（商品详情/卡片、资讯详情/卡片、CMS 法律文档/关于我们、UI 主题资产各页面预览）

---

## 8. 任务完成总览

### 8.1 完成状态汇总

| 章节 | 任务范围 | 任务数量 | 完成数量 | 状态 |
|------|---------|---------|---------|------|
| §6.1 | 商品预览渲染对齐小程序 | 7 | 7 | ✅ |
| §6.2 | 资讯预览渲染对齐小程序 | 7 | 7 | ✅ |
| §6.3 | CMS 预览渲染对齐小程序 | 5 | 5 | ✅ |
| §6.4 | UI 主题资产预览渲染对齐小程序 | 8 | 8 | ✅ |
| §6.5 | 文档与测试任务 | 3 | 2 | ✅（W-DT-03 待验收） |
| **合计** | **Web 预览渲染对齐小程序需求** | **30** | **29** | **✅** |

### 8.2 已完成功能清单

1. **商品预览**（W-PV-01~07）
   - 新建预览主题变量文件 `styles/preview-theme.scss`
   - 新建预览共享类型文件 `types/preview.ts`
   - 新建商品详情预览子组件 `ProductDetailPreview.vue`
   - 新建商品卡片预览子组件 `ProductCardPreview.vue`
   - 重构 `ProductPreviewPanel.vue` 支持模式切换

2. **资讯预览**（W-AV-01~07）
   - 新建资讯详情预览子组件 `ArticleDetailPreview.vue`
   - 新建资讯卡片预览子组件 `ArticleCardPreview.vue`
   - 重构 `ArticlePreviewPanel.vue` 支持三模式切换

3. **CMS 预览**（W-CV-01~05）
   - 新建法律文档预览子组件 `CmsLegalPreview.vue`
   - 新建关于我们预览子组件 `CmsAboutPreview.vue`
   - 重构 `CmsPreviewPanel.vue` 支持条件渲染

4. **UI 主题资产预览**（W-TV-01~08）
   - 新建底部导航栏组件 `PreviewBottomDock.vue`
   - 新建首页预览子组件 `HomePreview.vue`
   - 新建商城预览子组件 `MallPreview.vue`
   - 新建宠物页预览子组件 `PetPreview.vue`
   - 新建订单页预览子组件 `OrderPreview.vue`
   - 新建我的页预览子组件 `MinePreview.vue`
   - 重构 `ThemeAssetsPreviewPanel.vue` 支持动态渲染

5. **文档更新**（W-DT-01~02）
   - 项目开发说明文档升级至 v0.33.0
   - 每日工作记录文档创建完成

### 8.3 待办事项

| 事项 | 说明 |
|------|------|
| W-DT-03 | 预览效果人工对比测试 - 需商户验收 |
| 联调验证 | 确认预览组件与后端配置接口的实际对接效果 |

---

## 7. 附录

### 7.1 小程序端参考文件清单

| 场景 | 文件路径 |
|------|---------|
| 商品详情页 | `../pet-app/src/pages/product/detail.vue` |
| 商城页（商品卡片） | `../pet-app/src/pages/mall/mall.vue` |
| 资讯详情页 | `../pet-app/src/subpk-news/detail.vue` |
| 资讯列表页（列表卡片） | `../pet-app/src/subpk-news/list.vue` |
| 首页（资讯轮播卡片） | `../pet-app/src/pages/index/index.vue` |
| 法律文档/关于我们/商家资质 | `../pet-app/src/subpk-info/index.vue` |
| InfoMiniCard 组件 | `../pet-app/src/components/info-mini-card/InfoMiniCard.vue` |
| 主题样式变量 | `../pet-app/src/styles/theme.scss` |

### 7.2 Web 管理端待改造文件清单

| 文件 | 改造类型 |
|------|---------|
| `src/components/ProductPreviewPanel.vue` | 拆分 + 改造 |
| `src/components/ArticlePreviewPanel.vue` | 拆分 + 改造 |
| `src/components/CmsPreviewPanel.vue` | 拆分 + 改造 |
| `src/components/ThemeAssetsPreviewPanel.vue` | 拆分 + 改造 |
| `src/styles/preview-theme.scss` | 新建 |

### 7.3 品牌色变量对照表

| 变量名 | 色值 | 用途 |
|--------|------|------|
| `$brand` | `#07c160` | 主品牌色（绿色），用于按钮、选中态、强调 |
| `$brand-dark` | `#06ad56` | 深品牌色，用于渐变结束色 |
| `$brand-light` | `#e8f8ef` | 浅品牌色，用于背景、标签 |
| `$accent` | `#ff6b35` | 强调色（橙色），用于价格、徽章 |
| `$text` | `#1a1a1a` | 主文本色 |
| `$sub` | `#8a8a8a` | 次文本色 |
| `$muted` | `#b0b0b0` | 弱化文本色 |
| `$border` | `#efefef` | 边框色 |
| `$page-bg` | `#f5f6f8` | 页面背景色 |
