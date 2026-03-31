# examples/tasks/feature-example.md

## 目标

提供一条可直接复用的 `feature` 任务单样例，并与当前 Phase 1 的 action-driven / record-driven 任务推进口径保持一致。

---

> 说明：以下 `current_owner / next_owner` 为治理层人类可读写法；正式运行时字段、owner 真相与可更新边界，以 `product/domain-model.md`、`product/api-contracts.md`、`product/task-transition-api-and-actions.md` 为准。

---

## 任务单样例

```yaml
task:
  title: 后台增加封禁记录查询页
  task_type: feature
  scenario: engineering
  status: Draft
  current_owner: pm_agent
  next_owner: pm_agent
  goal: |
    让运营可以在后台按用户维度查看封禁记录，减少人工排查成本。
  non_goals:
    - 本期不做批量解封
    - 本期不做导出 Excel
  acceptance_criteria:
    - 支持按用户 ID 查询封禁记录
    - 支持按时间范围筛选
    - 展示封禁原因、操作人、操作时间
    - 无权限用户不可访问
    - 页面结果展示正常
  definition_of_done:
    - 需求边界已澄清
    - 技术方案已确认
    - 实现与自测已完成
    - 审核 / 回归通过
    - 已形成交付摘要并进入最终确认
  constraints:
    - 不影响现有后台菜单结构
    - 不泄露敏感数据
```

---

## 推荐初始推进方式

- 推荐初始状态：`Draft`
- 推荐第一步 action：`ready_task`
- 推荐第一步接手：`pm_agent`

---

## 推荐关键记录

- PM -> Tech Lead：若范围已明确，可准备 handoff 摘要
- 若权限边界存在争议：进入 `Waiting Decision` 并生成 `DecisionRecord`
- Dev -> QA：进入审核前应有 `HandoffRecord`
- QA 放行：应生成 `ReviewRecord(result=passed)`
- 进入 `Ready for Delivery` 前应有 `DeliverySummaryRecord`
