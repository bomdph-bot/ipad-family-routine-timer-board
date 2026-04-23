# 状态语义到 GitHub 留痕的映射

## 项目 / 任务常用状态

- `new`：刚被提出，尚未完成立项
- `planned`：范围已收敛，等待执行或已可派发
- `in_progress`：已有明确执行动作正在推进
- `reviewing`：已有 PR 或待评审结果
- `blocked`：存在未满足依赖或待拍板问题
- `done`：已完成当前目标并被确认

## 在 GitHub 中的推荐留痕方式

### 1. `new`

- 体现在项目立项 Issue 刚创建
- Issue 中写清输入来源、当前理解和默认范围

### 2. `planned`

- README、docs、第一批任务 Issue 已建立
- 任务 Issue 中写清输入 / 输出 / 边界 / 验收标准

### 3. `in_progress`

- 已有执行分支或 PR
- 进展应通过 Issue / PR comment 更新，不只在聊天里说明

### 4. `reviewing`

- PR 已创建
- review 结论、测试结果和风险点已写入 PR

### 5. `blocked`

- 在 Issue 或 PR 中明确写出阻塞原因、依赖项和解锁条件

### 6. `done`

- PR 已合并或阶段产物已被确认
- Issue 中应写明完成条件已满足
