# 资讯详情接口 content_json 字段命名规范统一方案

> 版本：v1.2  
> 日期：2026-07-26  
> 涉及项目：pet-app-backend（后端）、pet-app-admin-web（管理端Web）、pet-app（小程序）  
> 状态：**✅ 已完成**（后端开发 + 全部验证验收通过）

---

## 一、问题背景

小程序前端反馈：资讯详情页中的「示意图」（`inlineUrl`）未正常显示。经排查，根因是 `article.content_json` 字段在数据库中存在 **snake_case 与 camelCase 混用** 的情况，导致后端读取字段时遗漏了 camelCase 命名的数据。

### 1.1 数据库现状

线上 `article` 表共 6 条记录，经全量扫描 `content_json` 中的字段命名：

| article_id | 顶层字段 | 异常字段（camelCase） |
|---|---|---|
| `banner-1` | `source, date, paragraphs, cta, **inlineUrl**` | ⚠️ `inlineUrl`、`cta.action.productId` |
| `banner-2` | `source, date, paragraphs, cta` | ⚠️ `cta.action.categoryId` |
| `banner-3` | `source, date, paragraphs, external_link, cta` | ✅ 无 |
| `news-1` | `source, date, paragraphs, quote, tail, internal_link, cta` | ⚠️ `internal_link.action.categoryId`、`cta.action.productId` |
| `news-2` | `source, date, paragraphs, quote, internal_link, cta` | ✅ 无 |
| `news-3` | `source, date, paragraphs, quote, internal_link, cta` | ✅ 无 |

> 注：`cta.action.productId`、`cta.action.categoryId` 为业务层面的 Action 参数，**保留 camelCase 是合理的**（在数据种子 `data/articles.js` 中即以此定义），不在本次修复范围内。

**本次需修复的核心字段**：

| 问题字段（camelCase） | 规范字段（snake_case） | 影响记录数 | 影响说明 |
|---|---|---|---|
| `inlineUrl` | `inline_url` | 1 条（banner-1） | 直接导致示意图不显示 |

### 1.2 各端命名习惯

| 端 | 代码位置 | 使用的命名风格 |
|---|---|---|
| **后端模型/迁移** | `models/Article.js`、`scripts/migrate-schema-v2.js` | snake_case ✅ |
| **后端接口输出** | `controllers/articleController.js` → `formatArticleRow()` | snake_case（只读 `content.inline_url`） |
| **后端管理端入库** | `services/adminArticleService.js` → `stringifyContentJson()` | **透传，无转换** ⚠️ |
| **管理端 Web** | `types/article.ts` → `ArticleContent` 接口 | **camelCase** ⚠️ |
| **小程序** | `src/services/news.js` → `normalizeArticle()` | 双兼容（`raw.inline_url ?? raw.inlineUrl`） |

### 1.3 问题链路

```
管理端 Web (camelCase: inlineUrl)
    ↓ POST/PUT
adminArticleService → stringifyContentJson()  ← 透传，无转换
    ↓ JSON.stringify
MySQL article.content_json = {"inlineUrl": "..."}
    ↓ 查询
articleController → formatArticleRow()  ← 只读 content.inline_url (snake_case)
    ↓ 输出 { inline_url: null, ... }
小程序 normalizeArticle()  ← 双兼容，但后端已返回 null
    ↓
detail.vue → v-if="article.inlineUrl" ← false，示意图不渲染
```

---

## 二、解决方案（方案 C）

### 2.1 总体策略

**后端读兼容两种 + 写统一 snake_case + 存量数据修复**

1. **读路径兼容**：`formatArticleRow()` 同时支持 snake_case 和 camelCase 字段读取，对两种命名都做 fallback。
2. **写路径统一**：`adminArticleService` 入库前增加 `normalizeContentJson()` 函数，将 camelCase 字段递归转换为 snake_case，确保入库数据一律规范。
3. **存量数据修复**：编写一次性 SQL 迁移脚本，将现有 `content_json` 中的 `inlineUrl` 转为 `inline_url`。

### 2.2 字段命名规范（统一 snake_case）

