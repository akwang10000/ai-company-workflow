# examples/tasks/bugfix-example.md

## 目标
提供一条可直接复用的 **bugfix 任务单样例**，并与当前 Phase 1 的 action-driven / record-driven 任务推进口径保持一致。

---

> 说明：以下 `current_owner` / `next_owner` 为治理层人类可读写法；
> 正式运行时字段、owner 真相与可更新边界，以 `product/domain-model.md`、`product/api-contracts.md`、`product/task-transition-api-and-actions.md` 为准。

## 示例任务
- task_id: `BUG-001`
- title: `修复后台用户列表页切换分页后筛选条件丢失`
- task_type: `bugfix`
- scenario: `engineering`
- goal: `修复后台用户列表页在切换分页后丢失筛选条件的问题`
- business_context: `运营和客服在筛查用户列表时需要频繁切页，筛选条件丢失会导致重复筛查和误操作。`
- problem_statement: `切换分页或每页条数后，当前筛选条件被清空，不符合预期体验。`
- expected_outcome: `用户在切换分页、切换每页条数、返回列表页时，筛选条件按设计保持。`
- non_goals:
  - `本次不重构整个列表页状态管理`
  - `本次不调整与本 bug 无关的筛选交互`
- status: `Draft`
- priority: `high`
- current_owner: `PM Agent`
- next_owner: `Tech Lead Agent`
- known_risks:
  - `修复后可能影响现有排序与分页逻辑`
  - `状态持久化策略若处理不当，可能引入新的回归问题`

---

## 验收标准示例
- 选择筛选条件后切换分页，条件不会丢失
- 切换每页条数后筛选条件仍保持
- 返回列表页时筛选条件按设计保留
- 不影响现有排序和分页能力

---

## Done 判定示例
- 问题复现路径已明确
- 修复方案已确认
- 核心回归项已执行并通过
- 审核结论明确
- 已形成交付摘要并进入最终确认

---

## 推荐初始推进方式
- 推荐初始状态：`Draft`
- 推荐第一步 action：`submit_ready`
- 推荐第一步接手：`PM Agent` 或 `Tech Lead Agent`

### 推荐关键记录
- Tech Lead 判断修复范围时，如涉及是否升级 hotfix，可生成 `DecisionRecord`
- Dev 提交 QA 前，应有 handoff 摘要
- QA 若驳回返工，应形成 `ReviewRecord`
- 进入 `Ready for Delivery` 前，应有回归结论与 `DeliverySummaryRecord`

### 推荐进入 `Waiting Decision` 的场景
- 修复方案可能影响共享列表组件，超出当前 bugfix 范围
- 问题严重度上升，需判断是否升级为 hotfix

---

## 推荐流转（示意）
PM / Tech Lead -> Dev -> QA / Review -> Ready for Delivery -> Done

> 说明：bugfix 的真正主线不只是角色箭头，还应显式经过 review / regression / delivery summary 等记录环节。
