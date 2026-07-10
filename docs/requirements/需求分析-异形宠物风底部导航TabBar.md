# 需求分析：异形宠物风底部导航 TabBar（Bottom Dock）

> **文档性质**：UI / 架构可行性评估与实现规格  
> **创建日期**：2026-06-07  
> **最后修订**：2026-06-08（**v0.5** · Phase 1 已实现 · 实现说明已并入开发文档）  
> **涉及模块**：Bottom Dock、主页面路由（R1/R2）、全局布局避让、商户视觉资产  
> **状态**：**✅ Phase 1 已实现**（2026-06-07）  
> **关联文档**：[项目开发说明-小程序前端.md](../pet-app/docs/项目开发说明-小程序前端.md) §2.4

---

## 1. 需求概述

底部区域为一张 **2000×90px** 常驻 PNG（**不缩放**，屏幕底部居中，左右裁切）；其上叠加若干 **圆形点击区 + 双态图标 + 代码渲染文案**。交互模型为 **Bottom Dock**（弃「TabBar / Tab 页」产品语义），点击按钮切换主页面并更新图标与文字样式。

| # | 需求摘要 | 状态 |
|---|----------|------|
| D1 | 底图 **2000×90px**，屏幕 **底部水平居中**，**禁止缩放** | ✅ 已实现 |
| D2 | 仅对左右超出部分 **裁切**；最窄参照宽度 **W_min = 320px** | ✅ 已实现 |
| D3 | PNG 中垂线 **x=1000** 与中央按钮「宠物」圆心对齐 | ✅ 已实现 |
| D4 | 底图 **不对按钮圆位挖空**；图标 **覆盖显示** 于底图之上 | ✅ 已实现 |
| D5 | 每按钮交付：pageId、pageName（两字）、circle、双态图标、**labelAnchor**、**labelStyle**、**enabled** | ✅ 已实现 |
| D6 | 点击圆形热区 → 切换页面 + 选中图标 + **labelStyle.active** | ✅ 已实现 |
| D7 | 商户坐标：**底边中点原点，x 右 y 上**（坐标系 A）；开发换算至画布像素 | ✅ 已实现 |
| D8 | **Phase 1 仅 5 个 slot**（首页 / 商城 / 宠物 / 订单 / 我的）；扩展 **Phase 3 再议** | ✅ 已实现 |
| D9 | **仅中心「宠物」按钮**图标上溢超出 90px 画布；其余按钮不超出 | ✅ 已实现 |
| D10 | 资产 **仅 1×**（不提供 @2x/@4x 高清底图） | ✅ 已实现 |

### 版本演进

| 维度 | v0.4（评审后规格） | **v0.5（本版 · 已实现）** |
|------|-------------------|---------------------------|
| 画布高度 | 2000×75 px | **2000×90 px**（底图与坐标画布一致） |
| 上溢 | `centerOverflowPx` 参考 18 | **`centerOverflowPx = 22`** |
| Slot 布局 | 宽间距示例（`cx=±800`） | **紧凑五等分**（`cx=±140,±70,0`） |
| 图标尺寸 | 示例 `r=28/36` | **普通 `r=16`（32px）· 宠物 `r=18`（36px）** |
| 文案字号 | 10px / 12px | **12px / 16px（宠物）** |
| 实现状态 | Phase 1 开发依据 | **Phase 1 已落地**；实现说明见开发文档 §2.4 |

---

## 2. 商户交付规范

### 2.1 底图资产 `dock-bg.png`

| 项 | 规格 |
|----|------|
| 尺寸 | **2000 × 90 px**（RGBA，**1× 仅此一套**） |
| 对齐 | 几何中垂线位于 PNG **x = 1000** |
| **圆位处理** | **不在按钮位置挖空**；底图可为完整不透明/渐变/装饰层，透明区域仅用于底图外轮廓（若有） |
| 文案 | **不在底图中 baked 按钮文案**；文案由代码按 `pageName` + `labelAnchor` 渲染 |
| 窄屏 | 在 **W_min = 320px** 居中裁切窗内，5 个按钮的 **图标圆 + 文案中心** 均须完整可见 |

