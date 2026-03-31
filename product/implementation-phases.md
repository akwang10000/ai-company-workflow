# product/implementation-phases.md

## 目标

定义当前项目进入真实实现前后的 phase 划分、每个 phase 的目标、验证方式与退出标准。

本文件是 **实现节奏蓝图**，不是运行时主真相。

---

## 总体原则

- 当前只做 Phase 1：软件公司模板 / 研发团队版
- 当前先跑通研发任务主链路，不扩多行业
- 先冻结主真相，再做实现验证
- 文档发现问题时，优先补歧义，不再扩写第二套总纲

---

## Phase 0：文档收口与施工基线期

### 目标

- 收死 Phase 1 固定范围
- 收死主对象、主状态机、主 transition 口径
- 统一阅读入口和执行顺序
- 把 workflow / role 文档从说明书升级为执行手册

### 必须达成

- `domain-model.md` 明确任务、记录、workflow 投影边界
- `task-status-guards.md` 明确状态机与 guard
- `task-transition-api-and-actions.md` 明确 action / payload / side effect
- `api-contracts.md` 明确资源接口边界
- `AGENTS.md / IMPLEMENTATION-PROGRESS.md / execution-order.md / codex-delivery-rules.md` 共享同一套入口顺序

### Validation

- 复审结论为：可开始实现，但仍不把自己称为可无歧义自动执行

### Exit Criteria

- 当前主真相层可视为冻结
- 执行入口无第二套顺序

---

## Phase 1：模板启动与任务骨架实现期

### 目标

先打通从模板启动到任务创建与查看的最小骨架。

### 范围

- 模板列表与模板详情
- 基于模板实例化 workspace
- workspace 基本首页
- 任务创建、查看、普通 PATCH（非流转字段）
- 固定任务类型：`feature / bugfix / hotfix`

### Validation

- 能基于“软件公司模板”实例化一个 workspace
- 能创建一个带最小字段的任务
- 能查看任务详情、当前 owner、当前状态
- 普通 PATCH 无法直接改 `status / currentOwner / nextOwner`

### Exit Criteria

- 模板启动链路可跑通
- 任务基础 CRUD 可用
- API 与前端骨架对齐

---

## Phase 2：transition 引擎与记录闭环期

### 目标

打通 action-driven task transition 与关键记录闭环。

### 范围

- `POST /api/tasks/{taskId}/transition`
- `request_decision`
- `resume_after_decision`
- `submit_handoff`
- `accept_handoff`
- `start_review`
- `reject_to_rework`
- `mark_ready_for_delivery`
- `complete_delivery`
- 关键 records：`HandoffRecord / DecisionRecord / ReviewRecord / DeliverySummaryRecord / ExecutionLog`

### Validation

必须打通以下 3 条主链路：

1. `submit_handoff -> accept_handoff -> start_review`
2. `request_decision -> resolve decision -> resume_after_decision`
3. `start_review -> reject_to_rework` 与 `start_review -> mark_ready_for_delivery`

### Exit Criteria

- 前端不再直接发裸状态
- owner 变化由 transition side effect 驱动
- decision resolve 与 task resume 明确分离

---

## Phase 3：workflow 视图与轻操作期

### 目标

提供 workflow / canvas 的查看能力，以及围绕当前任务的轻操作入口。

### 范围

- workflow / task board / dashboard 聚合视图
- 当前状态、当前 owner、下一建议动作展示
- 最近一条 handoff / decision / review / delivery summary 摘要展示

### Validation

- 用户能快速看出当前卡在哪
- 用户能看到当前可执行动作，而不是理解整套内部状态机

### Exit Criteria

- workflow 可视化与 action 模型对齐
- 任务详情页能够支撑实际操作

---

## Phase 4：实现验证后的文档回补期

### 目标

只做实现验证后暴露出的边界修补，不重开新总纲。

### 规则

发现问题时按以下顺序处理：

1. 先判断是不是文档歧义
2. 若是歧义，只允许补一处真相，不允许再开新总纲
3. 若不是歧义，而是范围过大，优先缩小当前 phase 交付面
4. 若是未来需求，显式标记为后续 phase，不回写当前 Phase 1 主真相

---

## 当前阶段统一结论

当前阶段已经不再适合继续扩大文档面。

更合理的路线是：

- 以当前 product 主文档为冻结基线
- 以 workflow / role 执行手册化补齐低猜测执行
- 尽快进入真实实现验证
