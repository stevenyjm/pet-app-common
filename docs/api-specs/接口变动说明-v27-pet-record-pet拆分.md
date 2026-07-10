# 接口变动说明 - v27 pet_record_pet 表拆分

> **版本**：v27
> **日期**：2026-07-09
> **影响模块**：宠物日记录（餐食/运动会话参与者）

---

## 1. 变更背景

原 `pet_record_pet` 表用于存储餐食会话（`pet_meal_session`）和运动会话（`pet_exercise_session`）的参与宠物关系，但由于两个会话表的 `session_id` 可能存在冲突（均从 1 自增），导致关联查询时出现数据混淆。

**解决方案**：将 `pet_record_pet` 拆分为两个独立的关联表：
- `pet_meal_session_pet`：餐食会话与宠物的关联
- `pet_exercise_session_pet`：运动会话与宠物的关联

---

## 2. 数据库变更

### 2.1 旧表（已删除）

| 表名 | 说明 |
|------|------|
| `pet_record_pet` | 原会话-宠物关联表（已删除） |

### 2.2 新表

**pet_meal_session_pet**

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | INT | 主键，自增 |
| `session_id` | INT | 餐食会话 ID（关联 `pet_meal_session.session_id`） |
| `pet_id` | INT | 宠物 ID（关联 `pet.pet_id`） |
| `create_time` | DATETIME | 创建时间 |

**索引**：
- `uk_meal_session_pet`：`(session_id, pet_id)` 唯一索引
- `idx_meal_pet`：`(pet_id)` 普通索引

**pet_exercise_session_pet**

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | INT | 主键，自增 |
| `session_id` | INT | 运动会话 ID（关联 `pet_exercise_session.session_id`） |
| `pet_id` | INT | 宠物 ID（关联 `pet.pet_id`） |
| `create_time` | DATETIME | 创建时间 |

**索引**：
- `uk_exercise_session_pet`：`(session_id, pet_id)` 唯一索引
- `idx_exercise_pet`：`(pet_id)` 普通索引

---

## 3. 后端接口变动

### 3.1 接口列表

以下接口的 **请求参数** 和 **响应格式均无变化**，后端内部实现已适配新表结构：

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/v1/pets/:pet_id/daily-records` | GET | 获取宠物日记录列表 |
| `/api/v1/pets/:pet_id/daily-records/:record_date` | GET | 获取宠物指定日期日记录详情 |
| `/api/v1/pets/:pet_id/daily-records/:record_date` | PUT | 更新宠物日记录 |
| `/api/v1/pets/daily-records/joint` | POST | 创建联合日记录（多宠物） |
| `/api/v1/pets/daily-records/session/:session_id/participants` | PUT | 更新会话参与者列表 |
| `/api/v1/pets/daily-records/session/:session_id` | GET | 获取会话详情 |
| `/api/v1/pets/daily-records/session/:session_id` | DELETE | 删除会话 |

### 3.2 参数说明

所有接口均通过 `type` 查询参数区分会话类型：

| 参数 | 值 | 说明 |
|------|-----|------|
| `type` | `meal` | 餐食会话（默认） |
| `type` | `exercise` | 运动会话 |

### 3.3 响应格式示例

**获取日记录详情** `/api/v1/pets/:pet_id/daily-records/:record_date`

响应中 `meal_sessions` 和 `exercise_sessions` 的 `participants` 字段格式不变：

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "status_id": 1,
    "pet_id": 52,
    "record_date": "2026-06-30",
    "daily_mood": "happy",
    "daily_weight_kg": 5.2,
    "daily_stool_health": "normal",
    "meal_sessions": [
      {
        "session_id": 100,
        "period": "breakfast",
        "meal_time": "08:30:00",
        "participants": [
          {
            "pet_id": 52,
            "name": "旺财",
            "avatar_url": "https://example.com/avatar.jpg"
          }
        ],
        "items": []
      }
    ],
    "exercise_sessions": [
      {
        "session_id": 50,
        "location": "公园",
        "start_time": "18:00:00",
        "duration_minutes": 30,
        "participants": [
          {
            "pet_id": 52,
            "name": "旺财",
            "avatar_url": "https://example.com/avatar.jpg"
          }
        ]
      }
    ]
  }
}
```

---

## 4. C端（小程序）配合内容

### 4.1 需要确认的事项

✅ **接口调用无需修改**：所有接口的 URL、请求参数、响应格式均保持不变。

### 4.2 注意事项

1. **会话 ID 唯一性**：拆分后，餐食会话和运动会话的 `session_id` 各自独立自增，不再冲突
2. **参与者查询**：获取会话详情时，`participants` 列表将正确返回对应会话类型的参与宠物
3. **联合记录创建**：`POST /api/v1/pets/daily-records/joint` 接口会自动将参与者写入对应的新表

### 4.3 测试建议

1. 创建联合日记录（同时包含餐食和运动），验证两个会话的参与者均正确保存
2. 查询已存在的历史记录，确认数据迁移成功，参与者信息完整
3. 更新会话参与者，验证餐食和运动会话的参与者可独立修改

---

## 5. Web端（管理后台）配合内容

### 5.1 需要确认的事项

✅ **接口调用无需修改**：所有接口的 URL、请求参数、响应格式均保持不变。

### 5.2 注意事项

1. **会话管理**：在管理后台查看/编辑宠物日记录时，餐食和运动会话的参与者将正确关联
2. **数据展示**：历史数据已通过迁移脚本同步到新表，展示不受影响

### 5.3 测试建议

1. 在管理后台查看宠物日记录详情，确认餐食和运动会话的参与者正确显示
2. 测试会话参与者的增删改操作，验证数据一致性

---

## 6. 数据迁移说明

### 6.1 迁移脚本

| 脚本 | 说明 |
|------|------|
| `scripts/migrate-schema-v27-split-record-pet.js` | 创建新表 `pet_meal_session_pet` 和 `pet_exercise_session_pet` |
| `scripts/migrate-data-v27-split-record-pet.js` | 将 `pet_record_pet` 数据按 `session_type` 迁移到对应新表 |

### 6.2 迁移逻辑

```
pet_record_pet 数据
    │
    ├── session_type = 'meal'  →  pet_meal_session_pet
    │
    └── session_type = 'exercise'  →  pet_exercise_session_pet
```

### 6.3 迁移状态

- ✅ Schema 迁移已完成
- ✅ 数据迁移已完成
- ✅ 旧表 `pet_record_pet` 已删除
- ✅ 数据库验证通过（54 张表，无异常）

---

## 7. 错误码

本次变更不涉及新错误码，沿用现有错误码体系：

| 错误码 | 说明 |
|--------|------|
| 40101 | 未登录 / Token 失效 |
| 40301 | 无权访问（无宠物查看权限） |
| 40401 | 宠物不存在 |
| 40402 | 日记录不存在 |
| 50001 | 服务器内部错误 |

---

## 8. 兼容性

- ✅ 向后兼容：所有现有接口保持不变
- ✅ 历史数据：已通过迁移脚本完整迁移
- ✅ 无破坏性变更：C端和Web端无需修改代码即可正常使用