**不挖空的原因（评审结论）**

- 图标覆盖显示即可，视觉等价于「窗口内换图」  
- Phase 3 启用 `enabled:false` 的预留 slot 时，底图 **不会出现空白圆洞**  
- `enabled:false` 的 slot **不渲染**（无图标、无文案、无热区），底图保持完整

### 2.2 每个 Slot 交付字段（Phase 1：5 条）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `pageId` | string | ✅ | 页面编号（稳定主键，如 `home`、`pet`） |
| `pageName` | string | ✅ | 页面名称，**两个字**（渲染文案内容） |
| `circle` | object | ✅ | `{ cx, cy, r }`，**坐标系 A**（§2.3） |
| `iconDefault` | PNG | ✅ | 未选中图标 |
| `iconActive` | PNG | ✅ | 选中图标 |
| `labelAnchor` | object | ✅ | `{ ax, ay }`，相对 **圆心** 的偏移，**坐标系 A**；**文本显示中心点** |
| `labelStyle` | object | ✅ | `default` / `active`，各含 `color`、`fontSize`、`isBold` |
| `enabled` | boolean | ✅ | Phase 1 五条均为 **`true`**；Phase 3 可增 slot 并设 `false` 占位 |
| `route` | string | 开发填 | uni-app 路径；商户可不填，由 `pageId` 映射表维护 |
| `navMode` | string | 开发填 | Phase 1 均为 `"tab"`；Phase 3 扩展页为 `"reLaunch"` |

**`enabled` 运行时行为**

| 值 | 底图 | 图标 | 文案 | 热区 | 导航 |
|----|------|------|------|------|------|
| `true` | 不变 | 渲染 | 渲染 | 响应点击 | 可切换 |
| `false` | 不变（无空洞） | **不渲染** | **不渲染** | **无** | **无** |

**`labelStyle` 结构（已定）**

```json
{
  "default": { "color": "#999999", "fontSize": 12, "isBold": false },
  "active": { "color": "#07C160", "fontSize": 12, "isBold": true }
}
```

- `isBold: true` → `font-weight: 600`（或 bold）；`false` → `400`  
- 禁止使用 `fontWeight` 字段，以 **`isBold`** 为准

**`labelAnchor` 语义（已定）**

- 与 `circle` 相同：**底边中点坐标系 A**（§2.3）  
- `(ax, ay)` 为 **相对圆心 `(cx, cy)` 的偏移**  
- 文本 **中心点** 位于圆心 + 偏移：  
  - 设计坐标：`(cx + ax, cy + ay)`  
  - 非左上角锚点；渲染时 `transform: translate(-50%, -50%)` 居中于该点

### 2.3 坐标系 A（商户交付 · 已确认）

**定义**

| 项 | 约定 |
|----|------|
| 画布 | PNG **2000 × 90** |
| 原点 **O** | 画布 **底边中点**（PNG 像素 **(1000, 90)**） |
| **x 轴** | 水平向右为正 |
| **y 轴** | 竖直向上为正 |
| 单位 | **px** |

**换算至开发内部画布像素（左上角原点）**

```text
px = 1000 + cx
py = 90 - cy
pr = r

/* labelAnchor：先算设计坐标再换算 */
lx = 1000 + (cx + ax)
ly = 90 - (cy + ay)
```

**约束**

- 中央「宠物」slot：**`circle.cx === 0`**（与 PNG x=1000 中垂线重合）  
- 代码端统一经 `coordinate.js` 换算，**商户仅提供坐标系 A 数值**

### 2.4 完整配置示例（Phase 1 · 当前实现）

与 `src/config/dock.theme.json` 一致：

