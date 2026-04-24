# 家长后台字段、任务模型与完成纠偏规则

- Issue: #5
- 目标：收敛家长后台最小配置页的字段定义、任务模型与完成纠偏交互规则，为后续 API/存储设计打底

---

## 1. 家长后台字段清单

### 1.1 孩子档案（FamilyChild）

| 字段 | 类型 | 说明 | 必填 |
|------|------|------|------|
| child_id | UUID / auto | 系统生成唯一标识 | ✓ |
| name | string(20) | 孩子昵称，如"大宝" | ✓ |
| color | hex string | 面板内用于区分该孩子的视觉颜色，如 #FF6B6B | ✓ |
| sort_order | int | 多孩子切换时的排列顺序，1-4 | ✓ |
| avatar_url | string (optional) | 头像图片 URL，不设则用 color 圆形占位 | |

**约束：最多 4 个孩子。**

### 1.2 任务模板（TaskTemplate，可选，用于复用）

| 字段 | 类型 | 说明 | 必填 |
|------|------|------|------|
| template_id | UUID / auto | 系统生成唯一标识 | ✓ |
| title | string(50) | 任务名称，如"写作业""洗澡" | ✓ |
| default_duration_minutes | int | 默认预计时长（分钟） | ✓ |
| scope_type | enum | `shared` = 共用任务，`personal` = 个人任务 | ✓ |
| default_assigned_child_id | UUID (nullable) | 个人任务默认归属孩子；共用任务为空 | shared 时为空 |
| default_order_index | int | 默认顺序号 | |

### 1.3 当日任务实例（TaskInstance，当日数据）

| 字段 | 类型 | 说明 | 必填 |
|------|------|------|------|
| instance_id | UUID / auto | 系统生成唯一标识 | ✓ |
| task_template_id | UUID (nullable) | 若来自模板则关联模板 ID；手动创建可为空 | |
| title | string(50) | 任务名称（直接冗余存储，便于展示） | ✓ |
| duration_minutes | int | 当日配置的预计时长（分钟） | ✓ |
| order_index | int | 当日在该孩子视图中的排列顺序 | ✓ |
| scope_type | enum | `shared` 或 `personal` | ✓ |
| assigned_child_id | UUID (nullable) | personal 时为孩子 ID；shared 时为空 | personal 时必填 |
| status | enum | `pending` / `done` | ✓ 默认 pending |
| completed_at | datetime (nullable) | 孩子点击完成的时间戳 | done 时必填 |
| completed_by_child_id | UUID (nullable) | 谁点的完成 | done 时必填 |
| parent_override | boolean | 家长是否手动干预过此任务（影响撤销逻辑） | 默认 false |

### 1.4 当日计划（DailyPlan）

| 字段 | 类型 | 说明 | 必填 |
|------|------|------|------|
| plan_id | UUID / auto | 系统生成唯一标识 | ✓ |
| plan_date | date | 当日日期，格式 YYYY-MM-DD | ✓ 唯一 |
| start_time | time | 当日晚间起始时间，如 17:30 | ✓ |
| end_time | time | 当日晚间结束时间，如 21:00 | ✓ |
| notes | string(200) (optional) | 家长备注，记录当天特殊安排 | |

**约束：start_time < end_time；总任务时长不应超过时间窗口（后台仅警告提示，不阻止保存）。**

---

## 2. 任务模型与状态定义

### 2.1 任务类型（scope_type）

| 类型 | 值 | 含义 | 出现在谁的面板上 |
|------|-----|------|----------------|
| 共用任务 | `shared` | 所有孩子都需要完成的任务 | 所有 4 个孩子的面板 |
| 个人任务 | `personal` | 仅指定孩子需要完成的任务 | 仅指定孩子的面板 |

**生成规则：** 每个孩子的展示任务列表 = `scope_type=shared` 的全部任务 + `scope_type=personal AND assigned_child_id=该孩子` 的任务，按 `order_index` 升序排列。

### 2.2 任务状态（status）

| 状态 | 值 | 触发条件 | 界面表现 |
|------|-----|---------|---------|
| 待完成 | `pending` | 初始状态 / 被家长撤销后 | 正常显示在时间条格子里 |
| 已完成 | `done` | 孩子点击完成（或家长手动标记） | 格子上叠加完成标记（如 ✓），不再消耗时间条 |
| 已跳过 | `skipped` | 家长手动跳过（可选，MVP 可不实现） | 不显示或带跳过标记 |

