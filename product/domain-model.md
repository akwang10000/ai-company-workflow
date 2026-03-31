# product/domain-model.md

## 目标

定义 Phase 1：软件公司模板 / 研发团队版 的核心对象边界，明确哪些是模板对象、哪些是运行时对象、哪些是投影对象，以及任务状态与记录的单一真相在哪里。

---

## 建模原则

- 当前只服务 Phase 1 软件研发团队版
- 先明确主对象与边界，不扩终态大而全模型
- 任务业务状态真相只保留一处
- owner 真相只保留一处
- workflow 可视图层与 task 业务状态层分离
- record 是审计与责任转移事实，不是装饰性备注

---

## 模板层对象

### 1. `CompanyTemplate`

代表“第一个软件公司模板”这类最小启动模板。

#### 最小字段建议

- `companyTemplateId`
- `name`
- `summary`
- `includedRoleTemplateIds`
- `includedWorkflowTemplateIds`
- `includedTaskTemplateIds`
- `recommendedStartFlow`
- `status`
- `version`

### 2. `RoleTemplate`

代表一个可复用角色模板。

#### 最小字段建议

- `roleTemplateId`
- `name`
- `roleKey`
- `roleType`
- `department`
- `summary`
- `responsibilities`
- `nonResponsibilities`
- `inputs`
- `outputs`
- `decisionBoundaries`
- `status`
- `version`

### 3. `WorkflowTemplate`

代表 `feature / bugfix / hotfix` 的标准流转骨架。

#### 最小字段建议

- `workflowTemplateId`
- `name`
- `scenario`
- `taskType`
- `summary`
- `entryConditions`
- `nodeDefinitions`
- `edgeDefinitions`
- `defaultRoles`
- `defaultDecisionGates`
- `status`
- `version`

---

## 运行时对象

### 4. `Workspace`

基于模板实例化后的实际协作空间。

#### 最小字段建议

- `workspaceId`
- `companyTemplateId`
- `name`
- `status`
- `memberAssignments`
- `createdAt`

### 5. `RoleAssignment`

代表某个 workspace 内的实际角色分配。

#### 最小字段建议

- `roleAssignmentId`
- `workspaceId`
- `roleKey`
- `displayName`
- `assigneeType`
- `assigneeId`
- `status`

### 6. `TaskInstance`

代表单个任务实例，是 **任务业务状态与 owner 的唯一主真相**。

#### 最小字段建议

- `taskId`
- `workspaceId`
- `taskType`
- `scenario`
- `title`
- `goal`
- `acceptanceCriteria`
- `definitionOfDone`
- `status`
- `priority`
- `plannedOwnerRoleAssignmentId`
- `currentOwnerRoleAssignmentId`
- `nextOwnerRoleAssignmentId`
- `riskLevel`
- `decisionRequired`
- `createdAt`
- `updatedAt`
- `version`

#### 关键说明

- `TaskInstance.status` 是任务业务状态主真相
- `TaskInstance.currentOwner*` / `nextOwner*` 是 owner 主真相
- 任务状态与 owner 的业务变化，必须来自 transition

### 7. `TaskRecord`

任务推进过程中的结构化事实基类。

#### 通用字段建议

- `recordId`
- `taskId`
- `recordType`
- `createdByRoleKey`
- `createdAt`
- `summary`
- `payload`
- `status`

### 8. `HandoffRecord`

表示责任转移事实。

#### 关键字段

- `fromRole`
- `toRole`
- `handoffSummary`
- `deliveredArtifacts`
- `risksOrNotes`
- `acceptedAt`

### 9. `DecisionRecord`

表示等待拍板与拍板结果。

#### 关键字段

- `decisionReason`
- `options`
- `recommendedOption`
- `approver`
- `decisionStatus`（`active / approved / rejected / superseded`）
- `decisionResult`

### 10. `ReviewRecord`

表示审核开始、审核驳回、审核通过等事实。

#### 关键字段

- `reviewType`
- `reviewStatus`
- `result`
- `checklistSummary`
- `issuesFound`
- `returnToRole`
- `reviewedByRole`

#### 关键约束

- `ReviewRecord` 是 append-only
- 不通过覆盖旧 review 来“改历史”
- 最新有效 review 结果用于判断是否可进入 `Ready for Delivery`

### 11. `DeliverySummaryRecord`

表示任务交付前的收口摘要。

#### 关键字段

- `changeSummary`
- `affectedScope`
- `validationSummary`
- `remainingRisks`
- `deliveryTo[]`

### 12. `ExecutionLog`

表示 transition 与关键动作的操作审计日志。

#### 关键字段

- `action`
- `previousStatus`
- `currentStatus`
- `actorRoleKey`
- `comment`
- `idempotencyKey`

---

## 投影 / 视图层对象

### 13. `WorkflowInstance`

代表某个任务或 workspace 在 workflow / 画布上的投影视图。

#### 关键说明

- `WorkflowInstance` 主要服务展示与轻操作
- 它不是任务业务状态主真相
- 不与 `TaskInstance.status` 形成双轨维护

### 14. `TaskActionProjection`

代表前端当前应看到的可执行动作、禁用原因和下一建议动作。

#### 最小字段建议

- `taskId`
- `currentStatus`
- `currentOwnerRoleKey`
- `availableActions`
- `disabledActions`
- `nextSuggestedActions`
- `blockReasons`

---

## 角色边界说明

当前 Phase 1 默认角色：

- `pm_agent`
- `tech_lead_agent`
- `dev_agent`
- `qa_review_agent`
- `ops_release_agent`

辅助参与方：

- `approver`
- `requester`
- `system`

### 关键约束

- `approver` 不是普通常驻执行 owner
- `system` 不是业务角色，不承担普通 handoff

---

## 当前阶段统一结论

当前运行时单一真相必须坚持：

1. `TaskInstance` 是任务状态与 owner 的唯一业务真相
2. `WorkflowInstance` 是视图投影，不与 task 双轨维护状态
3. `HandoffRecord / DecisionRecord / ReviewRecord / DeliverySummaryRecord / ExecutionLog` 是任务推进的结构化事实层