```json
{
  "version": 1,
  "background": "/static/dock/dock-bg.png",
  "canvas": { "width": 2000, "height": 90 },
  "coordinateMeta": {
    "type": "bottom-center-up",
    "canvasWidth": 2000,
    "canvasHeight": 90,
    "unit": "px"
  },
  "minViewportWidthPx": 320,
  "centerOverflowPx": 22,
  "slots": [
    {
      "pageId": "home",
      "pageName": "首页",
      "enabled": true,
      "circle": { "cx": -140, "cy": 34, "r": 16 },
      "labelAnchor": { "ax": 0, "ay": -22 },
      "iconDefault": "/static/tabbar/home.png",
      "iconActive": "/static/tabbar/home-active.png",
      "labelStyle": {
        "default": { "color": "#999999", "fontSize": 12, "isBold": false },
        "active": { "color": "#07C160", "fontSize": 12, "isBold": true }
      },
      "route": "/pages/index/index",
      "navMode": "tab"
    },
    {
      "pageId": "pet",
      "pageName": "宠物",
      "enabled": true,
      "circle": { "cx": 0, "cy": 60, "r": 18 },
      "labelAnchor": { "ax": 0, "ay": -31 },
      "iconDefault": "/static/tabbar/pet.png",
      "iconActive": "/static/tabbar/pet-active.png",
      "labelStyle": {
        "default": { "color": "#999999", "fontSize": 16, "isBold": true },
        "active": { "color": "#07C160", "fontSize": 16, "isBold": true }
      },
      "route": "/pages/pet/pet",
      "navMode": "tab"
    }
  ]
}
```

> 完整 5 slot 见仓库 `src/config/dock.theme.json`。`centerOverflowPx = 22` 覆盖 pet `cy=60, r=18` 几何上溢。

### 2.5 最窄屏可见窗（W_min = 320px）

```text
可见画布 x 区间：[1000 - 160, 1000 + 160] = [840, 1160]
```

商户导出前须在该 **320×90** 参考框内自检：5 个 **circle + labelAnchor 文本中心** 均在框内。

---

## 3. 现状与改造范围

| 项 | 改造前 | Phase 1 目标 | 当前状态 |
|----|--------|--------------|----------|
| UI | `custom-tab-bar` 矩形 flex | **`BottomDock.vue`** + `dock.theme.json` | ✅ |
| 语义 | TabBar / Tab index | **Dock / pageId** | ✅ |
| 路由 | 分散 `switchTab` | **`navigatePrimary(pageId)`** 封装 **R1** | ✅ |
| 高度 | 56px | **90px + centerOverflowPx** | ✅ |
| 文案 | wxml `<text>` | **labelAnchor + labelStyle** | ✅ |

---

## 4. 可行性结论

| 维度 | 结论 |
|------|------|
| 不缩放 + 裁切 + 320 W_min | ✅ 可行 · 已实现 |
| 不挖空 + 图标覆盖 | ✅ 可行；优于挖空 + enabled 组合 |
| labelAnchor 精确定位 | ✅ 可行 · 已实现 |
| enabled 扩展 | ✅ 可行；false 时零 UI footprint |
| R1（5 主页面） | ✅ 可行 · 已实现 |
| R2（Phase 3 扩展页） | ✅ 可行；与 R1 共存于 `primaryNav`（架构预留） |
| 1× 资源无 @2x | ✅ 可行；Retina 略糊可接受 |
| 中心按钮上溢 | ✅ 可行；`centerOverflowPx` + `DOCK_ENVELOPE_PX` 全局避让 |

**结论**：**Phase 1 已按 v0.4 评审规格落地**；画布尺寸与 slot 数值以 v0.5 / 仓库配置为准。

---

## 5. UI / 布局

### 5.1 零缩放 + 居中裁切

```
|←────────── 2000px 画布 ──────────→|
[··裁切··|════ 320px 可见 ════|··裁切··]
              ↑ 屏宽 W，居中于 x=1000
```

- 画布：**2000×90 px**，不 `scale`  
- 视口：`width:100%`，`height: DOCK_ENVELOPE_PX`，`overflow-x:hidden`  
- 定位：`left:50%` + `marginLeft:-1000px`（画布居中）

