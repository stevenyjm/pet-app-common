# 阶段一：微信退款（BE-405~407 Mock 前置）API 规范

> 更新日期：2026-08-14
> 适用：后端 pet-app-backend · 前端 pet-app-admin-web（二期 FE-601~602）
> 参考：multi-project.md §2.1 / §3.1 字段命名对齐（蛇形命名，响应 camelizeResponse 自动 camelCase）

## 1. 字段与枚举

### 1.1 退款模式 refund_mode（snake_case 入参）

| label | code（DB refund_mode TINYINT） | 说明 |
|-------|-----|------|
| mark_only | 0 | 仅标记退款（兼容旧协议默认，不发起真实微信退款） |
| wechat_refund | 1 | 调用微信原路退款（mock/real 双模式） |

### 1.2 退款状态 refund_status（DB refund_record.refund_status TINYINT）

| code | label | 说明 |
|------|-------|------|
| 0 | init | 初始化（暂未使用） |
| 1 | submitting | 提交中（已事务创建 RefundRecord，未收到微信响应） |
| 2 | processing | 处理中（微信异步处理或 mock PROCESSING） |
| 3 | success | 退款成功 |
| 4 | failed | 退款失败（微信失败或 mock 5% 随机失败） |
| 5 | cancelled | 取消（暂未开放给管理员） |

### 1.3 错误码（utils/businessCodes.js 新增 50003~50007）

| 错误码 | 常量名 | HTTP 状态 | 场景 |
|--------|--------|------|------|
| 50003 | REFUND_AMOUNT_INVALID | 400 | 退款金额非法（>实付 / <=0 / 小数超 2 位 / 缺字段） |
| 50004 | REFUND_SUBMIT_FAILED | 502 | 微信退款 SDK/网络抛错或 result_code FAIL |
| 50005 | REFUND_NO_PAYMENT_RECORD | 422 | 订单没有成功支付单，无法发起 wechat_refund |
| 50006 | REFUND_ALREADY_EXISTS | 422 | 同一售后已有成功退款，禁止重复提交 |
| 50007 | REFUND_NOT_SUPPORTED | 422 | 订单非微信支付通道（pay_channel != wechat） |

---

## 2. 管理端接口（/api/v1/admin/...）

### 2.1 PATCH /admin/after-sales/:id/audit （BE-405 主入口）

**鉴权**：`X-Admin-Key` 或 admin JWT（由 `adminAuthMiddleware` 注入 `req.admin.adminId`）。

**请求 body**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| status | INT(2/3/4) | 是 | 2 approved / 3 rejected / 4 refunded（兼容旧字段） |
| note | STRING | 否 | 审核备注，<=100 字符 |
| refund_mode | ENUM mark_only \| wechat_refund | 当 status=4 时有效 | 缺省 mark_only（兼容现有旧请求行为，不产生 RefundRecord） |
| refund_amount | DECIMAL(10,2) | 否 | 仅 wechat_refund 生效；缺省取售后单 refund_amount；<=订单实付且<=售后退款金额 |

**响应 data（经 camelizeResponse，snake/camel 都兼容消费）**：

```json
{
  "afterSalesId": 1001,
  "statusCode": 2,
  "status": "approved / refunded",
  "refund": {
    "refundId": 1,
    "outRefundNo": "REF202608141001",
    "refundMode": "wechat_refund",
    "refundModeCode": 1,
    "refundStatus": "processing / success / failed",
    "refundStatusCode": 2,
    "mock": true,
    "refundAmount": 99.90,
    "totalAmount": 99.90,
    "successTime": null,
    "submitTime": "2026-08-14T09:00:00.000Z",
    "callbackTime": null,
    "failReason": null,
    "statusDesc": "processing（等待退款结果回调）",
    "retryCount": 0,
    "canRetry": false
  },
  "latestRefund": "<同 refund 字段，冗余提供列表兼容>"
}
```

**Mock 注入（仅开发/QA）**：
Header: `X-Mock-Hint: SUCCESS | FAIL | PROCESSING`
- 仅 `WECHAT_REFUND_MODE=mock`（或 real 模式但配置不全 → 自动降级 mock）时生效
- 不提供 Hint → 默认 95% SUCCESS / 5% FAIL（模拟真实失败分布）

### 2.2 GET /admin/after-sales?page_size=20&page=1&status=pending

