# product/task-status-guards.md

## 目标
定义第一阶段任务状态机与状态守卫规则，确保 feature / bugfix / hotfix 三类任务在系统里不是“随便改状态”，而是按统一约束推进。

这份文档主要回答：
- 系统里有哪些核心状态
- 哪些状态之间允许流转，哪些不允许
- 什么时候必须生成 decision / handoff / review / delivery summary 记录
- 什么时候必须停在 `Waiting Decision`
- 什么时候才允许进入 `Ready for Delivery` 与 `Done`

---

## 适用范围
本规则适用于第一阶段所有任务运行态，至少覆盖：
- `feature`
- `bugfix`
- `hotfix`

第一阶段允许不同任务类型在流程细节上略有差异，但不允许脱离统一状态守卫体系各跑各的。

---

## 设计原则

### 1. 状态必须可解释
每个状态都要回答清楚：
- 当前任务在做什么
- 当前由谁接手
- 下一步要满足什么条件才能往下走

### 2. 状态流转必须可追溯
关键流转不只改一个字段，还要能追到是谁、基于什么信息、把任务推进到了哪里。

### 3. 停机优先于误推进
遇到重大不确定性、范围争议、高风险动作、缺少拍板时，优先进入 `Waiting Decision`，而不是硬推进。

### 4. 收口必须有门槛
不是“代码改了”就能 Done，必须先满足 review、回归、摘要、交付等条件。

---

## 核心状态定义
第一阶段建议统一采用以下核心状态：

### 1. Draft
任务刚创建，信息还不完整，尚未进入正式执行。

### 2. Ready
任务信息已达到最小执行要求，等待进入正式流转。

### 3. In Progress
当前有明确负责人正在处理该任务。

### 4. Waiting Handoff
当前执行人已完成本阶段输出，等待交接给下一个角色。

### 5. Waiting Decision
任务因关键问题、争议、风险、授权缺失或方向待拍板而暂停，等待人类或指定 approver 决策。

### 6. In Review
任务已进入审核、验收、回归验证或技术复核环节。

### 7. Rework Required
审核未通过，需要返工。

### 8. Ready for Delivery
任务已满足最小交付条件，等待最终确认、发布、展示或归档前收口。

### 9. Done
任务已完成且收口结束。

### 10. Archived
任务已归档，不再作为活跃任务参与流转。

### 11. Cancelled
任务被明确取消，不再继续推进。

---

## 合法状态流转表

| From | To | 是否允许 | 说明 |
|---|---|---|---|
| Draft | Ready | 允许 | 基础字段补齐后进入可执行态 |
| Draft | Cancelled | 允许 | 明确放弃该任务 |
| Ready | In Progress | 允许 | 指定负责人后开始执行 |
| Ready | Waiting Decision | 允许 | 开工前即存在方向/资源/权限问题 |
| In Progress | Waiting Handoff | 允许 | 当前阶段产出已完成，等待交接 |
| In Progress | In Review | 允许 | 当前阶段完成后直接进入审核 |
| In Progress | Waiting Decision | 允许 | 执行中遇到必须拍板问题 |
| In Progress | Cancelled | 条件允许 | 明确终止任务时 |
| Waiting Handoff | In Progress | 允许 | 下一责任人接手成功 |
| Waiting Handoff | Waiting Decision | 允许 | 交接中发现需拍板问题 |
| Waiting Handoff | Cancelled | 条件允许 | 任务被终止 |
| Waiting Decision | Ready | 允许 | 决策后回到待执行 |
| Waiting Decision | In Progress | 允许 | 决策后直接继续执行 |
| Waiting Decision | Cancelled | 允许 | 决策结果为终止 |
| In Review | Rework Required | 允许 | 审核未通过 |
| In Review | Ready for Delivery | 允许 | 审核通过且满足收口条件 |
| In Review | Waiting Decision | 允许 | 审核中遇到拍板问题 |
| Rework Required | In Progress | 允许 | 返工开始 |
| Rework Required | Waiting Decision | 允许 | 返工方向需拍板 |
| Ready for Delivery | Done | 允许 | 交付完成或最终确认完成 |
| Ready for Delivery | In Progress | 条件允许 | 收口前发现问题重新打开 |
| Ready for Delivery | Waiting Decision | 允许 | 最终确认阶段仍需拍板 |
| Done | Archived | 允许 | 完成后归档 |

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