### 5.2 图层与 Slot 渲染

```text
z-order（自下而上）：
  1. dock-bg.png（2000×90，无按钮挖空）
  2. 各 enabled slot：icon（覆盖于 circle 区域）
  3. 各 enabled slot：pageName 文本（labelAnchor 中心点）
  4. 透明圆形热区（与 circle 同心，r 等）
```

| 元素 | 规则 |
|------|------|
| 图标 | 以 `(px, py)` 为圆心，`width/height = 2×pr`；可超出 90px 顶边（仅 pet） |
| 文案 | `pageName` 两字；中心在 `(lx, ly)`；样式取 `labelStyle.default/active` |
| 热区 | 圆心 `(px, py)`，半径 `pr`；`2×pr < 44` 时扩展至 44pt |
| 选中 | 唯一 `activePageId`：`iconActive` + `labelStyle.active` |

### 5.3 中心「宠物」上溢（D9）

| 项 | 说明 |
|----|------|
| 画布高度 | 底图仍为 **90px** |
| 图标 | 可向上超出画布顶边（`py - pr < 0`） |
| 配置 | `centerOverflowPx`：**22px**（覆盖 pet 几何） |
| 布局常量 | `DOCK_ENVELOPE_PX = 90 + centerOverflowPx = 112` |
| 页面避让 | 主页面 `padding-bottom`、FAB 使用 **包络高度** |
| 槽位偏移 | 全部 slot Y 加 `centerOverflowPx`，底图贴底 |

其余 4 个 slot：**圆与图标均不得超出 y∈[0,90] 设计范围**。

### 5.4 1× 资产与 Retina（D10）

- 仅使用 **2000×90 @1×** PNG；不提供 @2x/@4x  
- Retina 屏可能略糊；**评审接受**，后续若有需要再增高清包

### 5.5 安全区

- 结构：`90px 画布` + `padding-bottom: env(safe-area-inset-bottom)`  
- Home Indicator 区：底图延伸色或纯色条，**不放置 circle/label**

---

## 6. 技术架构

### 6.1 组件结构

```text
BottomDock（fixed bottom, z-index:9999）
├─ .dock-viewport       overflow-x:hidden; height:DOCK_ENVELOPE_PX
│  └─ .dock-canvas      2000×DOCK_ENVELOPE_PX, centered
│     ├─ image.dock-bg  2000×90, bottom-aligned
│     └─ template v-for slot in enabledSlots
│        ├─ .dock-hit
│        ├─ image.dock-icon
│        └─ text.dock-label
└─ .dock-safe           safe-area-inset-bottom
```

- **enabledSlots** = `slots.filter(s => s.enabled !== false)`  
- Phase 1：过滤后恒为 **5** 条

**挂载**：5 个主页面各引入 `<BottomDock :active-page-id="..." />`；二级页不挂载。

### 6.2 路由策略（评审确认：R1 + R2 混合）

#### R1 · 现有 5 主页面（Phase 1 · ✅ 已实现）

| 项 | 做法 |
|----|------|
| `pages.json` | **保留** `tabBar.list`（5 项）；`custom: true` 空壳 |
| Dock 点击 | `navigatePrimary(pageId)` → 内部 **`uni.switchTab`** |
| 选中态 | 主页面传入 `active-page-id` |

#### R2 · Phase 3 扩展主页面（如健康、运动 · 未实现）

| 项 | 做法 |
|----|------|
| `pages.json` | 扩展页为 **普通 page**（不在 `tabBar.list`） |
| slot 配置 | `navMode: "reLaunch"` |
| Dock 点击 | `navigatePrimary(pageId)` → 内部 **`uni.reLaunch`** |
| 选中态 | 全局 store / `primaryNav` 模块维护 `activePageId` |

**Phase 1 范围**：5 slot 均为 `navMode: "tab"`；R2 分支已在 `primaryNav.js` 预留。

### 6.3 代码结构

