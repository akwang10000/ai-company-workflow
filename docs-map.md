# docs-map.md

## 目标

这份文档只负责一件事：

为当前仓库提供清晰、可执行的文档导航地图。

它不替代 README，也不替代产品主真相文档，更不重新定义 API、状态机或动作名。

---

## 当前文档治理口径

当前仓库已经进入 **Phase 1：主真相冻结、按 phase 实现验证** 阶段。

因此文档默认分三层：

### A. 主入口真相层

这些文档共同构成当前 Phase 1 的默认实现入口：

1. `tasks/IMPLEMENTATION-PROGRESS.md`
2. `AGENTS.md`
3. `product/domain-model.md`
4. `product/task-status-guards.md`
5. `product/task-transition-api-and-actions.md`
6. `product/api-contracts.md`
7. `product/implementation-phases.md`
8. `governance/task-schema.md`
9. `governance/decision-gates.md`
10. 对应 `workflows/playbooks/*.md`
11. 对应 `roles/playbooks/*.md`
12. `governance/ready-for-delivery-checklist.md`

### B. 补充参考层

这些文档可以辅助实现，但不再与主入口并列：

- `README.md`
- `docs-map.md`
- `product/screens-and-flows.md`
- `product/canvas-ui-spec.md`
- `workflows/workflow.md`
- `examples/tasks/*.md`

### C. 历史 / 次级设计层

若仓库中存在旧版路线图、旧版模块拆分、旧版总纲或其它尚未收口的说明文档：

- 仅作背景参考
- 不得反向定义当前 Phase 1 的动作、状态、owner 边界与执行顺序

---

## 各文档职责

### `tasks/IMPLEMENTATION-PROGRESS.md`

- 用途：当前唯一动态进度真相
- 什么时候看：每次恢复会话、恢复施工前
- 主要回答：做到哪了、当前阻塞是什么、下一步先做什么

### `AGENTS.md`

- 用途：项目级硬规则
- 什么时候看：确定 Phase 1 范围、冲突优先级、执行准入时
- 主要回答：什么任务可执行、什么情况下必须停下、谁不能反向重定义规则

### `product/domain-model.md`

- 用途：定义运行时对象与主真相边界
- 什么时候看：建实体、建状态容器、处理 owner / record 边界时
- 主要回答：谁是 source of truth，哪些只是投影

### `product/task-status-guards.md`

- 用途：定义状态机与状态守卫
- 什么时候看：实现状态切换、做校验、做前后端按钮展示时
- 主要回答：哪些状态允许进入、退出条件是什么、哪些前置记录必须存在

### `product/task-transition-api-and-actions.md`

- 用途：定义动作、payload、record、副作用、权限与幂等
- 什么时候看：实现 transition endpoint、动作按钮、错误处理时
- 主要回答：每个 action 如何推进任务、写哪些记录、如何改 owner

### `product/api-contracts.md`

- 用途：定义资源 API 与边界
- 什么时候看：实现 REST 契约、区分 PATCH 与 transition 时
- 主要回答：哪些字段能普通更新，哪些只能通过 action 驱动

### `product/implementation-phases.md`

- 用途：定义 phase 与退出标准
- 什么时候看：安排实现顺序、确定当前验收面时
- 主要回答：当前应先实现什么、验证什么、什么算完成

### `governance/task-schema.md`

- 用途：定义治理层任务字段与固定范围
- 什么时候看：写任务单、补字段、做治理校验时
- 主要回答：当前允许哪些 task type、字段该怎么填

### `governance/decision-gates.md`

- 用途：定义必须停机拍板的情形
- 什么时候看：遇到高风险动作、边界冲突、多方案并存时
- 主要回答：何时必须进入 `Waiting Decision`，如何恢复

### `workflows/playbooks/*.md`

- 用途：把主真相落成各任务类型执行手册
- 什么时候看：进入具体 `feature / bugfix / hotfix` 任务执行时
- 主要回答：当前阶段允许什么动作、要补什么记录、交给谁

### `roles/playbooks/*.md`

- 用途：把主真相落成角色动作手册
- 什么时候看：当前 owner 已明确时
- 主要回答：当前角色在哪些状态能做什么、产出什么、何时 handoff、何时 request decision

### `governance/ready-for-delivery-checklist.md`

- 用途：定义收口前最低检查项
- 什么时候看：准备进入 `Ready for Delivery` 时
- 主要回答：哪些记录和说明必须齐

### `product/screens-and-flows.md`

- 用途：定义页面清单、页面跳转、页面文案映射
- 什么时候看：做页面规划、动作展示与端侧职责拆分时
- 主要回答：页面之间如何走、默认给用户看什么
- 不负责：反向定义状态机与 transition 真相

### `product/canvas-ui-spec.md`

- 用途：定义画布布局、节点与连线视觉规则
- 什么时候看：实现画布页时
- 主要回答：画布怎么摆、节点怎么显示、信息如何分层
- 不负责：反向定义动作名、状态机或 workflow 主链路

### `workflows/workflow.md`

- 用途：给第一次看项目的人提供 workflow 总览
- 什么时候看：先理解为什么需要 workflow / role / handoff / review 时
- 主要回答：当前 Phase 1 的协作模型是什么
- 不负责：替代具体 playbook

### `examples/tasks/*.md`

- 用途：给任务单编写与演示提供样例
- 什么时候看：需要参考 feature / bugfix / hotfix 任务写法时
- 主要回答：一个合格任务单大概长什么样
- 不负责：定义运行时真相

---

## 默认阅读路径

### 路径 A：第一次看项目的人

1. `README.md`
2. `AGENTS.md`
3. `docs-map.md`
4. `product/implementation-phases.md`
5. `product/domain-model.md`
6. `product/task-status-guards.md`
7. `product/task-transition-api-and-actions.md`

### 路径 B：准备按文档施工的 Codex / 开发者

1. `tasks/IMPLEMENTATION-PROGRESS.md`
2. `AGENTS.md`
3. `product/domain-model.md`
4. `product/task-status-guards.md`
5. `product/task-transition-api-and-actions.md`
6. `product/api-contracts.md`
7. `product/implementation-phases.md`
8. `governance/task-schema.md`
9. `governance/decision-gates.md`
10. 对应 `workflows/playbooks/*.md`
11. 对应 `roles/playbooks/*.md`
12. `governance/ready-for-delivery-checklist.md`
13. 当前任务单 / 改动上下文

### 路径 C：做页面与前端交互的人

1. 先走路径 B 的前 7 项
2. `product/screens-and-flows.md`
3. `product/canvas-ui-spec.md`
4. 对应 workflow / role playbook

---

## 当前统一结论

当前 Phase 1 的文档治理目标不是“文档越多越全”，而是：

- 只有一套主入口
- 只有一套动作名
- 只有一套状态机口径
- workflow / role 文档消费主真相，不再另起一套真相
