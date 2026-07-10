# CMS「关于我们」页 · Web 管理端功能设计

> **文档类型**：页面功能设计（交付 Web 前端项目 `pet-app-admin-web`）  
> **文档版本**：v1.1  
> **创建日期**：2026-06-14  
> **更新日期**：2026-06-14  
> **需求来源**：商户确认 — 小程序 C 端「关于我们」展示为权威样式；Web CMS 需与之对齐并支持部分字段可配置  
> **关联仓库**  
> - C 端参考实现：`pet-app` · `src/constants/info.js`（`ABOUT_INFO`）· `src/pages/info/index.vue`  
> - Web 管理端：`pet-app-admin-web` · `views/cms/Index.vue` · `CmsContentEditor.vue` · `CmsPreviewPanel.vue`  
> - 后端 API：`pet-app-backend` · `GET/PUT /admin/cms/pages/{type}` · `GET /cms/pages/{type}`  
> **关联文档**  
> - [项目开发说明-Web前端.md](../pet-app-admin-web/docs/项目开发说明-Web前端.md) §2.8 · §4.3 · §5.4  
> - [项目开发说明-小程序前端.md](../pet-app/docs/项目开发说明-小程序前端.md) §4.10 · §5.9a  
> - [项目开发说明-后端.md](../pet-app-backend/docs/项目开发说明-后端.md) §5.14 · §5.22  

---

## 1. 背景与问题

### 1.1 现象

用户在小程序 **首页 Tab → 页脚「关于我们」** 进入信息页后，看到的为 **卡片式布局**（品牌 Hero + 公司信息 + 联系我们 + 法律与资质）。  
当前 Web 管理端 **CMS 页面管理 → 关于我们** 使用的是与 **用户协议 / 隐私政策** 相同的 **法律文档编辑器**（版本号、章节 `sections[]`、目录预览等），保存后 C 端 **并不按该结构渲染**。

### 1.2 根因（C 端现状）

| 项 | 说明 |
|----|------|
| 布局分支 | C 端 `pages/info/index.vue` 在 `type=about` 时走 **`view-about` 卡片布局**，不读 `page.sections` |
| 数据来源 | 「关于我们」卡片内容来自常量 **`ABOUT_INFO`**；Hero 区 Logo Emoji、品牌名 **写死在模板**，slogan 亦来自常量；**均未**从 CMS API 读取 |
| CMS 接口 | `GET /cms/pages/about` 已存在，但 C 端对该 type 仅用于判定页存在；正文块与 UI 字段未打通 |

### 1.3 商户决策

1. **以当前小程序前端展示为准**（布局、分区、交互），Web CMS 编辑与预览须对齐该样式。  
2. **文案修正**（C 端与 Web 预览同步）：  
   - 「联系我们」中 **「微信公众号」→「微信服务号」**（标签文案，非字段值）。  
   - 「法律与资质」中 **「食品经营许可证」→「商家资质」**（与 `type=merchant` 页标题及首页页脚一致）。  
3. **CMS 可编辑范围**（本期）：  
   - **品牌 Hero（页顶）**：Logo 图标（图片上传）、Logo 文案（品牌名）、两行小字号描述（主 slogan / 副 slogan）。  
   - **公司信息**：正文描述 + 其下方 **标签**（tags）。  
   - **联系我们**：客服热线号码、商务邮箱地址、微信服务号名称、公司地址。  
4. **本期不在 CMS 开放编辑**（保持 C 端内置或与 `merchant` 等页联动）：法律与资质三条链接本身、页脚版本号与备案文案、Hero 区渐变背景样式。

---

## 2. 目标与范围

### 2.1 Web 端交付目标

