# product/task-status-guards.md

## 目标

定义第一阶段任务状态机与状态守卫规则，确保 `feature` / `bugfix` / `hotfix` 三类任务在系统里不是“随便改状态”，而是按统一约束推进。

这份文档主要回答：

- 系统里有哪些核心状态
- 哪些状态之间允许流转，哪些不允许
- 什么时候必须生成 `decision` / `handoff` / `review` / `delivery summary` 记录
- 什么时候必须停在 `Waiting Decision`
- 什么时候才允许进入 `Ready for Delivery` 与 `Done`
- handoff 与 review 的默认顺序是什么
- 哪些规则属于状态机原则，哪些细节交给 transition 文档

---

## 核心原则

- 状态推进必须由 action 驱动，不允许裸改状态
- 状态是业务结果，不是用户可随便下拉选择的字段
- guard 的目标不是“卡流程”，而是防止跳过必要记录与责任转移
- Phase 1 先收紧，不先追求最大自由度
- **任务 owner 变化默认由 transition 驱动，不由普通字段更新驱动**
- **`Waiting Decision` 是阻塞态；存在未决 decision 时，不得继续普通推进**
- **`In Review` 不是一个抽象标签，而是一个有 reviewer、有 review record 的真实阶段**

---

## 状态总表（Phase 1）

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

## 默认允许的主流转路径

### 主路径

- `Draft -> Ready`
- `Ready -> In Progress`
- `In Progress -> Waiting Handoff`
- `Waiting Handoff -> In Progress`
- `In Progress -> In Review`
- `In Review -> Rework Required`
- `Rework Required -> In Progress`
- `In Review -> Ready for Delivery`
- `Ready for Delivery -> Done`
- `Done -> Archived`

### 拍板分叉

以下状态在满足条件时都允许进入 `Waiting Decision`：

- `Ready`
- `In Progress`
- `Waiting Handoff`
- `In Review`
- `Rework Required`
- `Ready for Delivery`

`Waiting Decision` 允许根据已解决 decision 进入：

- `Ready`
- `In Progress`
- `Cancelled`

### 其他受控分叉

- `Ready for Delivery -> In Progress`（显式 reopen）
- `Draft -> Cancelled`
- `Ready -> Cancelled`
- `In Progress -> Cancelled`
- `Waiting Handoff -> Cancelled`
- `Waiting Decision -> Cancelled`

---

## 非法跳转规则

以下跳转第一阶段默认视为非法，除非后续文档明确扩展：

- `Draft -> Done`
- `Draft -> Ready for Delivery`
- `Ready -> Done`
- `In Progress -> Done`
- `Waiting Decision -> Done`
- `Rework Required -> Done`
- `Cancelled -> 任何活跃状态`
- `Archived -> 任何活跃状态`

解释：

- 不允许绕过执行、审核、收口直接宣告完成
- 不允许取消或归档后的任务悄悄复活
- 如果确实需要恢复，应新建任务或由系统支持显式 reopen 机制，而不是直接偷改状态

---

## handoff 与 review 的默认顺序（Phase 1）

这是 Phase 1 必须收死的一条规则。

### 默认规则

当开发角色把任务交给审核角色时，默认顺序是：

1. `submit_handoff`
2. `accept_handoff`
3. 任务重新回到 `In Progress`，但 owner 已变为审核侧
4. 审核侧再执行 `start_review`
5. 任务进入 `In Review`

### 为什么这样定

这样可以把三件事拆清：

- 责任转移：由 `HandoffRecord` 表达
- 审核开始：由 `ReviewRecord` 表达
- 审核结论：由 review 结果表达

### 允许直接 `In Progress -> In Review` 的唯一前提

若当前 `TaskInstance.current_owner` 已经是 review-capable 角色，即：

- `qa_review_agent`
- `tech_lead_agent`

则允许直接执行 `start_review`。

### 明确禁止

以下情况默认不允许：

- `dev_agent` 在仍持有 owner 时直接把任务送入正式 `In Review`
- 用 `start_review` 同时隐式完成 owner 迁移
- 跳过 `accept_handoff` 就把 reviewer 视为已正式接手

---

## 每个状态的进入条件与退出条件

### Draft

#### 进入条件

- 任务被创建但未补齐最小字段

#### 退出条件

- 标题、目标、范围、验收标准、任务类型等最小字段已补齐，转 `Ready`
- 或明确放弃，转 `Cancelled`

---

### Ready

#### 进入条件

- 基础字段完整
- 任务类型明确
- 有可执行范围

#### 退出条件

- 计划负责人明确并开始执行，转 `In Progress`
- 若有关键争议，转 `Waiting Decision`
- 若任务终止，转 `Cancelled`

---

### In Progress

#### 进入条件

- 当前任务已有明确 `current_owner`
- 当前阶段开始处理

#### 退出条件

- 当前阶段产出完成并提交给下一角色，转 `Waiting Handoff`
- 当前 owner 已是审核侧且正式开始审核，转 `In Review`
- 若遇到关键拍板问题，转 `Waiting Decision`
- 若任务终止，转 `Cancelled`

---

### Waiting Handoff

#### 进入条件

- 当前负责人已提交阶段产出
- 已生成有效 `HandoffRecord`
- 下一责任人尚未明确接收或尚未开始

#### 退出条件

- 下一责任人明确接手，转 `In Progress`
- 若交接中暴露拍板问题，转 `Waiting Decision`
- 若任务终止，转 `Cancelled`

