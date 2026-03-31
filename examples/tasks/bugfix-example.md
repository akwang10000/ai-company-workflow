# examples/tasks/bugfix-example.md

## 目标

提供一条可直接复用的 `bugfix` 任务单样例，并与当前 Phase 1 的修复、回归、审核和收口链路保持一致。

---

## 任务单样例

```yaml
task:
  title: 修复订单详情页偶发空白问题
  task_type: bugfix
  scenario: engineering
  status: Draft
  current_owner: tech_lead_agent
  next_owner: tech_lead_agent
  goal: |
    修复订单详情页在弱网重试后偶发空白的问题，恢复基础查看能力。
  acceptance_criteria:
    - 已知复现路径下不再出现空白页
    - 失败状态有明确错误提示
    - 相关详情页基础回归通过
  constraints:
    - 不改动订单核心业务逻辑
    - 不在本任务中做页面重构
```

---

## 推荐初始推进方式

- 推荐初始状态：`Draft`
- 推荐第一步 action：`ready_task`
- 推荐第一步接手：`tech_lead_agent` 或 `pm_agent`（按问题清晰度决定）

---

## 推荐关键记录

- 若根因不明但影响范围较大：先 `request_decision` 或补定位摘要
- Dev -> QA：应有修复点、自测摘要、未测项与剩余风险
- QA 返工：应生成 `ReviewRecord(result=rejected)`
- QA 放行：应生成 `ReviewRecord(result=passed)` 与 `DeliverySummaryRecord`
