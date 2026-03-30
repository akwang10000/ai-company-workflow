# examples/tasks/feature-example.md

## 目标
提供一条可直接复用的 **feature 任务单样例**，并与当前 Phase 1 的 action-driven / record-driven 任务推进口径保持一致。

---

> 说明：以下 `current_owner` / `next_owner` 为治理层人类可读写法；
> 正式运行时字段、owner 真相与可更新边界，以 `product/domain-model.md`、`product/api-contracts.md`、`product/task-transition-api-and-actions.md` 为准。

## 示例任务
- task_id: `FEAT-001`
- title: `新增后台用户封禁记录查询页`
- task_type: `feature`
- scenario: `engineering`
- goal: `新增后台用户封禁记录查询页，降低运营人工查库支持成本`
- business_context: `运营当前遇到用户投诉时，需要研发临时查库确认封禁历史，支持成本高且响应慢。`
- problem_statement: `缺少运营自助查询封禁记录的后台能力。`
- expected_outcome: `运营主管可在后台按用户 ID 和时间范围查询封禁记录，无需研发临时查库。`
- non_goals:
  - `第一版不支持直接解封`
  - `第一版不支持修改历史封禁记录`
- status: `Draft`
- priority: `medium`
- current_owner: `PM Agent`
- next_owner: `Tech Lead Agent`
- known_risks:
  - `若查询范围过大，列表性能可能不稳定`
  - `权限边界不清时可能需要补充拍板`

---

## 验收标准示例
- 支持按用户 ID 查询封禁记录
- 支持按时间范围筛选
- 展示封禁原因、操作人、操作时间
- 无权限用户不可访问
- 页面结果展示正常

---

## Done 判定示例
- 需求边界已澄清
- 技术方案已确认
- 实现与自测已完成
- 审核 / 回归通过
- 已形成交付摘要并进入最终确认

---

## 推荐初始推进方式
- 推荐初始状态：`Draft`
- 推荐第一步 action：`submit_ready`
- 推荐第一步接手：`PM Agent`

### 推荐关键记录
- PM -> Tech Lead：若范围已明确，可准备 handoff 摘要
- 若权限边界存在争议：进入 `Waiting Decision` 并生成 `DecisionRecord`
- Dev -> QA / Review：进入审核前应有 handoff 记录
- 进入 `Ready for Delivery` 前应有 `DeliverySummaryRecord`

### 推荐进入 `Waiting Decision` 的场景
- 是否允许运营看到全部封禁原因字段存在争议
- 是否需要支持更复杂筛选超出当前范围

---

## 推荐流转（示意）
PM -> Tech Lead -> Dev -> QA / Review -> Ready for Delivery -> Done

> 说明：样例中的“推荐流转”仅用于帮助理解主路径；
> 实际推进以 `status + action + record` 体系为准，而不是只按箭头切角色。