---

### Waiting Decision

#### 进入条件

- 缺少必要拍板
- 范围存在争议
- 存在高风险操作
- 缺少关键资源 / 权限 / 外部确认
- 是否继续、如何继续不能由当前角色单独判断
- 已生成 active `DecisionRecord`

#### 退出条件

- active `DecisionRecord` 已被 resolve
- 再通过显式 transition 恢复到 `Ready`、`In Progress` 或 `Cancelled`

#### 明确说明

仅仅 `DecisionRecord` 被 approve / reject，**不会自动让任务离开** `Waiting Decision`。

---

### In Review

#### 进入条件

- 当前 owner 已经是审核侧
- 已提交审核开始动作
- 已生成 `ReviewRecord(review_status=started, result=pending)`

#### 退出条件

- 审核不通过，转 `Rework Required`
- 审核通过且收口条件满足，转 `Ready for Delivery`
- 审核中发现拍板问题，转 `Waiting Decision`

---

### Rework Required

#### 进入条件

- 审核或回归明确未通过
- 已输出返工意见
- 已生成 `ReviewRecord(result=rejected)`

#### 退出条件

- 返工开始，转 `In Progress`
- 返工方向仍不明确，转 `Waiting Decision`

---

### Ready for Delivery

#### 进入条件

- 审核通过
- 已存在最新一条有效 `ReviewRecord(result=passed)`
- 关键交付物齐全
- 已生成有效 `DeliverySummaryRecord`
- 适用的回归检查已完成

#### 退出条件

- 正式完成交付，转 `Done`
- 交付前重新打开，转 `In Progress`
- 发现仍需拍板事项，转 `Waiting Decision`

---

### Done

#### 进入条件

- 任务已完成交付
- 当前阶段已收口

#### 退出条件

- 进入归档，转 `Archived`

---

### Cancelled

#### 进入条件

- 任务被明确取消
- 当前阶段不再继续

#### 退出条件

- Phase 1 默认无普通退出路径

---

### Archived

#### 进入条件

- `Done` 任务被归档
- 不再参与活跃看板与主操作面

#### 退出条件

- Phase 1 默认无普通退出路径

---

## 记录必填场景

### 1. decision record 必填场景

以下场景必须生成 `DecisionRecord`：

- `request_decision`

最少字段建议包含：

- `decisionTopic`
- `decisionQuestion`
- `options`
- `recommendedOption`
- `riskSummary`
- `approverRoleKey`

---

### 2. handoff record 必填场景

以下场景必须生成 `HandoffRecord`：

- `submit_handoff`

最少字段建议包含：

- `fromRole`
- `toRole`
- `handoffSummary`
- `deliveredArtifacts`

---

### 3. review record 必填场景

以下场景必须生成 `ReviewRecord`：

- `start_review`
- `reject_to_rework`
- `mark_ready_for_delivery`

#### 最少字段建议

- `reviewer`
- `reviewType`
- `reviewStatus`
- `result`
- `checklistSummary`
- `issuesFound`
- `nextAction`
- `reviewedAt`

#### 统一约束

- `start_review` 必须生成 `ReviewRecord(review_status=started, result=pending)`
- `reject_to_rework` 必须生成或落地一条 `ReviewRecord(result=rejected)`
- `mark_ready_for_delivery` 必须带出审核通过结论，生成或落地一条 `ReviewRecord(result=passed)`

---

### 4. delivery summary record 必填场景

以下场景必须生成 `DeliverySummaryRecord`：

- `mark_ready_for_delivery`

最少字段建议包含：

- `changeSummary`
- `affectedScope`
- `validationSummary`
- `remainingRisks`

---

## 状态阻塞规则

### 1. 未决 decision 阻塞

若存在 active 且未 resolve 的 `DecisionRecord`，默认不允许：

- `start_review`
- `mark_ready_for_delivery`
- `complete_delivery`
- `restart_rework`
- 其他继续推进的普通 action

除非动作本身就是：

- `resume_after_decision`
- `cancel_task`

---

### 2. 缺少 handoff 阻塞

若当前想把责任从一个角色交给另一个角色，但不存在有效 `HandoffRecord`，不允许通过其他接口偷改 owner。

---

### 3. 缺少 review 结论阻塞

若任务想进入 `Ready for Delivery`，但没有最新有效 `ReviewRecord(result=passed)`，必须阻塞。

---

### 4. 缺少 delivery summary 阻塞

若任务想进入 `Ready for Delivery`，但没有有效 `DeliverySummaryRecord`，必须阻塞。

---

## 与 transition 文档的边界

这份文档负责：

- 状态枚举
- 主允许路径
- 非法跳转原则
- 进入 / 退出条件
- 哪些记录是状态推进前置条件

这份文档不重复定义：

- 每个 action 的 payload 结构
- 角色权限矩阵
- owner 如何变化
- 错误码
- 前端动作按钮建议

以上内容统一以：

- `product/task-transition-api-and-actions.md`

为准。

---

## 当前阶段统一结论

Phase 1 的状态机必须坚持三件事：

1. **状态推进靠 action，不靠状态下拉框**
2. **责任迁移靠 handoff，不靠 PATCH owner**
3. **审核收口靠 review + delivery summary，不靠口头默认通过**

只要这三条不破，Codex 在实现时就不会轻易写出“双轨流转”和“隐式跳状态”的代码。