| # | 目标 |
|---|------|
| G1 | CMS「关于我们」Tab 使用 **专用表单编辑器**，不再复用法律文档 `sections` 编辑器 |
| G2 | 右侧 **`CmsPreviewPanel`** 的 `about` 分支渲染与 C 端卡片布局 **视觉/文案一致**（含上述两处标签修正） |
| G3 | 保存后 `content_json` 结构 **稳定、可版本化**，供 C 端后续对接（见 §7 协同） |
| G4 | 表单校验、空状态、保存 Toast 与现有 CMS 页（FE-207）体验一致 |
| G5 | Hero Logo 支持 **上传 / 清空 / 预览回退**，与店铺封面、`ImageUpload` 体验一致 |

### 2.2 范围边界

| 包含 | 不包含 |
|------|--------|
| Web CMS 关于我们编辑 UI + 预览 + 类型定义 + API 封装适配 | C 端读取 CMS 字段（需 C 端单独迭代，见 §7） |
| `content_json` 新 Schema 设计与默认值迁移说明 | `shop_config` 联系方式字段合并（仍独立维护，见 §5.3） |
| Hero 区 Logo / 品牌名 / slogan 可编辑 + 上传 | 关于我们保存二次确认（仅 `user`/`privacy` 需要 `confirmed: true`） |
| 预览内法律区、页脚版本（只读占位） | Hero 渐变背景、Logo 容器圆角等 **样式**在线配置 |

---

## 3. C 端权威 UI 参考

以下为 Web 预览与字段对齐的 **唯一视觉基准**（与 `ABOUT_INFO` + `info/index.vue` 一致）。

### 3.1 页面结构（自上而下）

```text
┌─────────────────────────────────────┐
│  Hero                          【可编辑】│
│  Logo 图 · 品牌名 · slogan · sloganSub │
│  （渐变背景样式固定，不可配置）         │
├─────────────────────────────────────┤
│ 🏢 公司信息                    【可编辑】│
│   description（多行正文）            │
│   [tag] [tag] [tag]            【可编辑】│
├─────────────────────────────────────┤
│ 📞 联系我们                    【可编辑】│
│   客服热线    400-888-6688           │
│   商务邮箱    service@zzp.com        │
│   微信服务号  追煮跑官方    ← 标签已改 │
│   公司地址    上海市浦东新区          │
├─────────────────────────────────────┤
│ ⚖ 法律与资质（只读 · 固定三条）       │
│   用户协议 / 隐私政策 / 商家资质 ← 标签已改 │
├─────────────────────────────────────┤
│ 版本 & 版权（只读）                  │
└─────────────────────────────────────┘
```

### 3.2 交互（预览说明用，Web 预览可 Toast 模拟）

| 联系项 | C 端行为 | Web 预览 |
|--------|----------|----------|
| 客服热线 / 商务邮箱 / 公司地址 | 点击复制到剪贴板 | Toast「已复制」 |
| 微信服务号 | 点击 Toast 引导搜索关注 | Toast「请在微信搜索：xxx」 |

### 3.3 文案修正对照表

| 位置 | 原文案 | 新文案 | 备注 |
|------|--------|--------|------|
| 联系我们 · 行标签 | 微信公众号 | **微信服务号** | 仅改 label，不改 `key` |
| 法律与资质 · 第三项 | 食品经营许可证 | **商家资质** | 跳转 `type=merchant`，与首页页脚一致 |

C 端 fallback 常量同步修改见 §7.1；Web 预览 **不得**再出现旧文案。

### 3.4 Hero 区字段与 C 端现状对照

| UI 元素 | C 端当前实现 | 对应 CMS 字段 | 说明 |
|---------|--------------|---------------|------|
| Logo 图标 | 模板硬编码 Emoji `🐾`（非图片文件） | `hero.logo_url` | 对接 CMS 后：有 URL 则 `<image>`，否则回退 Emoji |
| Logo 文案 | 模板硬编码 `追煮跑` | `hero.brand_name` | 主标题，大号白字 |
| 主 slogan（小字 1） | `ABOUT_INFO.slogan` | `hero.slogan` | 如「专注宠物鲜食 · …」 |
| 副 slogan（小字 2） | `ABOUT_INFO.slogan_sub` | `hero.slogan_sub` | 如「让每一只毛孩子…」 |

