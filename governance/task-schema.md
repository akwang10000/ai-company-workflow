# governance/task-schema.md

## 目标
定义本项目统一的任务单结构，让 Codex / 智能体在接单、执行、交接、审核、归档时使用同一套字段。

这份文档优先服务于：
- 研发团队版第一阶段
- 后续迁移到 AI 短剧、自媒体运营等场景时的复用

---

## 设计原则
- 没有任务单，不进入正式执行
- 没有验收标准，不进入执行阶段
- 没有负责人，不允许悬空流转
- 没有状态变化，不算有效推进
- 任务单必须支持交接、返工、审核、归档

---

## 最小可执行任务单字段

### 1. 基础标识
- `task_id`：任务唯一 ID
- `title`：任务名称
- `task_type`：任务类型
  - `feature`
  - `bugfix`
  - `research`
  - `refactor`
  - `ops`
  - `content`
  - `custom`
- `scenario`：所属场景
  - `engineering`
  - `short_drama`
  - `self_media`
  - `generic`

### 2. 目标定义
- `goal`：任务目标
- `business_context`：业务背景
- `problem_statement`：当前要解决的问题
- `expected_outcome`：期望结果
- `non_goals`：明确不做的范围

### 3. 输入资料
- `inputs`：输入材料列表
- `dependencies`：依赖项
- `constraints`：约束条件
  - 时间约束
  - 技术约束
  - 权限约束
  - 成本约束

### 4. 输出要求
- `deliverables`：需要交付的产物
- `output_format`：输出格式
  - `doc`
  - `code`
  - `report`
  - `checklist`
  - `media`
  - `mixed`
- `acceptance_criteria`：验收标准列表
- `definition_of_done`：Done 判定

### 5. 执行控制
- `priority`：优先级
  - `low`
  - `medium`
  - `high`
  - `critical`
- `status`：当前状态
  - `Draft`
  - `Queued`
  - `In Analysis`
  - `In Progress`
  - `Review`
  - `Rework`
  - `Waiting Decision`
  - `Ready for Delivery`
  - `Done`
  - `Archived`
- `current_owner`：当前负责人
- `next_owner`：下一步接收方
- `due_at`：截止时间
- `risk_level`：风险等级
  - `low`
  - `medium`
  - `high`

### 6. 决策与审批
- `decision_required`：是否需要人工决策
- `decision_reason`：为什么需要决策
- `decision_options`：可选方案列表
- `recommended_option`：推荐方案
- `approver`：审批人

### 7. 执行记录
- `execution_plan`：当前执行计划
- `work_log`：执行日志
- `handoff_note`：交接说明
- `review_note`：审核意见
- `rework_note`：返工意见
- `known_risks`：已知风险
- `blocked_by`：阻塞项

### 8. 归档信息
- `final_result`：最终结果摘要
- `artifacts`：产物清单
- `postmortem_needed`：是否需要复盘
- `archived_at`：归档时间

---

## 字段解释与使用要求

### `goal`
必须用一句清晰的话说明任务想达成什么，而不是只写动作。

错误示例：
- 做个页面

正确示例：
- 新增后台封禁记录查询页，降低运营人工找研发查库的成本

### `acceptance_criteria`
必须写成可检查项，不能写成空话。

错误示例：
- 功能正常
- 体验良好

正确示例：
- 输入用户 ID 可以查到对应封禁记录
- 无权限用户访问时被拒绝
- 查询结果包含封禁原因、操作人、操作时间

### `definition_of_done`
用于防止代理“自认为差不多就算完成”。必须明确完成边界。

### `handoff_note`
角色切换时必须补。至少说明：
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

output_format: mixed
deliverables:
  - 需求摘要
  - 技术方案
  - 代码实现
  - 自测说明
  - 审核结论
acceptance_criteria:
  - 支持按用户 ID 查询
  - 支持按时间范围筛选
  - 展示封禁原因、操作人、操作时间
  - 无权限用户不可访问
  - 查询页结果展示正常
definition_of_done: 需求、实现、审核、交付说明均完成，并获得最终确认

priority: medium
status: In Analysis
current_owner: PM Agent
next_owner: Tech Lead Agent
due_at: TBD
risk_level: medium

decision_required: false
decision_reason: ""
decision_options: []
recommended_option: ""
approver: CEO

execution_plan:
  - PM 澄清需求
  - Tech Lead 输出方案
  - Dev 实现并自测
  - QA 审核
  - CEO 确认交付
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

## 状态推进规则
- `Draft`：仅允许补充任务信息
- `Queued`：等待分派，不允许直接宣称完成
- `In Analysis`：允许补需求、补方案、补上下文
- `In Progress`：必须有明确执行计划
- `Review`：必须附带可审核产物
- `Rework`：必须附带返工意见
- `Waiting Decision`：必须附带决策原因、选项、推荐项
- `Ready for Delivery`：表示已具备交付条件，等待最终确认
- `Done`：必须满足 `definition_of_done`
- `Archived`：必须有最终摘要和产物索引
