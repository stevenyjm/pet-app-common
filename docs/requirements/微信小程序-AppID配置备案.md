# 微信小程序 - AppID 配置备案

> **文档用途**：记录小程序各环境 AppID / AppSecret 配置，供三端开发与运维同步查阅。  
> **维护责任**：项目管理员 · 变更后更新本文档 · 通知三端开发。  
> **关联文档**：[项目开发说明-小程序前端.md](../pet-app/docs/项目开发说明-小程序前端.md) §5.15 · [项目开发说明-后端.md](../pet-app-backend/docs/项目开发说明-后端.md) §5.18 · [项目开发说明-Web前端.md](../pet-app-admin-web/docs/项目开发说明-Web前端.md) §5.18

---

## 1. 环境配置一览

| 环境 | AppID | AppSecret | 用途 | 配置位置 |
|------|-------|-----------|------|----------|
| **开发环境** | `wx1234567890abcdef` | `dev_secret_1234567890` | 微信开发者工具调试、本地开发 | 小程序 `project.config.json`、后端 `.env.local`、Web `.env.local` |
| **测试环境（Staging）** | `wx0987654321fedcba` | `staging_secret_0987654321` | 预发布测试、CI/CD 自动测试 | 小程序 `project.staging.config.json`、后端 `.env.staging`、Web `.env.staging` |
| **生产环境** | `wxabcdef1234567890` | `prod_secret_abcdef123456` | 正式上线 | 小程序 `project.prod.config.json`、后端 `.env.production`、Web `.env.production` |

---

## 2. 三端配置说明

### 2.1 小程序前端配置

**文件路径**：`pet-app/project.config.json`

```json
{
  "appid": "wx1234567890abcdef",
  "projectname": "pet-app",
  "setting": {
    "urlCheck": false,
    "es6": true,
    "minified": true
  }
}
```

**说明**：
- 开发环境使用 `project.config.json`
- 构建测试环境时使用 `project.staging.config.json`
- 构建生产环境时使用 `project.prod.config.json`

### 2.2 后端配置

**文件路径**：`pet-app-backend/.env.local`（开发）

```env
WECHAT_APP_ID=wx1234567890abcdef
WECHAT_APP_SECRET=dev_secret_1234567890
```

**说明**：
- 后端通过 `WECHAT_APP_ID` 和 `WECHAT_APP_SECRET` 进行微信登录验证
- 不同环境使用不同的 `.env.*` 文件

### 2.3 Web 管理端配置

**文件路径**：`pet-app-admin-web/.env.local`（开发）

```env
VITE_WECHAT_APP_ID=wx1234567890abcdef
```

**说明**：
- Web 端主要用于前端展示和跳转小程序
- 配置项作为环境变量注入

---

## 3. 微信公众平台配置

### 3.1 服务器域名配置

在微信公众平台「开发管理 → 开发设置」中配置：

| 类型 | 域名 | 说明 |
|------|------|------|
| **request 合法域名** | `api.pet-fresh.com` | 后端 API 接口域名 |
| **socket 合法域名** | `ws.pet-fresh.com` | WebSocket 域名（如使用） |
| **uploadFile 合法域名** | `oss.pet-fresh.com` | 文件上传域名 |
| **downloadFile 合法域名** | `oss.pet-fresh.com` | 文件下载域名 |

### 3.2 业务域名配置

| 类型 | 域名 | 说明 |
|------|------|------|
| **业务域名** | `pet-fresh.com` | 小程序内跳转的业务网页域名 |

---

## 4. 配置变更流程

1. **申请新配置**：由项目管理员在微信公众平台申请或修改配置
2. **更新文档**：修改本文档，记录新的 AppID / AppSecret
3. **同步三端**：通知小程序前端、后端、Web 管理端开发人员更新配置文件
4. **测试验证**：在对应环境进行功能测试，确保微信登录等功能正常
5. **记录变更**：在本文档末尾添加变更记录

---

## 5. 注意事项

1. **AppSecret 保密**：AppSecret 属于敏感信息，禁止提交到代码仓库
2. **环境隔离**：不同环境使用不同的 AppID，避免开发环境影响生产数据
3. **配置同步**：三端配置必须保持一致，否则会导致微信登录等功能异常
4. **域名备案**：所有域名必须完成 ICP 备案，否则无法在生产环境使用
5. **定期检查**：定期检查配置是否过期或被修改

---

## 6. 变更记录

| 版本 | 日期 | 变更内容 |
|------|------|----------|
| v1.0 | 2026-06-15 | 初始版本，记录开发、测试、生产环境配置 |