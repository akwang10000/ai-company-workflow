# examples/tasks/hotfix-example.md

## 目标
提供一条可直接复用的 **hotfix 任务单样例**，并与当前 Phase 1 的 action-driven / record-driven 任务推进口径保持一致。

---

> 说明：以下 `current_owner` / `next_owner` 为治理层人类可读写法；
> 正式运行时字段、owner 真相与可更新边界，以 `product/domain-model.md`、`product/api-contracts.md`、`product/task-transition-api-and-actions.md` 为准。

## 示例任务
- task_id: `HOTFIX-001`
- title: `紧急修复支付回调重复入账导致订单状态异常`
- task_type: `hotfix`
- scenario: `engineering`
- goal: `先止血支付回调重复入账问题，阻止新订单继续异常`
- business_context: `线上支付链路出现重复回调时，订单被重复入账，造成订单状态异常和资金风险。`
- problem_statement: `当前系统未正确防重处理支付回调，重复通知会触发重复入账。`
- expected_outcome: `重复回调不再导致重复入账，新订单主支付路径恢复稳定。`
- non_goals:
  - `本次不一次性重构整个支付回调架构`
  - `本次不覆盖所有历史脏数据治理`
- status: `Draft`
- priority: `critical`
- current_owner: `Tech Lead Agent`
- next_owner: `Dev Agent`
- known_risks:
  - `止血方案若不完整，可能仍有重复入账风险`
  - `是否直接上线、是否回滚需要明确拍板`
  - `历史异常订单可能需要 follow-up 任务单独治理`
- decision_required: `true`
- decision_reason: `需要明确当前 hotfix 是直接上线、先回滚，还是先限流止血。`
- recommended_option: `先止血并验证主支付路径，再决定是否补充根因修复任务`

---

## 验收标准示例
- 同一支付单号重复回调时不会重复入账
- 新订单支付主路径可继续完成
- 当前 hotfix 是止血还是根因修复已明确说明
- follow-up 治理任务已提出

---

## Done 判定示例
- 止血方案已明确并执行
- 关键支付主路径验证通过
- 是否直接上线 / 是否回滚已有明确拍板记录
- 已输出 hotfix 交付摘要
- follow-up 治理任务已登记

---

## 推荐初始推进方式
- 推荐初始状态：`Draft`
- 推荐第一步 action：`submit_ready` 或 `request_decision`
- 推荐第一步接手：`Tech Lead Agent`

### 推荐关键记录
- 若是否直接上线 / 回滚存在分歧，先进入 `Waiting Decision` 并生成 `DecisionRecord`
- Dev 提交验证前，应形成 handoff 摘要
- QA / Review 对关键路径验证结果应形成 `ReviewRecord`
- 进入 `Ready for Delivery` 前必须有 `DeliverySummaryRecord`

### 推荐进入 `Waiting Decision` 的场景
- 需要决定是否直接上线止血补丁
- 需要决定是否先回滚或降级
- 需要决定是否同时处理历史异常订单

---

## 推荐流转（示意）
Tech Lead -> Waiting Decision -> Dev -> QA / Review -> Ready for Delivery -> Done

> 说明：hotfix 样例必须显式体现拍板、风险和收口；
> 实际推进以 `status + action + record` 体系为准，而不是把外部 approver 写成系统内默认角色。