> C 端参考：`src/pages/info/index.vue`（Hero 模板）· `src/constants/info.js`（`ABOUT_INFO.slogan` / `sloganSub` 默认值）。

---

## 4. 数据模型

### 4.1 存储位置

继续使用现有表 **`cms_page`**，`type = 'about'`。  
仅调整 **`content_json`** 结构；**无需**新增 DB 列（后端若 seed 仍为旧 `sections` 结构，需一并更新默认 JSON，见 §7.2）。

### 4.2 `content_json` Schema（v2 · about 专用）

```typescript
/** types/cms.ts — 新增/扩展 */
export interface CmsAboutHero {
  /** Logo 图片 HTTPS URL；空/null 时 C 端回退默认 Emoji */
  logo_url: string | null
  /** Logo 下方品牌名（主标题） */
  brand_name: string
  /** Hero 主 slogan（小字号描述第 1 行） */
  slogan: string
  /** Hero 副 slogan（小字号描述第 2 行） */
  slogan_sub: string
}

export interface CmsAboutContactItem {
  /** 展示标签，本期前端写死，不开放商户改 label */
  label: string
  value: string
}

export interface CmsAboutContentJson {
  /** 固定为 about，便于 C 端与 Admin 校验 */
  layout: 'about'
  /** 页顶品牌 Hero */
  hero: CmsAboutHero
  /** 公司信息 · 正文 */
  description: string
  /** 公司信息 · 标签，有序数组 */
  tags: string[]
  /** 联系我们 · 四项；key 固定，仅存 value */
  contacts: {
    phone: CmsAboutContactItem    // label 固定「客服热线」
    email: CmsAboutContactItem    // label 固定「商务邮箱」
    wechat: CmsAboutContactItem   // label 固定「微信服务号」
    address: CmsAboutContactItem  // label 固定「公司地址」
  }
}
```

**PUT 请求体**（与现有 CMS 接口一致）：

```json
{
  "content_json": {
    "layout": "about",
    "hero": {
      "logo_url": "https://{oss-domain}/cms/about-logo.png",
      "brand_name": "追煮跑",
      "slogan": "专注宠物鲜食 · 人食级原料 · 冷链直达",
      "slogan_sub": "让每一只毛孩子吃得健康、放心"
    },
    "description": "追煮跑科技有限公司，致力于通过数字化供应链为宠物家庭提供安全、营养、便捷的鲜食解决方案。我们在全国核心城市建立冷链仓储，确保每一份鲜食新鲜送达。",
    "tags": ["宠物鲜食", "冷链配送", "AI 喂养建议"],
    "contacts": {
      "phone": { "label": "客服热线", "value": "400-888-6688" },
      "email": { "label": "商务邮箱", "value": "service@zzp.com" },
      "wechat": { "label": "微信服务号", "value": "追煮跑官方" },
      "address": { "label": "公司地址", "value": "上海市浦东新区" }
    }
  }
}
```

> **说明**：`contacts.*.label` 由 Web 端保存时 **自动写入固定文案**，表单不提供 label 编辑，避免与 C 端 UI 漂移。商户仅编辑 **value** 字段。

### 4.2a Hero · Logo 图片规范

| 项 | 规范 |
|----|------|
| 存储 | `hero.logo_url` 存 **HTTPS 绝对 URL**（与商品图、首页封面一致） |
| 上传接口 | 复用 `POST /admin/storage/upload`（Admin JWT） |
| 建议 `category` | **`cms`**（若后端尚未支持，联调前扩展白名单；临时可沿用 `product` 仅作 PoC，**上线前须改为 `cms`**） |
| 文件格式 | PNG / JPG / WebP |
| 建议尺寸 | **正方形 288×288 px**（C 端展示约 144rpx，2× 图）；最小 144×144 |
| 大小上限 | ≤ **500 KB**（与 `ImageUpload` 常见限制对齐） |
| 透明底 | 允许；C 端 Logo 容器为 **白底圆角**（与现 Emoji 容器一致） |
| 清空 | 表单提供「恢复默认图标」→ `logo_url: null`；C 端与 Web 预览均回退 Emoji `🐾` |
| 域名校验 | URL 须落在小程序 **downloadFile 合法域名**（与现有 OSS 域名一致即可） |

