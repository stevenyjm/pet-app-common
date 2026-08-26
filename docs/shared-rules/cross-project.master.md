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

## 数据库基础字段与逻辑删除规范

> **生效日期**：2026-08-24（v2.0，基于数据库表重构与数据服务优化需求修订）
> **适用范围**：本项目（宠物生鲜电商）下所有数据库表，含既有表与新增表。
> **父级规则**：`../multi-project.md` §8（同源母版，本节为副本，差异以父级为准）。
> **后端实现侧**：`pet-app-backend/.trae/rules/naming-conventions.md` §2.8。
> **需求来源**：`pet-app-common/docs/requirements/2026-08-24-需求-数据库表重构与数据服务优化.md`

### 1. 必备字段

任何数据库表（包括字典表、关联表、日志表、配置表）都必须包含以下四类字段：

| 字段类别 | 字段名（强制） | 类型与约束 | 说明 |
|----------|---------------|------------|------|
| 主键 | 见 §2 | 业务主键（`VARCHAR(n)`），禁止 `BIGINT AUTO_INCREMENT` 逻辑主键 | 每张表必须有主键 |
| 创建时间 | `create_time` | `DATETIME NOT NULL`（无 `DEFAULT CURRENT_TIMESTAMP`，由后端主动推送） | 记录创建时间（北京时间，格式 `yyyy-MM-dd HH:mm:ss`） |
| 最后修改时间 | `last_modified_time` | `DATETIME NOT NULL`（无 `DEFAULT CURRENT_TIMESTAMP`、无 `ON UPDATE CURRENT_TIMESTAMP`，由后端主动推送） | 记录最后修改时间（北京时间，格式 `yyyy-MM-dd HH:mm:ss`） |
| 逻辑删除标记 | `is_deleted` | `TINYINT NOT NULL DEFAULT 0` | `0`-未删除（默认）；`1`-已逻辑删除 |

> 禁止使用 `created_at` / `updated_at` / `create_time` / `update_time`（除 `create_time` / `last_modified_time` 外）/ `deleted` / `deleted_at` / `is_hidden`（用于软删语义）等替代命名；既有字段需在迁移期收敛为上述标准命名。
> **时间字段不由数据库自动创建和更新**：建表语句中 `create_time` 与 `last_modified_time` 不得使用 `DEFAULT CURRENT_TIMESTAMP` 与 `ON UPDATE CURRENT_TIMESTAMP`；时间字段的写入与更新由后端服务在业务逻辑中主动推送（创建时写入 `create_time`，更新时写入 `last_modified_time`）。

### 2. 主键规则

- **强制业务主键**：所有表（含既有表与新增表）必须使用具有明确业务含义的字段作为主键，禁止使用 `BIGINT AUTO_INCREMENT` 逻辑主键。
- 主键类型为 `VARCHAR(n)`，长度根据实际业务编码规则预留余量。
- 业务编码生成规则（由用户指定）：
  - 用户编码 `user_code`：格式 `U00000001`（前缀 `U` + 8 位数字），字段类型建议 `VARCHAR(16)`。
  - 宠物编码 `pet_code`：格式 `UP0001`（前缀 `UP` + 4 位数字），字段类型建议 `VARCHAR(16)`。
  - 订单编码 `order_code`：格式 `yyyyMMdd` + 4 位流水号（如 `202608250001`），字段类型建议 `VARCHAR(20)`。
  - 其他实体业务主键格式：**在实际确定数据库表字段时，由用户询问决策**（包括字段长度预留、并发安全方案等）。
- 联合主键仅用于纯关联表（如 `inventory_real_time`），命名仍遵循业务编码风格。
- 禁止无主键表。
- **既有 102 张表主键改造**：既有使用 `BIGINT AUTO_INCREMENT` 逻辑主键的表，需全量改造为业务主键；改造需按业务模块分批推进，每批配套数据迁移脚本（老 ID → 新 code 映射与回填）、API 兼容方案（双写过渡期）、回滚预案。

### 3. 逻辑删除规则