`content_json` 结构中，**顶层及嵌套业务字段**统一使用 snake_case：

| 字段（规范 snake_case） | 类型 | 说明 |
|---|---|---|
| `source` | string | 来源 |
| `date` | string | 日期 |
| `paragraphs` | string[] | 正文段落 |
| `quote` | string | 引用/金句 |
| `tail` | string | 尾部文案 |
| `inline_url` | string | 示意图 URL |
| `inline_emoji` | string | 示意图占位 emoji |
| `internal_link` | object | 内链配置 |
| `internal_link.icon` | string | 内链图标 |
| `internal_link.title` | string | 内链标题 |
| `internal_link.desc` | string | 内链描述 |
| `internal_link.action` | object | 内链跳转 Action（**保留原结构**） |
| `external_link` | object | 外链配置 |
| `external_link.title` | string | 外链标题 |
| `external_link.url` | string | 外链 URL |
| `cta` | object | 行动按钮 |
| `cta.text` | string | 按钮文案 |
| `cta.style` | string | 按钮样式 |
| `cta.action` | object | 按钮跳转 Action（**保留原结构**） |

> **注意**：`cta.action`、`internal_link.action` 内的字段（如 `productId`、`categoryId`、`type`）为业务层面的 Action 参数，**保持原有 camelCase 结构不变**，不在本次规范范围内。

---

## 三、各端任务划分

### 3.1 后端（pet-app-backend）

| # | 任务 | 优先级 | 涉及文件 | 说明 | 状态 |
|---|---|---|---|---|---|
| B1 | `formatArticleRow()` 读兼容（C端接口） | 高 | `controllers/articleController.js` | 对 `inline_url`、`inline_emoji`、`internal_link`、`external_link` 增加 camelCase fallback | ✅ 已完成 |
| B2 | 新增 `normalizeContentJson()` / `denormalizeContentJson()` 函数 | 高 | `services/adminArticleService.js` | 单源映射表 + 递归归一化/反归一化 | ✅ 已完成 |
| B3 | `createArticle()` / `updateArticle()` 入库前调用归一化 | 高 | `services/adminArticleService.js` | 在 `stringifyContentJson()` 之前调用 `normalizeContentJson()` | ✅ 已完成 |
| B4 | 存量数据修复脚本 | 中 | `scripts/migrate-schema-v41-fix-article-content-json.js` | 一次性脚本，已执行：banner-1 `inlineUrl` → `inline_url` | ✅ 已完成 |
| B5 | 验证脚本 | 中 | `scripts/_verify-content-json.js` | 已执行：全部 6 条记录验证通过 | ✅ 已完成 |
| B6 | `formatAdminArticle()` 返回管理端时反归一化 | 高 | `services/adminArticleService.js` | 返回 `content` 字段时 snake_case → camelCase | ✅ 已完成 |

### 3.2 管理端 Web（pet-app-admin-web）

| # | 任务 | 优先级 | 涉及文件 | 说明 |
|---|---|---|---|---|
| A1 | 无需修改 | - | - | 后端 B6 已处理反向映射，管理端继续使用 camelCase 读写 `ArticleContent` 接口，完全透明 |

> **说明**：后端通过 `normalizeContentJson()`（写入时 snake_case 化）和 `denormalizeContentJson()`（返回管理端时 camelCase 化）实现双端透明转换，管理端 Web 代码 **无需本次修改**。

### 3.3 小程序（pet-app）

| # | 任务 | 优先级 | 涉及文件 | 说明 |
|---|---|---|---|---|
| P1 | 确认 `normalizeArticle()` 双兼容逻辑 | 已完成 | `src/services/news.js` | 已兼容 `raw.inline_url ?? raw.inlineUrl`，无需修改 |
| P2 | 端到端验证示意图显示 | 高 | `src/subpk-news/detail.vue` | 后端修复部署后，验证 banner-1 的示意图正常显示 |

### 3.4 数据库运维

| # | 任务 | 优先级 | 说明 | 状态 |
|---|---|---|---|---|
| D1 | 执行存量修复脚本 B4 | 中 | 本地=生产共享数据库，已在本地执行修复 | ✅ 已完成 |
| D2 | 数据验证 | 中 | 执行 B5 验证脚本，全部 6 条记录验证通过 | ✅ 已完成 |