**Web 上传组件**：复用 `ImageUpload.vue`（FE-204），单图模式；上传成功后写入 `form.hero.logo_url`，预览即时刷新。

### 4.2b Hero · 文案字段校验（Web 表单）

| 字段 | 必填 | 长度 | 说明 |
|------|------|------|------|
| `brand_name` | 是 | 1~16 字 | 品牌主标题 |
| `slogan` | 是 | 1~40 字 | 允许 `·`、空格；单行展示 |
| `slogan_sub` | 是 | 1~40 字 | 第二行小字；可与主 slogan 换行 |
| `logo_url` | 否 | URL | 空则默认 Emoji |

### 4.3 与旧 Schema 的兼容

| 场景 | Web 端处理 |
|------|------------|
| GET 返回旧版 `sections[]` 法律文档结构 | 识别无 `layout: 'about'` 时，**整页替换为 §4.4 默认模板** 填入表单（可 Toast 提示「已切换为新版关于我们格式，请核对后保存」） |
| GET 返回部分字段缺失 | 用 §4.4 默认值补全 `hero`、`contacts` 四项与 `tags` |
| 仅有 v2 初版 JSON（无 `hero` 块） | `normalizeAboutContent` 从 C 端常量补全 `hero` 默认值 |
| 保存 | 始终写入完整 §4.2 结构，不再写 `sections` |

### 4.4 默认值（与 C 端 `ABOUT_INFO` / 模板对齐）

| 字段 | 默认值 |
|------|--------|
| `hero.logo_url` | `null`（C 端 / 预览显示 Emoji `🐾`） |
| `hero.brand_name` | `追煮跑` |
| `hero.slogan` | `专注宠物鲜食 · 人食级原料 · 冷链直达` |
| `hero.slogan_sub` | `让每一只毛孩子吃得健康、放心` |
| `description` | 见 C 端 `ABOUT_INFO.description` |
| `tags` | `["宠物鲜食", "冷链配送", "AI 喂养建议"]` |
| `contacts.phone.value` | `400-888-6688` |
| `contacts.email.value` | `service@zzp.com` |
| `contacts.wechat.value` | `追煮跑官方` |
| `contacts.address.value` | `上海市浦东新区` |

法律区 / 页脚版本文案：预览组件内 **hardcode** 与 C 端一致，不进入 `content_json`。

---

## 5. API 与配置关系

### 5.1 Admin API（已有）

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/admin/cms/pages` | 列表摘要；`about` 行展示 `updated_at` |
| GET | `/admin/cms/pages/about` | 读取 `content_json` |
| PUT | `/admin/cms/pages/about` | 写入 `content_json`；**不需要** `confirmed: true` |

错误码沿用后端 §5.19：`40425` 页面不存在 · `40001` 参数校验失败。

### 5.2 C 端公开 API（Web 不调用，供联调对照）

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/cms/pages/about` | 返回同上 `content_json`；C 端对接后用于渲染 §4.2 字段 |

### 5.3 与「店铺配置」`/shop-config` 的关系

| 配置项 | 店铺配置 `shop_config` | 关于我们 CMS |
|--------|------------------------|--------------|
| 客服电话 | `contact_phone`（在线客服等场景） | `contacts.phone.value`（关于我们页展示） |
| 地址 | `shop_address` | `contacts.address.value` |

**本期不做双向同步**。商户可能在两处填写相同信息；Web 关于我们编辑页 **无需**跳转店铺配置，仅在帮助文案中注明：「此处仅影响小程序关于我们页；在线客服/首页等仍读店铺配置。」