```text
src/
├── config/dock.theme.json
├── static/dock/dock-bg.png
├── components/bottom-dock/
│   ├── BottomDock.vue
│   └── coordinate.js      # toCanvasCircle, toCanvasLabel
├── utils/primaryNav.js    # navigatePrimary, PAGE_ID
└── utils/layout.js        # DOCK_*, getFabBottomRpx()
```

**废弃（Phase 1 已完成）**：`custom-tab-bar` UI 逻辑 → `BottomDock`；`tabBar.js` → `primaryNav.js` 薄迁移。

### 6.4 坐标换算（coordinate.js）

```javascript
const CANVAS_W = 2000
const CANVAS_H = 90
const ORIGIN_X = 1000

/** circle { cx, cy, r } → 画布像素 */
export function toCanvasCircle({ cx, cy, r }) {
  return {
    px: ORIGIN_X + cx,
    py: CANVAS_H - cy,
    pr: r,
  }
}

/** labelAnchor { ax, ay } 相对圆心 → 文本中心画布像素 */
export function toCanvasLabel(circle, anchor) {
  const lx = circle.cx + anchor.ax
  const ly = circle.cy + anchor.ay
  return {
    px: ORIGIN_X + lx,
    py: CANVAS_H - ly,
  }
}
```

### 6.5 CSS 定位要点

```text
.dock-icon-wrap {
  left: (px - pr)px; top: (py - pr + centerOverflowPx)px;
  width: (2 * pr)px; height: (2 * pr)px;
}
.dock-label {
  left: labelPx; top: (labelPy + centerOverflowPx)px;
  transform: translate(-50%, -50%);
  font-size: labelStyle.fontSize px;
  font-weight: labelStyle.isBold ? 600 : 400;
  color: labelStyle.color;
}
```

### 6.6 布局常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `DOCK_CANVAS_WIDTH_PX` | 2000 | 裁切计算 |
| `DOCK_CANVAS_HEIGHT_PX` | 90 | 底图高 |
| `DOCK_CENTER_OVERFLOW_PX` | 22 | 仅 pet 上溢 |
| `DOCK_ENVELOPE_PX` | 112 | FAB / padding 避让 |
| `W_MIN_PX` | 320 | 验收裁切宽度 |

---

## 7. Phase 3 扩展（健康 / 运动 · 再议）

| 项 | 策略 |
|----|------|
| 时间 | **Phase 3 再议** |
| Phase 1 | `slots` **仅 5 条**，均 `enabled: true` |
| 底图 | 可换更大装饰稿，**仍不挖空** |
| 新 slot | 增加配置 + `enabled: true` + **`navMode: reLaunch`** + 新页面 |
| 未启用 slot | `enabled: false`，**不渲染、底图无洞** |
| tabBar 5 项上限 | 扩展页 **不走 switchTab**，走 **R2** |

---

## 8. 风险与 Mitigation

| 风险 | Mitigation |
|------|------------|
| 320 屏裁切按钮/文案 | 商户 W_min 参考框自检；AC2 |
| enabled 与底图挖空混用 | **已禁止挖空**；文档与验收写死 |
| R1/R2 选中态不一致 | 统一 `activePageId`；reLaunch 页 onShow 同步 |
| 1× Retina 模糊 | 评审已接受；记录为已知限制 |
| 中心上溢挡内容 | `DOCK_ENVELOPE_PX` 全局避让 |
| rpx 误用导致缩放 | Dock 层 **仅 px** |

---

## 9. 实施路线图

### Phase 0 · 商户资产 — ✅

1. `dock-bg.png`（**无按钮挖空**）  
2. 5 条 slot 完整 JSON（含 **labelAnchor、enabled:true**）  
3. 320px 裁切自检  

### Phase 1 · Dock 落地 — ✅（2026-06-07）

1. `coordinate.js` + `dock.theme.json` + `BottomDock.vue`  
2. `primaryNav.js`（**R1**）  
3. 5 主页面挂载；`layout.js` 更新 **DOCK_ENVELOPE**  
4. 下线 `custom-tab-bar` UI；保留 tabBar.list  

