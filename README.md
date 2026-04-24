# iPad 家庭晚间任务时间面板

这是一个围绕“4 个孩子晚间任务时间条 + 家长后台配置”的立项基线仓库。

仓库当前目标不是直接交付完整产品代码，而是先把这个项目按 GitHub + 多 Agent 工作流正式立起来，让后续 UI 原型、技术设计、任务拆分与开发协作都能在同一个仓库内留痕推进。

## 项目定位

这个项目要解决的问题是：

- 让孩子在 iPad 上直观看见晚间时间如何被任务占用
- 让孩子一眼知道“当前下一步该做什么”
- 让完成任务后释放出来的自由时间可视化、可感知
- 让家长可以通过后台快速配置和纠偏，而不是靠反复催促

MVP 当前已确认的核心范围：

1. 支持 4 个孩子切换
2. 保留家长后台最小配置页
3. 支持共用任务 + 个人独立任务并存
4. 支持当日可调整的时间窗口
5. 自由时间定义为未被任务格子占据的发光时间
6. 孩子主界面突出“当前下一步任务”
7. 至少覆盖开始前、进行中、结束后未完成 3 种基础状态

## 本仓库当前交付什么

本仓库当前交付的是“立项基线”，主要包括：

1. `docs/`：调研、PRD、决策记录、来源与范围说明
2. `rules/`：Hermes / OpenClaw / 子 Agent 的职责与状态推进规则
3. `templates/`：README、立项、任务拆分、review 结论模板
4. `.github/`：Issue / PR 协作模板
5. 第一批阶段 Issue：用于后续原型、技术方案和开发立项推进

## 当前不包含什么

本仓库当前不直接包含：

- 完整可运行的 iPad/PWA 业务代码
- 自动化调度引擎
- 自动 merge / 自动排期
- 复杂账号系统与云端同步
- 统计报表、积分系统、AI 自动排任务

## 目录结构

```text
.github/
  ISSUE_TEMPLATE/
  PULL_REQUEST_TEMPLATE.md

docs/
  research.md
  prd.md
  decision-log.md
  source-links.md
  mvp-scope.md

rules/
  agent-role-playbook.md
  openclaw-patrol-rules.md
  state-mapping.md

templates/
  project-readme-template.md
  hermes-project-init-template.md
  issue-task-template.md
  hermes-review-template.md
  user-stage-prompts.md
```

## 推荐推进方式

1. 先阅读 `docs/prd.md` 和 `docs/decision-log.md`
2. Hermes 基于 PRD 做阶段拆分与 GitHub 立项
3. OpenClaw 主 Agent 根据 `rules/openclaw-patrol-rules.md` 做巡视、派工与回流
4. 子 Agent 只接明确 scope 的 Issue / PR 执行任务
5. 用户只在方向拍板、产品判断与最终验收时介入

## 当前建议的第一批推进方向

1. 页面信息架构与状态图
2. 孩子主界面高保真 UI 原型
3. 家长后台字段与任务模型设计
4. 技术架构与 NAS / PWA 落地方案

## 当前状态

- 项目状态：`planned`
- 当前阶段：立项基线已建立，等待进入原型 / 技术方案拆解
- 对应立项 Issue：#1

## 来源文档

- `docs/research.md`
- `docs/prd.md`
- `docs/decision-log.md`