### 5.4 对象存储上传（Hero Logo）

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/admin/storage/upload` | multipart：`file` + `category=cms`（推荐）；响应 `data.url` 写入 `hero.logo_url` |

与店铺封面（`home_cover_url`）、商品图共用上传链路；失败时 Toast 后端 `message`（如 `50002` 存储未配置）。详见 [项目开发说明-Web前端.md](../pet-app-admin-web/docs/项目开发说明-Web前端.md) §5.4 · `ImageUpload.vue`。

---

## 6. Web 页面功能设计

### 6.1 入口与布局

| 项 | 设计 |
|----|------|
| 路由 | 现有 `/cms`，子 Tab **关于我们**（`type=about`） |
| 布局 | 复用 `EditorPreviewLayout`（≥1280px 左编辑右预览；窄屏 Tab「编辑 \| 预览」） |
| 免责 | 预览区顶部保留 `PreviewDisclaimer`：「页面预览仅供参考」 |

### 6.2 编辑器：`AboutCmsEditor.vue`（新建）

当 `activeType === 'about'` 时，**替换**现有 `CmsContentEditor` 法律文档表单。  
表单分 **四个区块**，自上而下与 C 端视觉顺序一致。

#### 区块 A · 品牌 Hero（页顶）

| 表单项 | 组件 | 校验 / 交互 |
|--------|------|-------------|
| Logo 图标 | `ImageUpload` 单图 + 缩略图预览 | 可选；见 §4.2a；「恢复默认图标」清空 `logo_url` |
| 品牌名称 | `el-input` · `maxlength=16` · `show-word-limit` | 必填；1~16 字 |
| 主 slogan | `el-input` · `maxlength=40` · `show-word-limit` | 必填；1~40 字 |
| 副 slogan | `el-input` · `maxlength=40` · `show-word-limit` | 必填；1~40 字 |

**交互**：

- Logo 上传中禁用保存按钮（`ImageUpload` loading 态）。  
- 上传失败保留原 URL，Toast 错误信息。  
- 未上传 Logo 时，表单区与预览区均展示 **默认 Emoji 占位**（与 C 端一致）。  
- 区块标题：**品牌头图**；副标题说明：「对应小程序关于我们页顶部绿色区域」。

#### 区块 B · 公司信息

| 表单项 | 组件 | 校验 |
|--------|------|------|
| 公司介绍 | `el-input` `type=textarea` · `rows=4` · `maxlength=500` · `show-word-limit` | 必填；5~500 字 |
| 标签 | 动态列表（`el-tag` + 输入框 + 添加按钮） | 1~6 个；每项 1~12 字；去重；禁止空 tag |

**交互**：

- 默认展示 3 个 tag（来自接口或默认值）。  
- 「添加标签」在 `< 6` 个时可点；每个 tag 可删除（至少保留 1 个）。  
- 支持 Enter 添加；重复 tag 提示「标签已存在」。

#### 区块 C · 联系我们

四项 **固定顺序** 表单项（仅 value 可编辑）：

| 显示名（只读 label） | 字段 | 组件 | 校验 |
|----------------------|------|------|------|
| 客服热线 | `contacts.phone.value` | `el-input` | 必填；`/^[\d\-]{5,20}$/`（支持 400 电话） |
| 商务邮箱 | `contacts.email.value` | `el-input` | 必填；邮箱格式 |
| 微信服务号 | `contacts.wechat.value` | `el-input` | 必填；1~32 字 |
| 公司地址 | `contacts.address.value` | `el-input` `type=textarea` · `rows=2` | 必填；2~120 字 |

#### 区块 D · 只读说明（`el-alert` info）

> 法律与资质链接、版本备案信息由小程序固定展示；Hero 区 **绿色渐变背景** 不可在线修改。如需修改商家资质正文请编辑 **「商家资质」** CMS 页。

#### 底部操作

| 按钮 | 行为 |
|------|------|
| 保存 | 校验通过 → `PUT /admin/cms/pages/about` → `showSuccess('保存成功')` |
| 重置 | 恢复为进入页面时 GET 的快照（需二次确认） |

**不需要**协议类二次确认弹窗。

### 6.3 预览：`CmsPreviewPanel.vue` · `about` 分支改造

在现有 `about` 分支上 **按 §3.1 结构重绘**（可参考 C 端 `info/index.vue` 样式，缩小为 375px 卡片宽度）：

| 区域 | 数据来源 |
|------|----------|
| Hero · Logo | `hero.logo_url` 有值 → `<img>`  fit 白底圆角容器；否则 Emoji `🐾` |
| Hero · 品牌名 | `hero.brand_name` |
| Hero · 双行小字 | `hero.slogan` · `hero.slogan_sub` |
| Hero · 背景 | CSS 固定渐变（与 C 端 `$brand` 绿一致，不读表单） |
| 公司信息 | 表单 `description` + `tags` |
| 联系我们 | 表单四项；**行标签必须使用 §3.3 新文案** |
| 法律与资质 | 固定三行；第三项 label **商家资质** |
| 页脚 | 固定 `追煮跑小程序 v1.0.0` · 版权占位 |

预览应 **实时响应** 表单 `v-model`（与资讯/商品预览相同模式）。

### 6.4 类型与工具

| 文件 | 变更 |
|------|------|
| `types/cms.ts` | 增加 `CmsAboutContentJson` · 类型守卫 `isAboutContentJson` |
| `utils/cmsAbout.ts`（新建） | `normalizeAboutContent(raw)` · `buildAboutPayload(form)` · `ABOUT_CMS_DEFAULTS`；兼容缺 `hero` 的旧 JSON |
| `utils/form-rules.ts` | 增加 `aboutHeroRules` · `aboutDescriptionRules` · `aboutTagRules` · `aboutContactRules` |
| `api/cms.ts` | 泛型或 overload：`getCmsPage('about')` 返回类型收窄 |

### 6.5 `views/cms/Index.vue` 集成要点

```text
activeType === 'about'
  ├─ 左侧：AboutCmsEditor
  └─ 右侧：CmsPreviewPanel type="about" :about-content="form"

