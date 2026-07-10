# 餐食记录 API 字段统一方案

**版本**: v1.0
**日期**: 2026-06-21
**状态**: 待前端确认

---

## 1. 背景

当前 `GET /api/v1/pets/{pet_id}/daily-records/{date}` 响应中存在重复字段，增加了响应体积。为保持 API 简洁统一，特制定本方案。

> **约定**：`meal_time` 和 `session_time` 保持不变动。

---

## 2. 统一方案

### 2.1 会话时段字段

| 当前响应字段 | 保留字段 | 移除字段 | 保留原因 |
|-------------|---------|---------|---------|
| `period` | **保留** | - | 数据库表 `pet_meal_session.period` 原生字段，语义清晰（早/午/晚餐） |
| `session_slot` | - | **移除** | 模板表 `pet_meal_template.session_slot` 的遗留别名 |

### 2.2 营养参考字段

| 当前响应字段 | 保留字段 | 移除字段 | 保留原因 |
|-------------|---------|---------|---------|
| `nutrition_reference` | **保留** | - | session 级营养汇总的数据库原生字段（`pet_meal_session.nutrition_reference`），命名完整明确 |
| `nutrition_ref`（session 级） | - | **移除** | session 级仅作 `nutrition_reference` 的别名，冗余 |

> **注意**：item 级的 `nutrition_ref`（如 `items[].nutrition_ref`）**必须保留**，对应数据库表 `pet_meal_item.nutrition_ref`。

### 2.3 食物清单字段

| 当前响应字段 | 保留字段 | 移除字段 | 保留原因 |
|-------------|---------|---------|---------|
| `items` | **保留** | - | 简洁通用，与其他模块命名一致（如 `exercise_sessions`） |
| `meal_items` | - | **移除** | 完全冗余，内容与 `items` 完全相同 |

---

## 3. 修改后的响应结构示例

### 3.1 餐食会话（`meal_sessions[]`）

**修改前**：
```json
{
  "session_id": 62,
  "period": "早餐",
  "session_slot": "早餐",           // ❌ 移除
  "meal_time": "07:30:00",
  "session_time": "07:30",
  "nutrition_reference": "粗蛋白 65g · 粗脂肪 22g · ...",
  "nutrition_ref": "粗蛋白 65g · 粗脂肪 22g · ...",  // ❌ 移除
  "meal_items": [...],              // ❌ 移除
  "items": [...]
}
```

**修改后**：
```json
{
  "session_id": 62,
  "period": "早餐",
  "meal_time": "07:30:00",
  "session_time": "07:30",
  "nutrition_reference": "粗蛋白 65g · 粗脂肪 22g · ...",
  "items": [...]
}
```

### 3.2 食物项目（`items[]` / `meal_items[]`）

**修改前**：
```json
{
  "item_id": 100,
  "food_name": "鸡肉鲜食",
  "amount": "100g",
  "nutrition_ref": "粗蛋白 65g · 粗脂肪 22g · ...",
  "nutrition_json": {...},
  "nutrition_source": "manual",
  "sort_order": 0
}
```

**修改后**：（item 级 `nutrition_ref` 保留不变）

---

## 4. 字段变更总览

| 层级 | 移除字段 | 保留字段 |
|------|---------|---------|
| session | `session_slot` | `period` |
| session | `nutrition_ref` | `nutrition_reference` |
| session | `meal_items` | `items` |
| item | - | `nutrition_ref`（保留） |

---

## 5. 后端修改范围

| 文件 | 修改内容 |
|------|---------|
| `utils/petFormatters.js` | `formatMealSession()` 移除 `session_slot`、`nutrition_ref`、`meal_items` 别名 |

---

## 6. 前端适配要点

1. **读取 `period` 替代 `session_slot`**
2. **读取 `nutrition_reference` 替代 session 级 `nutrition_ref`**
3. **读取 `items` 替代 `meal_items`**
4. **item 级的 `nutrition_ref` 保持不变**

---

## 7. 兼容性说明

- 本次修改为**破坏性变更**，需要前端同步更新
- 修改后版本建议更新 API 版本号（如 `v1` → `v2`）或通过 Feature Flag 控制