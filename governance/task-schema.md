# governance/task-schema.md

## 目标

定义 **Phase 1（软件公司模板 / 研发团队版）** 使用的统一任务单结构，让 Codex / 智能体 / 人类执行者在接单、补全、交接、审核、交付、归档时使用同一套最小字段。

本文件当前只服务于：

- Phase 1 软件研发团队场景
- `feature` / `bugfix` / `hotfix` 三类任务
- 任务单创建、补字段、治理校验与示例编写

本文件不是：

- 通用多行业终态 schema
- 后端运行时模型单一真相
- API DTO 单一真相

正式运行时对象与接口契约仍以：

- `product/domain-model.md`
- `product/api-contracts.md`
- `product/task-transition-api-and-actions.md`

为准。

---

## Phase 1 固定范围

### `task_type`

当前只允许：

- `feature`
- `bugfix`
- `hotfix`

### `scenario`

当前固定为：

- `engineering`

### 固定状态枚举

- `Draft`
- `Ready`
- `In Progress`
- `Waiting Handoff`
- `Waiting Decision`
- `In Review`
- `Rework Required`
- `Ready for Delivery`
- `Done`
- `Cancelled`
- `Archived`

---

## 最小可执行任务单字段

### 1. 基础标识

- `task_id`
- `title`
- `task_type`
- `scenario`

### 2. 目标定义

- `goal`
- `business_context`
- `problem_statement`
- `expected_outcome`
- `non_goals`

### 3. 输入资料

- `inputs`
- `dependencies`
- `constraints`

### 4. 输出要求

- `deliverables`
- `output_format`
- `acceptance_criteria`
- `definition_of_done`

### 5. 执行控制

- `priority`
- `status`
- `planned_owner`
- `current_owner`
- `next_owner`
- `due_at`
- `risk_level`

### 6. 决策与审批

- `decision_required`
- `decision_reason`
- `decision_options`
- `recommended_option`
- `approver`

### 7. 执行记录

- `execution_plan`
- `work_log`
- `handoff_note`
- `review_note`
- `rework_note`
- `known_risks`
- `blocked_by`

### 8. 归档信息

- `final_result`
- `artifacts`
- `postmortem_needed`
- `archived_at`

---

## 与运行时真相的边界

这份文档里的字段写法以 **治理层 / 人类可读** 为主，允许使用 snake_case 和角色名称；但正式运行时真相不以这里为准，而以 product 主文档为准。

### owner 字段映射

- `planned_owner` -> `plannedOwner`
- `current_owner` -> `currentOwner`
- `next_owner` -> `nextOwner`

### 关键边界

- `planned_owner` 可在任务创建或补字段阶段维护
- `current_owner` / `next_owner` 的业务变化以 transition 为准
- 不允许把本文件当作普通 PATCH 更新 `status / currentOwner / nextOwner` 的依据

---

## 字段使用要求

### `goal`

必须描述想达成的业务结果，而不是只写动作。

### `acceptance_criteria`

必须写成可检查项，不能写“功能正常”“体验良好”这类空话。

### `definition_of_done`

必须明确 Done 的边界，防止“差不多就算完成”。

### `handoff_note`

角色切换时必须补，至少说明：

- 已完成什么
- 未完成什么
- 风险点是什么
- 下一角色应该先看什么

---

## 研发团队版任务单示例

```yaml
task_id: ENG-001
title: 新增用户封禁记录查询页
task_type: feature
scenario: engineering
goal: 新增后台封禁记录查询页，降低运营人工查库支持成本
business_context: 运营当前遇到用户投诉时，需要研发临时查库确认封禁历史
problem_statement: 缺少运营自助查询能力，研发被动承担重复支持工作
expected_outcome: 运营主管可在后台按用户 ID 和时间范围查询封禁记录
non_goals:
  - 不支持直接解封
  - 不支持修改封禁记录
inputs:
  - 后台权限系统说明
  - 封禁记录数据表结构
  - 现有后台页面路由规范
dependencies:
  - 权限中间件可复用
constraints:
  - 第一版仅支持内部后台使用
  - 必须复用现有权限体系
deliverables:
  - 需求摘要
  - 技术方案
  - 代码实现
  - 自测说明
  - 审核结论
output_format: mixed
acceptance_criteria:
  - 支持按用户 ID 查询
  - 支持按时间范围筛选
  - 展示封禁原因、操作人、操作时间
  - 无权限用户不可访问
definition_of_done: 需求、实现、审核、交付说明均完成，并获得最终确认
priority: medium
status: Ready
planned_owner: PM Agent
current_owner: PM Agent
next_owner: Tech Lead Agent
due_at: TBD
risk_level: medium
decision_required: false
decision_reason: ""
decision_options: []
recommended_option: ""
approver: CEO / 指定 Approver
execution_plan:
  - PM 澄清需求
  - Tech Lead 输出方案
  - Dev 实现并自测
  - QA 审核
  - Ops 收口
work_log: []
handoff_note: "PM 已明确需求和验收标准，待技术评估"
review_note: ""
rework_note: ""
known_risks:
  - 大范围查询性能未知
blocked_by: []
final_result: ""
artifacts: []
postmortem_needed: false
archived_at: ""
```

---

## 当前阶段统一结论

`governance/task-schema.md` 只负责：

- Phase 1 研发团队版任务单填写结构
- `feature / bugfix / hotfix` 示例字段
- 治理层补字段与校验口径

它不再承担：

- 多行业终态 schema
- 运行时 DB 设计真相
- 通用任务平台扩展入口