---

## 四、详细实施步骤

### 阶段 1：后端代码修改（B1 → B3 → B6）

#### B1：`formatArticleRow()` 读兼容（C端接口）

**文件**：`controllers/articleController.js`  
**修改函数**：`formatArticleRow()`

```js
// 修改前（第 47 行）：
inline_url: content.inline_url || null,

// 修改后：
inline_url: content.inline_url ?? content.inlineUrl ?? null,
// 同时补充 inline_emoji 读取：
inline_emoji: content.inline_emoji ?? content.inlineEmoji ?? null,
// internal_link 兼容：
internal_link: content.internal_link ?? content.internalLink ?? null,
// external_link 兼容：
external_link: content.external_link ?? content.externalLink ?? null,
```

#### B2：新增归一化/反归一化函数（单源映射）

**文件**：`services/adminArticleService.js`  
**设计原则**：使用单一正向映射表，反向映射通过 `invertMap()` 自动派生，避免两份映射表手动维护导致字段漂移。

```js
// 单一源：camelCase → snake_case 映射表
const CONTENT_FIELD_MAP = {
  inlineUrl: 'inline_url',
  inlineEmoji: 'inline_emoji',
  internalLink: 'internal_link',
  externalLink: 'external_link',
};

// 从正向映射派生反向映射（snake_case → camelCase）
function invertMap(map) {
  const result = {};
  for (const [k, v] of Object.entries(map)) {
    result[v] = k;
  }
  return result;
}

const SNAKE_TO_CAMEL_MAP = invertMap(CONTENT_FIELD_MAP);

/**
 * 写路径归一化：camelCase → snake_case
 * 支持对象或字符串输入；字符串会先 parse，归一化后由调用方 stringify
 */
function normalizeContentJson(input) {
  if (input === undefined || input === null) return input;
  
  let obj = input;
  if (typeof obj === 'string') {
    try {
      obj = JSON.parse(obj);
    } catch {
      return obj; // 无法解析则原样返回
    }
  }
  if (typeof obj !== 'object' || obj === null) return obj;
  
  const result = { ...obj };
  for (const [camelKey, snakeKey] of Object.entries(CONTENT_FIELD_MAP)) {
    if (camelKey in result && !(snakeKey in result)) {
      result[snakeKey] = result[camelKey];
      delete result[camelKey];
    }
  }
  return result;
}

/**
 * 读路径反归一化（返回管理端）：snake_case → camelCase
 * 仅处理顶层字段，不递归 action 等嵌套对象
 */
function denormalizeContentJson(content) {
  if (!content || typeof content !== 'object') return content;
  
  const result = { ...content };
  for (const [snakeKey, camelKey] of Object.entries(SNAKE_TO_CAMEL_MAP)) {
    if (snakeKey in result && !(camelKey in result)) {
      result[camelKey] = result[snakeKey];
      delete result[snakeKey];
    }
  }
  return result;
}
```

#### B3：入库前调用归一化

**文件**：`services/adminArticleService.js`  
**修改位置**：

```js
// createArticle() 中（第 129 行）：
// normalizeContentJson 已内置字符串 parse 处理，支持 payload.content_json 为对象或字符串
const contentJson = stringifyContentJson(
  normalizeContentJson(payload.content_json ?? payload.content)
);

// updateArticle() 中（第 173-174 行）：
if (payload.content_json !== undefined || payload.content !== undefined) {
  updates.content_json = stringifyContentJson(
    normalizeContentJson(payload.content_json ?? payload.content)
  );
}
```

#### B6：`formatAdminArticle()` 返回管理端时反归一化

**文件**：`services/adminArticleService.js`  
**修改 `formatAdminArticle()`**：

```js
// 在 includeContent 分支中，对 content 做反归一化（snake_case → camelCase）：
if (includeContent) {
  base.content_json = row.content_json;
  base.content = denormalizeContentJson(content);  // 应用 B2 的反归一化
}
```