### Phase 2 · 路由封装统一（P1，待续）

1. 全站 `switchTab` → `navigatePrimary(pageId)`  
2. H5 同组件（若需要）  

### Phase 3 · 扩展 slot + R2（再议）

1. 新增扩展页与普通 page 注册  
2. slot + `navMode: reLaunch`  
3. `primaryNav` 启用 R2 分支  

---

## 10. 验收标准

| # | 验收项 | Phase 1 |
|---|--------|---------|
| AC1 | 画布 **2000×90 px**，无 scale | ✅ |
| AC2 | **320px** 宽下，5 个 enabled slot 的 **图标圆 + 文案中心** 均在屏内 | ✅ |
| AC3 | 底图 **无按钮圆孔**；图标覆盖显示 | ✅ |
| AC4 | PNG x=1000 与「宠物」`circle.cx=0` 对齐（≤2px） | ✅ |
| AC5 | 文案中心与 `labelAnchor` 换算位置一致（≤2px） | ✅ |
| AC6 | `labelStyle`：`color` / `fontSize` / `isBold` 切换正确 | ✅ |
| AC7 | 点击热区 → R1 `switchTab` 正确页；唯一 active | ✅ |
| AC8 | 中心 pet 上溢可见；其余 4 slot 不超出 90px 画布 | ✅ |
| AC9 | `DOCK_ENVELOPE_PX` 下 FAB 不被挡 | ✅ |
| AC10 | 安全区正常；二级页无 Dock | ✅ |
| AC11 | （Phase 3）`enabled:false` slot 无任何 UI；底图无空洞 | — |

---

## 11. 评审确认事项（2026-06-07）

以下 8 项经商户确认，作为 v0.4 实现依据（Phase 1 已闭合）：

| # | 事项 | 确认结论 |
|---|------|----------|
| 1 | 最窄参照宽度 | **W_min = 320px** |
| 2 | 商户坐标系 | **坐标系 A**（底边中点，x 右 y 上）；代码端换算至画布左上角像素 |
| 3 | 文案渲染 | 代码渲染 `pageName`；位置由 **`labelAnchor`** 指定（**文本中心点**） |
| 4 | 文案样式 | **`labelStyle.default / active`**：`color`、`fontSize`、**`isBold`** |
| 5 | Phase 1 路由 | **R1**（5 主页面 `switchTab`，对外 `navigatePrimary`） |
| 6 | Phase 1 范围 | **仅 5 个 slot**；Phase 3 扩展再议 |
| 7 | 高清资产 | **暂不提供** @2x/@4x；仅 1× |
| 8 | 上溢 | **仅中心「宠物」**可超出画布；配置 `centerOverflowPx` |

**附加评审结论**

| 项 | 结论 |
|----|------|
| 底图挖空 | **取消**；图标覆盖；配合 **`enabled`** 避免禁用 slot 空洞 |
| Phase 3 扩展路由 | **R2**（`reLaunch`），与 Phase 1 **R1** 混合 |
| §2.2 字段 | **`labelAnchor`、`enabled` 为必填** |

---

## 12. 修订记录

| 版本 | 日期 | 说明 |
|------|------|------|
| v0.1 | 2026-06-07 | 曲多边形 SVG + 贴边 |
| v0.2 | 2026-06-07 | PNG + 缩放 + TabBar |
| v0.3 | 2026-06-07 | 零缩放裁切 · Dock 语义 · 坐标可自选 |
| v0.4 | 2026-06-07 | **评审闭合**：不挖空 · labelAnchor/enabled · 坐标 A · R1+R2 · 320 W_min · 1× · pet 上溢 |
| v0.5 | 2026-06-08 | **Phase 1 已实现**；画布 90px · 实现数值同步；去「临时」转正式档；实现说明并入开发文档 §2.4 |

---

**文档结束 · 日常开发维护请以 [项目开发说明-小程序前端.md](../pet-app/docs/项目开发说明-小程序前端.md) §2.4 为准；本文档保留完整规格与商户交付细节。**