## 每个状态的进入条件与退出条件

### Draft
进入条件：
- 任务被创建但未补齐最小字段

退出条件：
- 任务标题、背景、目标、范围、验收标准、任务类型等最小字段已补齐，转 `Ready`
- 或明确放弃，转 `Cancelled`

### Ready
进入条件：
- 基础字段完整
- 任务类型明确
- 有可执行范围

退出条件：
- 指派负责人并开始执行，转 `In Progress`
- 若有关键争议，转 `Waiting Decision`

### In Progress
进入条件：
- 当前任务已有明确 owner
- 当前阶段开始处理

退出条件：
- 当前阶段产出完成，进入 `Waiting Handoff` 或 `In Review`
- 若遇到关键拍板问题，转 `Waiting Decision`
- 若任务终止，转 `Cancelled`

### Waiting Handoff
进入条件：
- 当前负责人已提交阶段产出
- 下一责任人尚未明确接收或尚未开始

退出条件：
- 下一责任人明确接手，转 `In Progress`
- 若交接中暴露拍板问题，转 `Waiting Decision`

### Waiting Decision
进入条件：
- 缺少必要拍板
- 范围存在争议
- 存在高风险操作
- 缺少关键资源 / 权限 / 外部确认
- 是否继续、如何继续不能由当前角色单独判断

退出条件：
- 已有明确决策记录
- 根据决策内容回到 `Ready`、`In Progress` 或 `Cancelled`

### In Review
进入条件：
- 实现、修复或方案输出已提交审核
- 当前阶段需要 review / QA / 回归验证

退出条件：
- 审核不通过，转 `Rework Required`
- 审核通过且收口条件满足，转 `Ready for Delivery`
- 审核中发现拍板问题，转 `Waiting Decision`

### Rework Required
进入条件：
- 审核或回归明确未通过
- 已输出返工意见

退出条件：
- 返工开始，转 `In Progress`
- 返工方向仍不明确，转 `Waiting Decision`

### Ready for Delivery
进入条件：
- 审核通过
- 关键交付物齐全
- 收口摘要已准备
- 适用的回归检查已完成

退出条件：
- 完成交付确认，转 `Done`
- 若发现问题重新打开，转 `In Progress`
- 若最终确认仍需拍板，转 `Waiting Decision`

### Done
进入条件：
- 已完成交付或最终确认
- 任务对外输出已闭环
- 必要留痕已完成

退出条件：
- 仅允许转 `Archived`

### Archived
进入条件：
- Done 任务完成归档
- 不再参与活跃工作流

退出条件：
- 第一阶段不支持直接退出

### Cancelled
进入条件：
- 任务明确停止
- 决策结果为取消或无需继续

退出条件：
- 第一阶段不支持直接恢复

---

## 必填记录规则
关键状态流转不能只改状态，以下记录为第一阶段必填：

### 1. handoff record 必填场景
以下场景进入 `Waiting Handoff` 时必须生成 `HandoffRecord`：
- PM -> Tech Lead
- Tech Lead -> Dev
- Dev -> QA / Review
- QA / Review -> Ops / Release
- 任意角色把任务明确移交给下一角色

最少字段建议包含：
- fromRole
- toRole
- handoffSummary
- deliveredArtifacts
- risksOrNotes
- createdAt

### 2. decision record 必填场景
以下场景进入 `Waiting Decision` 时必须生成 `DecisionRecord`：
- 范围有争议
- 要做高风险改动
- 需要额外资源 / 权限
- 是否延期、降级、砍范围需要拍板
- hotfix 是否直接上线需要拍板

最少字段建议包含：
- decisionTopic
- decisionQuestion
- options
- recommendedOption
- riskSummary
- requestedBy
- requestedAt
- decisionResult（决策后补齐）

### 3. review record 必填场景
以下场景进入 `In Review` 或从 `In Review` 退出时必须生成 `ReviewRecord`：
- QA / Review 开始验收
- 技术评审开始
- 审核通过
- 审核驳回并要求返工

最少字段建议包含：
- reviewer
- reviewType
- checklistSummary
- result
- issuesFound
- nextAction
- reviewedAt

### 4. delivery summary 必填场景
以下场景进入 `Ready for Delivery` 时必须生成交付摘要：
- feature 即将交付
- bugfix 完成回归即将对外确认
- hotfix 止血完成即将收口