> **生效前提**：管理端 `ArticleContentEditor.vue` 使用接口返回的 `content` 字段（已解析对象），而非自行 `JSON.parse(content_json)`。经确认后端 `formatAdminArticle` 返回的 `content` 字段为已解析对象，管理端直接使用该字段，因此 B6 反归一化可正常生效。

### 阶段 2：存量数据修复（B4）

**新建文件**：`scripts/migrate-schema-v41-fix-article-content-json.js`

脚本逻辑：
1. 查询所有 `article` 记录
2. 解析 `content_json`
3. 调用 `normalizeContentJson()` 归一化
4. 写回 `content_json`
5. 输出修复日志

### 阶段 3：验证与部署（B5 + P2 + D2）

#### B5：验证脚本

**新建文件**：`scripts/_verify-content-json.js`

验证逻辑：
1. 查询 `article` 表全量数据
2. 解析每条记录的 `content_json`
3. 检查顶层字段是否包含 camelCase（正则 `/^[a-z]+[A-Z]/`）
4. 输出异常记录及具体的 camelCase 字段名
5. 返回通过/失败状态

**验证流程**：
1. **本地验证**：启动本地后端服务，调用 `GET /api/v1/articles/banner-1`，确认返回 `inline_url` 有值
2. **执行存量修复**：在 Devbox 执行 `node scripts/migrate-schema-v41-fix-article-content-json.js`
3. **数据验证**：执行 `node scripts/_verify-content-json.js`，确认所有 article 顶层字段均为 snake_case
4. **管理端回显验证**：在管理端打开 banner-1 编辑页，确认示意图 URL 正常回显（B6 生效）
5. **小程序验证**：在微信开发者工具中打开 banner-1 详情页，确认示意图显示
6. **管理端新建验证**：管理端新建一条带示意图的文章，保存后查 DB 确认写入 `inline_url`（非 `inlineUrl`）

---

## 五、验证清单

| # | 验证项 | 方法 | 预期结果 | 状态 |
|---|---|---|---|---|
| V1 | C端接口返回 snake_case | `GET /api/v1/articles/banner-1` | `data.inline_url` 有值（URL） | ✅ 已通过 |
| V2 | 存量修复后数据规范 | 执行 B5 脚本 | 所有 article 顶层字段均为 snake_case | ✅ 已通过 |
| V3 | 新写入数据规范 | 管理端新建/编辑文章后查 DB | `content_json` 顶层无 camelCase 字段 | ✅ 已通过 |
| V4 | 小程序示意图显示 | 打开 banner-1 详情页 | 示意图正常渲染 | ✅ 已通过 |
| V5 | 小程序无回归 | 检查 news-1/2/3、banner-2/3 详情页 | 所有页面内容正常显示 | ✅ 已通过 |
| V6 | 收藏状态无回归 | 资讯详情页收藏按钮 | 收藏/取消收藏正常工作 | ✅ 已通过 |
| V7 | 管理端编辑器回显正常 | 管理端打开 banner-1 编辑页 | 示意图 URL 在编辑器中正常显示 | ✅ 已通过 |
| V8 | 管理端新建流程正常 | 管理端新建带示意图的文章 → 保存 → 重新打开编辑 | 示意图 URL 正常保存和回显 | ✅ 已通过 |
| V9 | 管理端预览面板正常 | 管理端文章预览面板 | 示意图预览正常 | ✅ 已通过 |

---

## 六、风险与回滚

| 风险 | 影响 | 应对 |
|---|---|---|
| 归一化函数误改 `cta.action` 等嵌套字段 | 按钮跳转异常 | `normalizeContentJson` 仅处理顶层字段，不递归处理 `action` 对象 |
| 存量修复脚本误操作 | 数据丢失 | 脚本执行前备份 `article` 表；脚本使用 UPSERT 逻辑，幂等安全 |
| 管理端 Web 提交字段变更 | 前端异常 | 后端做了双兼容，管理端可保持 camelCase 提交，后端自动转换 |

**回滚方案**：
- 代码回滚：git revert 本次提交
- 数据回滚：从备份恢复 `article` 表；或重新执行迁移脚本的反向版本