activeType ∈ { user, privacy, merchant }
  └─ 保持现有 CmsContentEditor + 预览逻辑不变
```

加载逻辑：

1. 切换至「关于我们」→ `GET /admin/cms/pages/about`。  
2. `normalizeAboutContent(content_json)` → 填入 `form`。  
3. 预览与表单共享同一 reactive 对象。

---

## 7. 跨端协同（非 Web 任务，但联调依赖）

### 7.1 C 端（`pet-app`）后续改动摘要

供 Web 团队知悉验收闭环条件；**不在本仓库 Web 任务内**。

| 项 | 说明 |
|----|------|
| 文案 | `ABOUT_INFO.contacts[wechat].label` → `微信服务号`；`legalLinks[merchant].label` → `商家资质` |
| Hero 渲染 | 读取 `content_json.hero`：`logo_url` → `<image mode="aspectFit">`；`brand_name` / `slogan` / `slogan_sub` 替换模板硬编码与 `ABOUT_INFO` 对应字段 |
| Hero 回退 | API 失败或 `logo_url` 为空：Logo 用 Emoji `🐾`；文案回退 `ABOUT_INFO` 默认值 |
| 数据合并 | `fetchInfoPage('about')` 或独立 `fetchAboutInfo()` 解析 §4.2 全量字段，合并到页面 `aboutInfo`；API 失败回退 `ABOUT_INFO` |
| 验收 | 商户在 Web 修改 Hero / 公司介绍 / 标签 / 电话后，小程序关于我们页 **刷新即生效** |

### 7.2 后端（`pet-app-backend`）建议

| 项 | 说明 |
|----|------|
| Seed | `cms_page` type=`about` 的 `content_json` 更新为 §4.2 默认（含 `hero`） |
| 上传 | `POST /storage/upload` 的 `category` 白名单增加 **`cms`**（若尚未支持） |
| 校验 | `PUT` 时对 `layout=about` 做 JSON Schema 校验（可选，与 Web 校验规则一致） |
| 响应 | `GET /cms/pages/about` 直接返回新结构；旧客户端 ignore 未知字段 |

---

## 8. 任务拆分建议（Web FE）

| ID | 任务 | 优先级 | 预估 |
|----|------|--------|------|
| FE-ABT-01 | `types/cms.ts` + `utils/cmsAbout.ts` + 默认值/归一化 | P0 | 0.5d |
| FE-ABT-02 | 新建 `AboutCmsEditor.vue`（含 Hero 上传 + 文案 + 公司/联系表单） | P0 | 1.5d |
| FE-ABT-03 | `CmsPreviewPanel` about 分支重写（Hero 读表单 + 文案修正） | P0 | 1d |
| FE-ABT-04 | `views/cms/Index.vue` 分支集成 + 保存/重置 | P0 | 0.5d |
| FE-ABT-05 | 旧 `content_json` 迁移提示 + QA 对照 C 端截图 | P1 | 0.5d |
| FE-ABT-06 | Hero Logo 上传联调（`category=cms` · 清空回默认 · 预览 `@error` 回退 Emoji） | P0 | 0.5d |

**建议分支名**：`feature/cms-about-editor`（`pet-app-admin-web` · `yjm`）  
**合计预估**：约 **4 人日**（含 Hero 上传联调 FE-ABT-06）

---

## 9. 验收标准

### 9.1 功能验收

| # | 步骤 | 期望 |
|---|------|------|
| T1 | 进入 CMS → 关于我们 | 左侧为 **品牌 Hero + 公司信息 + 联系我们** 表单，**无**章节/目录编辑器 |
| T2 | 修改 Hero 品牌名与双行 slogan → 保存 → 刷新 | 数据持久化；预览 Hero 文案与表单一致 |
| T3 | 上传 Logo → 保存 → 刷新 | `hero.logo_url` 持久化；预览与表单显示上传图 |
| T4 | 点击「恢复默认图标」→ 保存 | `logo_url` 为 null；预览与 C 端均为 Emoji `🐾` |
| T5 | 修改公司介绍与 tags → 保存 → 刷新 | 数据持久化；预览与表单一致 |
| T6 | 修改四项联系方式 → 保存 | 校验非法邮箱/空 tag 拦截；合法数据保存成功 |
| T7 | 预览区检查文案 | 联系行显示 **微信服务号**；法律区第三项 **商家资质** |
| T8 | 切换至用户协议/隐私/商家资质 | 仍为原法律文档编辑器，无回归 |
| T9 | 窄屏 (<1280px) | 编辑/预览 Tab 切换正常 |

### 9.2 接口验收

| # | 检查 |
|---|------|
| A1 | `PUT` body 含完整 `hero` + §4.2 结构，无 `sections` |
| A2 | `GET /cms/pages/about`（C 端接口）与 Admin GET 内容一致 |
| A3 | 保存 **不需要** `confirmed: true` |

### 9.3 视觉对照

- 使用微信开发者工具打开 C 端关于我们页截图，与 Web 预览并排对比：**Hero Logo 容器、品牌名、slogan 双行**、分区顺序、标签样式、联系行布局 **一致**（允许 PC 预览缩放差异）。

---

## 10. 修订记录

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.1 | 2026-06-14 | **商户增量**：Hero 区可配置 — `hero.logo_url`（ImageUpload）、`brand_name`、`slogan`、`slogan_sub`；§3.4 · §4.2a/b · §5.4 · §6.2 区块 A · 验收 T2~T4 · FE-ABT-06 |
| v1.0 | 2026-06-14 | 首版：对齐 C 端卡片布局；定义 about 专用 `content_json`；Web 编辑器与预览改造范围；跨端协同说明 |