### 2.3 面板全局状态（PanelStatus）

与任务状态独立，描述当晚整体所处阶段：

| 状态 | 触发条件 | 孩子界面表现 |
|------|---------|------------|
| `before_start` | 当前时间 < start_time | 显示"今晚计划将于 HH:MM 开始"，主倒计时未进入消耗 |
| `in_progress` | start_time ≤ 当前时间 ≤ end_time | 实时剩余时间，任务格子消耗中 |
| `all_done` | end_time 内或外，所有任务 status=done | "今晚任务已完成！" |
| `overtime_incomplete` | 当前时间 > end_time 且仍有 pending 任务 | "已超出计划时间"，未完成格子仍显示紧迫感 |

---

## 3. 完成 / 撤销 / 超时关键交互规则

### 3.1 孩子点击完成

**触发：** 孩子点击某个 pending 任务格子的"完成"按钮。

**系统行为：**
1. 该任务 `status` 变为 `done`
2. `completed_at` 记录当前时间戳
3. `completed_by_child_id` 记录点击的孩子 ID
4. 立即重新计算该孩子的可用自由时间，更新时间条发光区域
5. 若点击的是当前下一步任务，自动高亮下一项 pending 任务

**边界处理：**
- 已 done 的任务不可重复点击（按钮隐藏或禁用）
- 进行中（in_progress）状态外也可以点完成——允许孩子提前做完
- 时间窗口结束后仍可标记完成，status 变为 done，但不再影响时间条刻度

### 3.2 家长撤销完成（纠偏）

**触发：** 家长在后台发现孩子误点，手动撤销。

**系统行为：**
1. 找到该孩子最近一条 `status=done` 且 `completed_at` 最新的 TaskInstance
2. 将 `status` 恢复为 `pending`
3. `completed_at` 和 `completed_by_child_id` 置为空
4. `parent_override` 标记为 `true`（便于后续审计）
5. 立即重新计算自由时间和紧迫感，更新该孩子面板

**约束：**
- 每次撤销仅回退一条最近完成记录
- 不支持回退多条（如需多条回退，分多次操作）
- 已超时的任务仍可撤销，不限时间

### 3.3 时间窗口超时

**触发：** 系统时钟跨过 `end_time`，仍有 `status=pending` 的任务。

**系统行为：**
1. 面板全局状态变为 `overtime_incomplete`
2. 未完成格子以紧迫感视觉强调（如发热/膨胀效果）
3. 主倒计时显示"已超出 XX 分钟"
4. 不自动标记任何任务为 done——超时≠完成

### 3.4 超时后任务完成

**触发：** 孩子在超时状态下点击完成。

**系统行为：**
- 与正常完成行为完全一致
- 全局状态仍为 `overtime_incomplete`，但该任务变为 done
- 家长可继续撤销

### 3.5 无任务配置

**边界：** 当日 `DailyPlan` 存在但无任何 TaskInstance，或仅有已 done 任务。

**孩子界面表现：** 显示"今晚暂无任务安排"，不显示空白错误或报错。

### 3.6 任务总时长超过时间窗口

**边界：** 所有任务 duration_minutes 之和 > (end_time - start_time)。

**家长后台行为：** 保存时给出警告提示"任务总时长超过时间窗口 XX 分钟，可能会来不及完成"，但不阻止保存。

**孩子界面行为：** 正常显示，时间条会完整消耗殆尽，视觉上体现"没有自由时间"的紧迫感。

---

## 4. 状态流转图

```
[pending]  --孩子点击完成-->  [done]
    ^                            |
    |                            | 家长撤销（最近一条）
    |                            |
    +------- parent_override = true（pending恢复）
```

```
[before_start] --> [in_progress] --> [all_done]
                        |
                        v
               [overtime_incomplete]
               （有pending且时间>end_time）
```

---

## 5. 后续设计提示

- **TaskInstance** 是当日数据，建议按 `plan_date` 分表或加索引，便于每日清理历史
- **撤销操作**建议记录操作日志（谁在何时撤销了哪条 instance），不写在核心 model 里但建议留审计字段
- **时间窗口**的实时判断建议在客户端做展示，服务端以 `plan_date + start_time + end_time` 为权威数据源
- **多人同时操作**（孩子点完成 + 家长撤销）暂不在 MVP 考虑，需要时加乐观锁