---

## 七、实施依赖顺序

```
阶段 1（后端代码）→ 阶段 2（存量修复）→ 阶段 3（验证部署）
  B1, B2, B3, B6        B4          B5, P2, D2
```

**关键路径**：B1 → B2 → B3 → B6 → B4 → V1 → V7 → P2

**依赖说明**：
- B6（反向映射）依赖 B2 的映射表定义（可共用同一张 MAP 常量）
- B4（存量修复）依赖 B2 的 `normalizeContentJson()` 函数
- V7/V8（管理端验证）依赖 B6 完成
- P2（小程序验证）依赖 B1 + B4 完成

**预计工时**：
- 后端代码修改（B1+B2+B3+B6）：0.5h
- 存量修复脚本编写 + 执行（B4）：0.5h
- 验证测试（B5 + V1~V9）：1.0h

---

## 八、实施记录（2026-07-26）

### 实际完成情况

| # | 任务 | 完成时间 | 涉及文件 | 备注 |
|---|---|---|---|---|
| B1 | `formatArticleRow()` 读兼容 | 2026-07-26 | `controllers/articleController.js` | 新增 `inline_emoji` 字段，4 个字段增加 camelCase fallback |
| B2 | 归一化/反归一化函数 | 2026-07-26 | `services/adminArticleService.js` | 单源 `CONTENT_FIELD_MAP` + `invertMap()` 派生反向映射 |
| B3 | 入库前调用归一化 | 2026-07-26 | `services/adminArticleService.js` | `createArticle()` 和 `updateArticle()` 均已接入 |
| B4 | 存量数据修复 | 2026-07-26 | `scripts/migrate-schema-v41-fix-article-content-json.js` | 修复 `banner-1`：`inlineUrl` → `inline_url` |
| B5 | 验证脚本 + 执行 | 2026-07-26 | `scripts/_verify-content-json.js` | 全部 6 条记录通过 |
| B6 | 反归一化（管理端回显） | 2026-07-26 | `services/adminArticleService.js` | `formatAdminArticle` 返回 `content` 时自动 snake_case → camelCase |

### 数据库变更

```sql
-- 已执行：banner-1 的 content_json 中 inlineUrl → inline_url
UPDATE article 
SET content_json = '{"source":"追煮跑官方","date":"2026-05-22","paragraphs":["..."],"cta":{...},"inline_url":"https://objectstorageapi.hzh.sealos.run/..."}'
WHERE article_id = 'banner-1';
```

### 验证验收记录（V1~V9 全部通过）

| # | 验证项 | 验证时间 | 验证方 | 结果 |
|---|---|---|---|---|
| V1 | C端接口返回 snake_case | 2026-07-26 | 后端 | ✅ `GET /api/v1/articles/banner-1` 返回 `inline_url` 有值 |
| V2 | 存量修复后数据规范 | 2026-07-26 | 后端 | ✅ 全部 6 条 article 顶层字段均为 snake_case |
| V3 | 新写入数据规范 | 2026-07-26 | 管理端 | ✅ 管理端编辑保存后，DB 写入 `inline_url`（snake_case） |
| V4 | 小程序示意图显示 | 2026-07-26 | 小程序 | ✅ banner-1 详情页示意图正常渲染 |
| V5 | 小程序无回归 | 2026-07-26 | 小程序 | ✅ banner-2/3、news-1/2/3 详情页内容正常显示 |
| V6 | 收藏状态无回归 | 2026-07-26 | 小程序 | ✅ 收藏/取消收藏功能正常 |
| V7 | 管理端编辑器回显正常 | 2026-07-26 | 管理端 | ✅ banner-1 编辑页示意图 URL 正常回显（B6 反归一化生效） |
| V8 | 管理端新建流程正常 | 2026-07-26 | 管理端 | ✅ 新建带示意图文章 → 保存 → 重新打开，snake_case 入库 + camelCase 回显正常 |
| V9 | 管理端预览面板正常 | 2026-07-26 | 管理端 | ✅ 文章预览面板示意图预览正常 |

### 剩余事项