1. **禁止硬删除**：所有表原则上不允许使用 `DELETE FROM ...` 物理删除；删除操作必须更新 `is_deleted = 1`。
2. **查询过滤**：所有读操作（除非有明确审计/恢复场景）必须带 `is_deleted = 0` 过滤条件。
3. **业务字段共存**：业务侧的"隐藏"（如 `is_hidden`）、"用户删除"（如 `user_deleted_at`）、"状态软删"（如 `status=0`）等业务字段保留，但**不能替代** `is_deleted` 字段；二者语义不同，并存使用。
4. **历史 `deleted_at` 字段**：既有表已存在的 `deleted_at`（记录删除时间戳）可保留作为补充信息，但逻辑删除判定以 `is_deleted` 为准。
5. **`status` 字段不可替代 `is_deleted`**：`status` 表示业务状态枚举（如订单状态、上下架），不参与逻辑删除判定。
6. **唯一索引兼容**：含唯一索引的表若需支持"删除后重建"，应在唯一索引中加入 `is_deleted` 或对删除记录做唯一键扰动（如拼接 `_del_{id}`）。
7. **`is_deleted` 不作单列索引**：禁止对 `is_deleted` 字段单独建索引（TINYINT 选择性低，单列索引无意义）；既有 `idx_is_deleted` 单列索引需在迁移期移除。如需提升"过滤未删除记录"查询性能，应改用联合索引（如 `(is_deleted, ...高频查询字段)`）。

### 4. 字段命名规则

1. **snake_case 统一**：数据表名、字段名、API 路径参数、请求/响应字段统一使用 `snake_case`。
2. **字段表源前缀（仅易混淆字段）**：仅在联表查询时无法区分表源与语义的字段需加表源前缀：
   - `status` → `{entity}_status`（如 `pet_status`、`order_status`）
   - `name` → `{entity}_name`（如 `pet_name`、`product_name`）
   - 其他易混淆字段（如 `amount`、`quantity`、`reason`、`description`）按需加前缀
   - **通用字段保留原名**：`phone`、`email`、`avatar_url`、`created_at`/`create_time` 等通用字段不加表源前缀
3. **时间字段命名**：统一使用 `create_time` / `last_modified_time`，由后端主动推送（见 §1）。

### 5. CONSTRAINT 约束规则

- **禁止使用 CONSTRAINT 约束**：数据表中不进行 `CONSTRAINT` 约束（含外键约束 `FOREIGN KEY`、检查约束 `CHECK` 等），该约束迁移至后端服务进行实现。
- **既有 CONSTRAINT 移除**：既有 80+ 处外键约束需全量移除；移除前必须先在后端服务补充引用完整性校验（如 `order_main.user_code` 必须存在于 `user_account`）、级联删除/更新逻辑（如删除 `pet` 时先处理 `pet_owner` 等子表）。
- 移除需按业务模块分批推进，每批配套后端引用完整性校验实现。

### 6. TINYINT 索引策略

- **通常不对 TINYINT 类型字段作索引**：TINYINT 选择性低，单列索引意义不大。
- 例外：联合索引中包含 TINYINT（如 `(user_code, order_status)`）是允许的，TINYINT 不应作为联合索引首列。
- `is_deleted` 单列索引禁止（见 §3.7）。

### 7. COMMENT 规范

- **建表语句必须补充 COMMENT**：表级 `COMMENT` 与每个字段的 `COMMENT` 均不可省略。
- 字段 `COMMENT` 需体现字段语义；枚举型字段需在 `COMMENT` 中标注取值映射（如 `'订单状态：10-待支付, 20-待发货'`）。
- **时间字段 COMMENT**：`create_time` / `last_modified_time` 及其他业务时间字段（如 `pay_time`、`shipped_at`、`expire_at` 等）的 `COMMENT` 须包含备注"北京时间(UTC+8)，格式 yyyy-MM-dd HH:mm:ss"。

### 8. TEXT 与 JSON 类型规范

- **避免使用 TEXT 类型**：数据库字段尽量不使用 `TEXT`（含 `LONGTEXT`/`MEDIUMTEXT`/`TINYTEXT`）类型；短文本改用 `VARCHAR(n)`，富文本/快照类评估子表或外部存储。
- **避免使用 JSON 类型**：数据库字段尽量不使用 `JSON` 类型，亦尽量不使用"以 JSON 为内容格式的字符串"（如 `VARCHAR`/`TEXT` 存储 JSON 文本，命名常为 `*_json`、`content_json`、`extra_json`、`config_json` 等）；数组类改子表，结构化对象改扁平字段或子表。
- **JSON 类型受控例外（数量限制 + 简单结构）**：在"具有明确数量限制"且"结构简单"的前提下，允许使用 `JSON` 类型存储以下场景数据（后端须在写入前校验数量与结构约束，字段 `COMMENT` 须标注结构说明与数量上限）：
  - **简短字符串数组**：如标签 `['标签1','标签2','标签3']`（单元素字数上限 6 字，元素数量上限 5 个）
  - **图像 URL 数组**：如 `['url1','url2']`（元素数量上限 6 张）
  - **键值对对象数组**：如 `[{name:'a',count:1},{name:'b',count:3}]`（每个对象键值对数量上限 2 个，对象数量上限 3 个）
  - **键值对对象**：如 `{'value1':1,'value2':2,'value3':3}`（键值对数量上限 6 个）
  > 不满足"数量限制 + 简单结构"的结构化数据仍须改子表或扁平字段。
