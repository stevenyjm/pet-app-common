# 跨项目协作规范母版

> 本文件是三端 `cross-project.md` 的共有部分母版。
> 修改时先更新此文件，再同步到各项目的 `.trae/rules/cross-project.md`。
> 各项目 `cross-project.md` 仅保留"当前项目标识"、"读写权限"、"读取路径"等差异部分。

## 项目结构

本项目位于 `宠物生鲜电商开发项目` 父目录下，包含以下兄弟项目：

```
宠物生鲜电商开发项目/
├── pet-app/              # 微信小程序前端（uni-app + Vue3）
├── pet-app-admin-web/    # Web 管理端（Vue3 + Element Plus）
├── pet-app-backend/      # 后端服务（Node.js + Express + Sequelize）
└── pet-app-common/       # 公共资源目录（独立 git 管理）
    ├── docs/
    │   ├── requirements/     # 跨项目需求文档
    │   ├── api-specs/        # API 接口规范
    │   ├── shared-rules/     # 公共规范母版
    │   └── shared-resources/ # 共享资源文档
    └── resources/
        ├── images/           # 共享图片资源
        └── templates/        # 模板文件
```

## 公共目录操作规范

所有跨项目共享的文档、资源应存放在 `pet-app-common/` 目录：

- **需求文档**：新建跨项目需求时，在 `pet-app-common/docs/requirements/` 创建文档
- **API 规范**：接口变更需同步更新 `pet-app-common/docs/api-specs/`
- **公共规范母版**：跨项目规则母版存放在 `pet-app-common/docs/shared-rules/`
- **共享资源**：通用图片、模板等存放在 `pet-app-common/resources/`

## 公共目录文档命名规范

存放于 `pet-app-common/docs/` 下（含 `requirements/`、`api-specs/`、`shared-rules/`、`shared-resources/`）的所有文档，**文件名按 "时间-文档类型-内容概要" 格式命名**。

### 命名格式

```
YYYY-MM-DD-文档类型-内容概要.md
```

### 文档类型枚举

| 文档类型 | 用途 | 示例 |
|----------|------|------|
| 需求 | 需求分析、功能方案 | `2026-08-18-需求-商户账号体系完善.md` |
| API规范 | API 接口规范 | `2026-08-18-API规范-退款回调.md` |
| 修复方案 | Bug 修复方案与验收 | `2026-08-18-修复方案-退款重复提交.md` |
| 任务清单 | 任务列表与拆解 | `2026-08-18-任务清单-命名规范收敛.md` |
| 分析报告 | 数据库审计、影响分析 | `2026-08-18-分析报告-数据库表结构审计.md` |
| 备案 | 配置备案、决策备案 | `2026-08-18-备案-底部导航静态化.md` |
| 规范 | 规范文档 | `2026-08-18-规范-API字段命名终态.md` |
| 临时 | 临时方案/分析（可后续归档） | `2026-08-18-临时-底部导航图标异常分析.md` |

### 命名注意事项

- 时间使用 ISO 8601 短格式：`YYYY-MM-DD`（北京时间）
- 各段之间使用半角连字符 `-` 分隔
- 内容概要需简明扼要（建议 ≤ 20 字）
- 文件扩展名统一为 `.md`
- **过渡期**：本规则生效前已存在的文档可保留原名；新创建的文档必须按此规范命名

## 数据库表项描述规范

当文档（含 `pet-app-common/docs/` 跨项目文档与各项目 `docs/` 项目文档）涉及对数据库表项的描述时，须遵守以下规则。

### 1. 字段信息表格

文档中以表格形式列举字段信息时，**类型列必须为数据库中实际使用的类型**（如 `BIGINT`、`VARCHAR(32)`、`TINYINT`、`DECIMAL(10,2)`、`DATETIME`、`TEXT`、`JSON` 等），不得使用 JS 类型、TypeScript 类型或其它抽象类型。

若需对类型进行补充说明（如取值范围、枚举映射、默认值含义），应附加在"描述"列中，**不得修改类型列本身**。

示例：

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| order_id | BIGINT | PK, AUTO_INCREMENT | 订单主键 |
| order_status | TINYINT | NOT NULL, DEFAULT 10 | 订单状态：10-待支付, 20-待发货, 30-待收货, 40-已完成, 50-已关闭 |
| pay_amount | DECIMAL(10,2) | NOT NULL, DEFAULT 0.00 | 订单支付金额（元） |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间（北京时间） |

### 2. 建表语句补充

除字段信息表格外，**必须附带该数据库表的建表语句**（`CREATE TABLE`），并在每个字段的 `COMMENT` 中体现字段说明。

```sql
CREATE TABLE `order_main` (
  `order_id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '订单主键',
  `order_status` TINYINT NOT NULL DEFAULT 10 COMMENT '订单状态：10-待支付, 20-待发货, 30-待收货, 40-已完成, 50-已关闭',
  `pay_amount` DECIMAL(10,2) NOT NULL DEFAULT 0.00 COMMENT '订单支付金额（元）',
  `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间（北京时间）',
  PRIMARY KEY (`order_id`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_order_status` (`order_status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单主表';
```

### 3. 适用范围

| 文档类型 | 是否适用 | 说明 |
|----------|----------|------|
| 数据库设计文档 | ✅ 必须 | 表结构设计阶段 |
| 数据库变更文档 | ✅ 必须 | 字段变更、新增表、新增索引 |
| API 规范文档 | ✅ 必须（涉及表时） | 当 API 文档说明响应字段对应表结构时 |
| 需求分析文档 | ✅ 必须（涉及表时） | 当需求文档涉及数据库设计时 |
| 项目开发说明 | ✅ 必须（表结构章节） | 后端项目开发说明中的表结构章节 |
| 开发日志 | ❌ 不强制 | 仅当涉及表结构变更时附带建表语句 |

## 文档同步规则

当修改涉及多个项目的文档时：
1. 在 `pet-app-common/docs/` 相应子目录创建主文档
2. 在各项目的 `docs/` 目录创建引用或摘要链接
3. 保持主文档为权威来源

## Git 管理规则

- `pet-app-common` 独立 git 管理，不参与任何项目的 git 版本管理
- 各项目的 `.gitignore` 已排除 `pet-app-common/`
- 禁止将 `pet-app-common` 的文件提交到项目仓库
- 公共规范母版变更后，需同步更新三端 `cross-project.md` 副本

## 各项目差异部分

### pet-app（微信小程序前端）

**当前项目**：微信小程序前端
**读写权限**：pet-app 读写，其余只读
**读取路径**：
- 后端路由：`../pet-app-backend/routes/`
- 管理端页面：`../pet-app-admin-web/src/views/`

### pet-app-admin-web（Web 管理端）

**当前项目**：Web 管理端
**读写权限**：pet-app-admin-web 读写，其余只读
**读取路径**：
- 后端路由：`../pet-app-backend/routes/`
- 小程序组件：`../pet-app/src/components/`

### pet-app-backend（后端服务）

**当前项目**：后端服务
**读写权限**：pet-app-backend 读写，其余只读
**读取路径**：
- 小程序组件：`../pet-app/src/components/`
- 管理端组件：`../pet-app-admin-web/src/components/`