> 无。所有任务（B1~B6、D1~D2、V1~V9）均已完成。

---

## 九、最终验证报告

### 9.1 修复概要

| 项 | 内容 |
|---|---|
| **需求来源** | 小程序前端反馈资讯详情页示意图未正常显示 |
| **根因** | `article.content_json` 字段 snake_case 与 camelCase 混用，后端读取遗漏 |
| **修复方案** | 方案 C：后端读兼容 + 写统一 snake_case + 存量数据修复 |
| **修复日期** | 2026-07-26 |
| **涉及项目** | pet-app-backend（后端） |
| **影响范围** | 资讯/轮播模块的 content_json 字段读写 |

### 9.2 交付清单

| # | 交付物 | 类型 | 状态 |
|---|---|---|---|
| 1 | `formatArticleRow()` 读兼容（articleController.js） | 代码修改 | ✅ 已部署 |
| 2 | `normalizeContentJson()` / `denormalizeContentJson()`（adminArticleService.js） | 代码新增 | ✅ 已部署 |
| 3 | `createArticle()` / `updateArticle()` 入库归一化（adminArticleService.js） | 代码修改 | ✅ 已部署 |
| 4 | `formatAdminArticle()` 反归一化（adminArticleService.js） | 代码修改 | ✅ 已部署 |
| 5 | 存量修复脚本 `migrate-schema-v41-fix-article-content-json.js` | 脚本 | ✅ 已执行 |
| 6 | 验证脚本 `_verify-content-json.js` | 脚本 | ✅ 已执行通过 |
| 7 | `article.content_json` 存量数据修复（banner-1 `inlineUrl` → `inline_url`） | 数据修复 | ✅ 已完成 |
| 8 | 后端说明文档 v1.0.07（含 content_json 命名规范） | 文档 | ✅ 已更新 |
| 9 | 方案文档（v1.2） | 文档 | ✅ 已完成 |

### 9.3 验证结果

| # | 验证项 | 结果 | 证据 |
|---|---|---|---|
| V1 | C端接口返回 snake_case | ✅ | `GET /api/v1/articles/banner-1` 返回 `inline_url` 有值 |
| V2 | 存量修复后数据规范 | ✅ | `_verify-content-json.js` 全部 6 条记录 PASS |
| V3 | 新写入数据规范 | ✅ | 管理端保存后 DB 写入 snake_case |
| V4 | 小程序示意图显示 | ✅ | banner-1 详情页示意图正常渲染 |
| V5 | 小程序无回归 | ✅ | 5 条资讯详情页内容正常 |
| V6 | 收藏状态无回归 | ✅ | 收藏/取消收藏功能正常 |
| V7 | 管理端编辑器回显 | ✅ | 反归一化生效，camelCase 回显正常 |
| V8 | 管理端新建流程 | ✅ | snake_case 入库 + camelCase 回显 |
| V9 | 管理端预览面板 | ✅ | 示意图预览正常 |

### 9.4 纳入项目规范

本次修复确立以下项目规范，纳入 `项目开发说明-后端.md`：

1. **`article.content_json` 字段统一使用 snake_case 命名**
2. 后端提供 `normalizeContentJson()`（写路径 camelCase→snake_case）和 `denormalizeContentJson()`（读路径 snake_case→camelCase）双端透明转换
3. 管理端 Web 可继续使用 camelCase 提交，后端自动归一化
4. C 端接口 `formatArticleRow()` 对 `inline_url`、`inline_emoji`、`internal_link`、`external_link` 四个顶层字段做双命名兼容
5. 新增资讯/迁移脚本需通过 `_verify-content-json.js` 验证字段命名规范

### 9.5 风险与后续

| 项 | 说明 |
|---|---|
| **风险** | 低。修改仅限于 article.content_json 顶层 4 个字段的命名处理，不涉及嵌套 action 对象结构 |
| **回滚** | 代码回滚：git revert `32b90d9`；数据回滚：迁移脚本幂等可重复执行 |
| **后续** | 建议后续 CMS 其他模块（如 `cms_page.content_json`）也采用 snake_case 统一规范 |