最少字段建议包含：
- changeSummary
- affectedScope
- validationSummary
- remainingRisks
- releaseNotesLink 或摘要文本
- preparedBy
- preparedAt

---

## Waiting Decision 触发条件
以下任一条件成立时，系统应支持或建议进入 `Waiting Decision`：
- 任务目标或范围无法继续明确
- 存在多个方案且取舍影响较大
- 涉及数据库结构、接口兼容、权限、线上风险等高影响修改
- 验收标准被推翻或显著变化
- 当前角色没有权限继续推进
- hotfix 是否回滚 / 是否绕过常规流程需要拍板
- 返工次数已超阈值，需要确认是否继续投入

原则：
**拿不准且影响大的，不硬冲，先停机等拍板。**

---

## Ready for Delivery 进入条件
任务只有满足以下条件，才允许进入 `Ready for Delivery`：
- 当前实现或修复已完成
- 适用的 review / QA / regression 已完成
- 若发生过返工，返工项已关闭或明确接受残留风险
- 当前状态、负责人、关键输出物完整可追溯
- 已生成 delivery summary
- 不再缺少关键决策

补充要求：
- `feature` 侧重验收标准达成
- `bugfix` 侧重缺陷复现关闭与回归范围说明
- `hotfix` 侧重关键路径验证、风险说明、后续复盘入口

---

## Done / Archived 进入条件

### Done 进入条件
只有满足以下条件，才允许进入 `Done`：
- 已在 `Ready for Delivery`
- 已完成最终确认、交付、发布或等价收口动作
- 对外输出已准备完毕
- 必填记录已补齐

### Archived 进入条件
只有满足以下条件，才允许进入 `Archived`：
- 任务已是 `Done`
- 已不需要继续跟踪活跃进展
- 必要历史记录可被检索

---

## feature / bugfix / hotfix 的额外守卫差异

### feature
额外关注：
- 验收标准是否明确
- 范围是否失控
- 是否需要产品确认范围裁剪

建议：
- feature 进入 `Ready` 前，验收标准必须可读
- feature 进入 `Ready for Delivery` 前，应有明确 change summary

### bugfix
额外关注：
- 是否可复现
- 根因是否基本明确
- 回归范围是否说明清楚

建议：
- bugfix 进入 `In Progress` 前，至少应有问题现象和影响描述
- bugfix 进入 `Ready for Delivery` 前，应有修复说明与回归摘要

### hotfix
额外关注：
- 时效性与风险平衡
- 是否允许绕过部分常规流程
- 是否要在 Done 后补 postmortem

建议：
- hotfix 可缩短部分文书，但不能跳过关键 decision / review 留痕
- hotfix 若有残留风险，必须在 delivery summary 中明确写出
- hotfix 完成后应支持挂接 postmortem 任务或记录

---

## API / 后端约束建议
为了让状态守卫真正落地，后端建议：
- 不允许前端任意写入目标状态
- 所有状态变更通过显式 transition API 或 command 完成
- transition 时做合法性校验
- transition 成功后自动创建对应记录或校验记录存在
- 对非法跳转返回明确错误码与错误原因

建议接口能力至少包括：
- `POST /tasks/{id}/transition`
- `POST /tasks/{id}/handoffs`
- `POST /tasks/{id}/decisions`
- `POST /tasks/{id}/reviews`
- `POST /tasks/{id}/delivery-summary`

---

## 前端约束建议
前端不应把状态切换做成裸下拉框，而应：
- 基于当前状态只展示可用动作
- 对需要补充记录的流转先弹出表单
- 对非法跳转给出明确说明
- 在任务详情与画布节点中展示“为什么卡在这里”

---

## Codex 实施建议
如果 Codex 要按本文档施工，建议顺序：
1. 先固化枚举与合法流转矩阵
2. 再为 transition API 增加守卫校验
3. 再补 handoff / decision / review / delivery summary 的持久化结构
4. 再在前端把“状态变更”改成“动作驱动 + 记录表单驱动”
5. 最后补不同任务类型的差异化校验

---

## 一句话总结
状态守卫文档的核心作用，是把“任务推进”这件事从随手改字段，收成**有门槛、有留痕、有停机点、有收口标准的统一状态机**。