**扩展**：list 每项追加 `latestRefund` 字段（RefundRecord 摘要，若无则 null）。批量一次性查询，无 N+1。

### 2.3 GET /admin/payments/config-status （支付/退款配置就绪查询）

**响应 data**：

```json
{
  "refundReady": false,
  "refundMode": "mock",
  "effectiveRefundMode": "mock",
  "paymentAvailable": true,
  "paymentStatus": "enabled",
  "notifyUrlConfigured": false,
  "refundNotifySuggestedUrl": "https://your-domain/api/v1/payments/refund-notify",
  "skipSignVerify": true
}
```

**用途**：FE-601 在「微信原路退款」开关处判断可用性：
- `effectiveRefundMode === 'real'` → UI 强提示生产环境，按钮绿
- `effectiveRefundMode === 'mock'` → 黄底提示「当前为 Mock 模式，不会真实扣款」

---

## 3. 回调接口（无鉴权）

### POST /api/v1/payments/refund-notify （BE-406）

**入参 body**：微信 XML（`text/xml`）。生产会做 `verifyNotifySign`；mock 模式或 `WECHAT_REFUND_SKIP_SIGN_VERIFY=true` 可跳过。

**响应 body**：`text/xml`，标准 `<xml><return_code>SUCCESS</return_code><return_msg>OK</return_msg></xml>`。

**幂等**：同一 `out_refund_no` 回调 N 次 →
- `after_sales.status=4` 仅变 1 次
- `after_sales_audit_wechat_refund_success` 审计只记 1 条
- `after_sales_refund_callback_received` 每次接收都会记录（用于排查重复回调）

---

## 4. 命令行脚本

| npm 命令 | 说明 | 生产调度建议 |
|----------|------|-------------|
| `npm run migrate:v43` | 创建 refund_record 表 + 索引（幂等） | 部署后一次性执行 |
| `npm run cron:refund-compensation` | 扫描 0/1/2 状态 5 分钟未处理 → 重试/轮询（mock 模式自动回调） | 每 5 分钟 cron |
| `npm run verify:refund-mock` | QA-001~008 一期联调全链路 Mock 验证 | CI/提测前执行 |

---

## 5. 三端对齐检查要点

### 5.1 字段命名（multi-project.md §3.2）
- 数据库 / Model：snake_case（refund_id、out_refund_no、after_sales_id）
- 路由 path：snake_case（/after-sales/:id/audit，:id 为 after_sales_id）
- 请求 body：snake_case（refund_mode、refund_amount）
- 响应：**snake_case 写入后，camelizeResponse 自动转 camelCase**（管理端消费 camelCase，小程序可直接消费 snake_case 原始路由输出）

### 5.2 时间字段（§3.3）
- 统一北京时间（返回 ISO8601 字符串，后端写入 UTC 本地时区）
- 前端统一使用 `formatDateTime` 格式化

### 5.3 新增字段清单
| 新增位置 | 字段 | 说明 |
|---------|------|------|
| refund_record 表 v43 | 20 列 | 见 scripts/schema.sql / models/RefundRecord.js |
| verify-db.js | refund_record | 字段 + 5 索引校验 |

---

## 6. 审计动作字典（admin_audit_log.action）

| action | resource_type | 触发点 |
|--------|---------------|--------|
| after_sales_audit_approve | after_sales | approved 或 wechat_refund 进入中间态前 |
| after_sales_audit_reject | after_sales | rejected |
| after_sales_audit_mark_refund | after_sales | mark_only 兼容旧流程 |
| after_sales_audit_wechat_refund_submit | refund_record | 提交微信退款 |
| after_sales_audit_wechat_refund_success | refund_record | 退款成功（回调或补偿） |
| after_sales_audit_wechat_refund_fail | refund_record | 退款失败（含原因） |
| after_sales_audit_wechat_refund_retry | refund_record | 补偿脚本重试 |
| after_sales_refund_callback_received | refund_record | 接收回调（含验签失败也记录，脱敏） |

**脱敏（NFR-3）**：
- `detail_json.transaction_id` → 前 4 后 4，其余 `*`
- `WECHAT_API_KEY` / `WECHAT_MCH_ID` 绝对不得出现在 detail_json / 响应 / 日志；仅 RefundRecord.extra_json 里脱敏快照。
