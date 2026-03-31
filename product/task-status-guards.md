# product/task-status-guards.md

## 目标

定义第一阶段任务状态机与状态守卫规则，确保 `feature` / `bugfix` / `hotfix` 三类任务在系统里不是“随便改状态”，而是按统一约束推进。

---

## 核心原则

- 状态推进必须由 action 驱动，不允许裸改状态
- 状态是业务结果，不是用户可任意下拉选择的字段
- guard 的目标不是卡流程，而是防止跳过必要记录与责任转移
- owner 变化默认由 transition 驱动，不由普通 PATCH 驱动
- `Waiting Decision` 是阻塞态，存在未决 decision 时，不得继续普通推进
- `In Review` 是真实审核阶段，必须有 reviewer 与 `ReviewRecord`

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

以下跳转在 Phase 1 默认非法：

- `Draft -> Done`
- `Draft -> Ready for Delivery`
- `Ready -> Done`
- `In Progress -> Done`
- `Waiting Decision -> Done`
- `Rework Required -> Done`
- `Cancelled -> 任何活跃状态`
- `Archived -> 任何活跃状态`

---

## handoff 与 review 的默认顺序

这是 Phase 1 必须收死的一条规则。

### 默认规则

当执行角色把任务交给审核角色时，默认顺序是：

1. `submit_handoff`
2. `accept_handoff`
3. 任务回到 `In Progress`，但 owner 已变为审核侧
4. 审核侧执行 `start_review`
5. 任务进入 `In Review`

### 为什么这样定

这样可以把三件事拆清：

- 责任转移：由 `HandoffRecord` 表达
- 审核开始：由 `ReviewRecord(result=pending)` 表达
- 审核结论：由 review pass / reject 表达

### 允许直接 `In Progress -> In Review` 的唯一前提

若当前 `TaskInstance.currentOwner` 已经是 review-capable 角色，即：

- `qa_review_agent`
- `tech_lead_agent`

则允许直接执行 `start_review`。

### 明确禁止

- `dev_agent` 在仍持有 owner 时直接送入正式 `In Review`
- 用 `start_review` 同时隐式完成 owner 迁移
- 跳过 `accept_handoff` 就把 reviewer 视为已正式接手

---

## 各状态进入 / 退出条件

### `Draft`

进入条件：

- 任务已创建但最小字段未补齐

退出条件：

- 最小字段补齐，转 `Ready`
- 明确放弃，转 `Cancelled`

### `Ready`

进入条件：

- 基础字段完整
- 任务类型明确
- 有可执行范围

退出条件：

- 开始执行，转 `In Progress`
- 命中决策门，转 `Waiting Decision`
- 明确终止，转 `Cancelled`

### `In Progress`

进入条件：

- 当前任务已有明确 `currentOwner`
- 当前阶段开始处理

退出条件：

- 当前阶段产出完成并交给下一角色，转 `Waiting Handoff`
- 当前 owner 已是审核侧并正式开始审核，转 `In Review`
- 命中决策门，转 `Waiting Decision`
- 明确终止，转 `Cancelled`

### `Waiting Handoff`

进入条件：

- 当前负责人已提交阶段产出
- 已生成有效 `HandoffRecord`
- 下一责任人尚未明确接收或尚未开始

退出条件：

- 下一责任人接手，转 `In Progress`
- 命中决策门，转 `Waiting Decision`
- 明确终止，转 `Cancelled`

### `Waiting Decision`

进入条件：

- 缺少必要拍板
- 范围存在争议
- 存在高风险操作
- 缺少关键资源 / 权限 / 外部确认
- 已生成 active `DecisionRecord`

退出条件：

- active `DecisionRecord` 已 resolve
- 再通过显式 transition 恢复到 `Ready`、`In Progress` 或 `Cancelled`

### `In Review`

进入条件：

- 当前 owner 已是审核侧
- 已提交审核开始动作
- 已生成 `ReviewRecord(result=pending)`

退出条件：

- 审核不通过，转 `Rework Required`
- 审核通过且收口条件满足，转 `Ready for Delivery`
- 审核中发现拍板问题，转 `Waiting Decision`

### `Rework Required`

进入条件：

- 审核明确未通过
- 已输出返工意见
- 已生成 `ReviewRecord(result=rejected)`

退出条件：

- 返工开始，转 `In Progress`
- 返工方向仍不明确，转 `Waiting Decision`

### `Ready for Delivery`

进入条件：

- 审核通过
- 已存在最新有效 `ReviewRecord(result=passed)`
- 已生成有效 `DeliverySummaryRecord`
- 适用的回归检查已完成

退出条件：

- 正式完成交付，转 `Done`
- 交付前重新打开，转 `In Progress`
- 发现仍需拍板事项，转 `Waiting Decision`

### `Done`

进入条件：

- 任务已完成交付
- 当前阶段已收口

退出条件：

- 进入归档，转 `Archived`

### `Cancelled`

进入条件：

- 任务被明确取消

退出条件：

- Phase 1 默认无普通退出路径

### `Archived`

进入条件：

- `Done` 任务被归档

退出条件：

- Phase 1 默认无普通退出路径

---

## 记录必填场景

- 发生 handoff 时，必须生成 `HandoffRecord`
- 进入 `Waiting Decision` 时，必须生成 active `DecisionRecord`
- 开始审核、审核驳回、审核通过时，必须生成 `ReviewRecord`
- 进入 `Ready for Delivery` 时，必须同时存在 `DeliverySummaryRecord`

---

## 当前阶段统一结论

Phase 1 的状态机必须坚持：

1. action 驱动状态推进
2. handoff 与 review 是两步，不混写
3. decision resolve 与 task resume 分开
4. `Ready for Delivery` 之前必须有 review pass 与 delivery summary
