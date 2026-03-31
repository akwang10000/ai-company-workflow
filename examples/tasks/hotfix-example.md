# examples/tasks/hotfix-example.md

## 目标

提供一条可直接复用的 `hotfix` 任务单样例，并体现“先止血、后补完整、但不能没有记录”的当前口径。

---

## 任务单样例

```yaml
task:
  title: 登录态失效导致大面积无法下单的紧急修复
  task_type: hotfix
  scenario: engineering
  status: Draft
  current_owner: tech_lead_agent
  next_owner: tech_lead_agent
  goal: |
    尽快恢复用户下单能力，并明确当前是临时止血还是完整修复。
  acceptance_criteria:
    - 关键用户链路恢复可用
    - 止血方案与剩余风险已说明
    - 是否需要 follow-up 已明确
  constraints:
    - 高风险生产动作前必须人工拍板
    - 必须保留审核和交付摘要
```

---

## 推荐初始推进方式

- 推荐初始状态：`Draft`
- 推荐第一步 action：`ready_task`
- 推荐第一步接手：`tech_lead_agent`

---

## 推荐关键记录

- 涉及回滚、降级、数据修复：必须 `request_decision`
- Dev / Tech Lead handoff：必须写清止血路径、回滚风险、未覆盖项
- QA 放行前：至少形成最小关键验证结果
- 完成交付时：必须说明是临时止血还是完整修复，并标记 follow-up