- **分表确认机制**：若因避免使用 `TEXT`/`JSON` 导致单表字段过多或单行数据过大而需要分表，**必须先向用户询问确认分表方案**，不得擅自拆分。
- **既有 TEXT/JSON 改造**：约 67 处字段需评估替代方案，分批改造；**该规范下的相关需求变更涉及范围大，必须由用户进行评估、评审、决策后进行**。
- **豁免候选**：审计/流水表的动态详情字段（如 `admin_audit_log.detail_json`、`cron_task_execution_log.result_summary`）、微信响应快照字段（如 `refund_record.extra_json`）建议豁免（结构由第三方决定，扁平化成本高）。

### 9. 既有表迁移策略

- 既有表缺少上述字段或不符上述规则时，需通过迁移脚本补齐/改造，不得直接重建表。
- 命名不符（如 `created_at` → `create_time`、`updated_at` → `last_modified_time`）的迁移需在 `pet-app-common/docs/api-specs/` 备案变更说明。
- 迁移完成后需同步更新 `scripts/schema.sql`、Sequelize Model 定义、`scripts/verify-db.js` 校验项。
- **既有表全量改造范围**：102 张表主键改造、95 张表时间字段命名与维护方式改造、60+ 字段表源前缀改造、50+ 张表 CONSTRAINT 移除、约 23 处 COMMENT 补齐、约 67 处 TEXT/JSON 改造、14+ 张表 `is_deleted` 单列索引移除。

### 10. 例外场景

仅以下场景可豁免 `is_deleted` 字段（仍需主键 + `create_time` + `last_modified_time`）：

- **Token 表**（如 `user_refresh_token`、`admin_refresh_token`）：通过 `revoked_at` 实现撤销语义，删除走物理清理。
- **审计/流水表**（如 `admin_audit_log`、`inventory_flow`、`payment_log`、`user_login_log`、`user_email_send_log`、`user_browse_log`、`user_sms_log`）：仅追加不删除。
- **会话消息表**（如 `service_message`、`user_message`）：通过业务状态或归档机制管理。

> 任何豁免需在表 `COMMENT` 或建表语句旁注释说明豁免理由；新增表若主张豁免，需在 PR 评审中明确批准。

### 11. 检查清单

新增/变更表结构时逐项核对：

- [ ] 表包含业务主键（`{entity}_code` 等，禁止 `BIGINT AUTO_INCREMENT`）
- [ ] 表包含 `create_time DATETIME NOT NULL`（无 `DEFAULT CURRENT_TIMESTAMP`，由后端推送）
- [ ] 表包含 `last_modified_time DATETIME NOT NULL`（无 `DEFAULT`/`ON UPDATE`，由后端推送）
- [ ] 表包含 `is_deleted TINYINT NOT NULL DEFAULT 0`
- [ ] 表包含表级 `COMMENT` 与每个字段 `COMMENT`
- [ ] 时间字段 `COMMENT` 已包含"北京时间(UTC+8)，格式 yyyy-MM-dd HH:mm:ss"备注
- [ ] 表未使用 `TEXT` 类型；`JSON` 类型仅用于受控例外（数量限制 + 简单结构，见 §8）
- [ ] 表未使用 `CONSTRAINT` 外键约束（引用完整性由后端校验）
- [ ] 表未对 `is_deleted` 单独建索引
- [ ] 易混淆字段（`status`、`name` 等）已加表源前缀
- [ ] 删除操作走 `UPDATE ... SET is_deleted = 1`，无 `DELETE FROM`
- [ ] 查询默认带 `is_deleted = 0` 过滤
- [ ] 后端 create/update 路径已主动推送 `create_time` / `last_modified_time`
- [ ] 若主张豁免 `is_deleted`，已注释说明理由

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