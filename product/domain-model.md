# product/domain-model.md

## 目标

定义 AI Company Workflow 在 Phase 1 MVP（软件公司模板 / 研发团队版）的核心领域模型，确保：

- 后端建模围绕同一套对象体系推进
- 前端显示状态、owner、动作时有统一来源
- 模板对象与运行时对象严格区分
- task transition 所需记录可直接落成表结构与服务
- Codex / 自动化编码代理不需要自行猜 source of truth

---

## 核心原则

- 先把核心对象收死，再谈实现细节
- 模板对象与运行时实例对象必须分开
- task、workflow、角色、技能不要混成一个大对象
- task transition 相关记录必须显式建模，不要全部塞进日志
- Phase 1 先满足软件公司模板，不急着一次抽象全行业终态
- **TaskInstance 是任务业务状态的主真相来源**
- **WorkflowInstance 是 workflow 视图与节点运行投影，不与 TaskInstance 双轨抢真相**
- **available actions 是运行时投影，不是持久化真相**

---

## Phase 1 角色键统一口径

Phase 1 默认采用以下 `role_key`：

- `pm_agent`
- `tech_lead_agent`
- `dev_agent`
- `qa_review_agent`
- `ops_release_agent`

### 特殊 actor 类型

以下不是普通 workspace 角色模板，但在实现层可能作为 actor 类型出现：

- `approver`
- `system`

约束：

- `approver` 表示“具有拍板权限的主体”，默认可由 workspace owner、PM 或指定审批者承担
- `system` 表示系统自动动作，不对应前端普通用户选择的角色
- `approver` 和 `system` 不应被当成普通 `RoleTemplate` 强行实例化为常驻团队岗位

---

## 第一阶段核心对象总览

### 模板层对象

- `CompanyTemplate`
- `RoleTemplate`
- `WorkflowTemplate`
- `TaskTemplate`
- `SkillPackageTemplate`

### 运行时对象

- `Workspace`
- `RoleAssignment`
- `TaskInstance`
- `WorkflowInstance`
- `DecisionRecord`
- `HandoffRecord`
- `ReviewRecord`
- `DeliverySummaryRecord`
- `ExecutionLog`

### 视图投影对象

- `CanvasNode`
- `CanvasEdge`
- `NodeDetailViewModel`
- `TaskActionProjection`

---

## 一、模板层对象

### 1. `CompanyTemplate`

代表“第一个软件公司模板”这类最小启动模板。

#### 最小字段建议

- `company_template_id`
- `name`
- `summary`
- `included_role_template_ids`
- `included_workflow_template_ids`
- `included_task_template_ids`
- `included_skill_package_template_ids`
- `recommended_start_flow`
- `status`
- `version`

#### 说明

它是 Phase 1 最关键的聚合模板对象。

---

### 2. `RoleTemplate`

代表一个可复用角色模板。

#### 最小字段建议

- `role_template_id`
- `name`
- `role_key`
- `role_type`
- `department`
- `summary`
- `responsibilities`
- `non_responsibilities`
- `inputs`
- `outputs`
- `decision_boundaries`
- `default_skill_package_ids`
- `status`
- `version`

#### 说明

它描述的是一种岗位模板，不是某个具体成员。

---

### 3. `WorkflowTemplate`

代表 feature / bugfix / hotfix 的标准流转骨架。

#### 最小字段建议

- `workflow_template_id`
- `name`
- `scenario`
- `task_type`
- `summary`
- `entry_conditions`
- `node_definitions`
- `edge_definitions`
- `default_roles`
- `default_decision_gates`
- `status`
- `version`

#### 说明

它描述的是流转骨架，不是具体任务实例本身。

---

### 4. `TaskTemplate`

代表可复用的任务单模板或任务样例骨架。

#### 最小字段建议

- `task_template_id`
- `name`
- `task_type`
- `scenario`
- `summary`
- `default_fields`
- `field_rules`
- `default_acceptance_criteria`
- `default_definition_of_done`
- `status`
- `version`

---

### 5. `SkillPackageTemplate`

代表一组可随角色或 workflow 默认挂载的技能包模板。

#### 最小字段建议

- `skill_package_template_id`
- `name`
- `summary`
- `skill_items`
- `compatible_roles`
- `compatible_workflows`
- `default_enabled`
- `version`

---

## 二、运行时对象

### 1. `Workspace`

代表用户基于某个 `CompanyTemplate` 实例化出来的一套工作空间。

#### 最小字段建议

- `workspace_id`
- `name`
- `company_template_id`
- `owner_user_id`
- `members`
- `status`
- `created_at`

---

### 2. `RoleAssignment`

代表某个角色模板在 workspace 中被实例化后的实际角色实例。

#### 最小字段建议

- `role_assignment_id`
- `workspace_id`
- `role_template_id`
- `role_key`
- `display_name`
- `bound_user_id`
- `bound_agent_id`
- `enabled_skill_package_ids`
- `status`

---

### 3. `TaskInstance`

代表某次真实任务。

#### 最小字段建议

- `task_instance_id`
- `task_template_id`
- `workspace_id`
- `title`
- `goal`
- `task_type`
- `scenario`
- `status`
- `priority`
- `severity`
- `planned_owner_role_assignment_id`
- `current_owner_role_assignment_id`
- `next_owner_role_assignment_id`
- `acceptance_criteria`
- `definition_of_done`
- `known_risks`
- `current_workflow_instance_id`
- `created_at`
- `updated_at`
- `version`

#### 补充建议字段

- `latest_review_result`
- `latest_delivery_readiness`
- `latest_decision_status`
- `available_action_keys`（仅允许作为缓存 / 投影字段）

#### Source of truth 规则

- `TaskInstance.status` 是任务业务状态的唯一主真相
- `TaskInstance.current_owner_role_assignment_id` 是任务当前责任人的唯一主真相
- `TaskInstance.next_owner_role_assignment_id` 表示“若需要交接，下一责任人是谁”
- `planned_owner_role_assignment_id` 用于 Draft / Ready 阶段的计划负责人，不等于当前执行 owner
- `available_action_keys` 只是投影缓存，可随时重算，不是主真相

#### 说明

凡是“任务当前处于什么状态、谁在负责、下一步能做什么”，都应最终以 `TaskInstance` 为主。  
不要让 `TaskInstance` 与 `WorkflowInstance` 同时各自维护一套互相可能冲突的业务状态真相。

---

### 4. `WorkflowInstance`

代表某个模板 workflow 在某个 workspace 中实例化后的运行对象。

#### 最小字段建议

- `workflow_instance_id`
- `workflow_template_id`
- `workspace_id`
- `task_instance_id`
- `current_node_id`
- `current_status_projection`
- `current_owner_role_assignment_id_projection`
- `started_at`
- `updated_at`

#### Source of truth 规则

- `WorkflowInstance` 主要负责节点路径、画布位置、高亮链路、节点详情投影
- `current_status_projection` 与 `current_owner_role_assignment_id_projection` 是从 `TaskInstance` 和当前节点推导出来的投影
- 若 `WorkflowInstance` 投影与 `TaskInstance` 主状态冲突，以 `TaskInstance` 为准
- 不允许出现“Task 已经 Done，但 Workflow 还显示 In Progress 且被当成主真相”的双轨实现

---

### 5. `DecisionRecord`

代表一次决策门记录。

#### 最小字段建议

- `decision_record_id`
- `task_instance_id`
- `workflow_instance_id`
- `decision_topic`
- `decision_question`
- `options`
- `recommended_option`
- `risk_summary`
- `approver_role_key`
- `decision_result`
- `status`
- `created_by_role_assignment_id`
- `resolved_by_role_assignment_id`
- `created_at`
- `resolved_at`

#### 说明

用于支撑：

- `request_decision`
- `resume_after_decision`

#### 生命周期说明

- `request_decision` 创建 `DecisionRecord`，并让任务进入 `Waiting Decision`
- `approve/reject` 只负责把 `DecisionRecord.status` 置为 `approved/rejected`
- 任务**不会**仅因为 decision record 已被 approve/reject 就自动离开 `Waiting Decision`
- 必须再通过 `resume_after_decision` 才能让任务进入下一业务状态

---

### 6. `HandoffRecord`

代表一次角色交接记录。

#### 最小字段建议

- `handoff_record_id`
- `task_instance_id`
- `workflow_instance_id`
- `from_role_assignment_id`
- `to_role_assignment_id`
- `handoff_summary`
- `delivered_artifacts`
- `known_risks`
- `next_actions`
- `created_at`

#### 说明

用于支撑 `submit_handoff` 这类 transition。

---

### 7. `ReviewRecord`

代表一次审核 / 回归 / 技术复核记录。

#### 最小字段建议

- `review_record_id`
- `task_instance_id`
- `workflow_instance_id`
- `review_type`
- `reviewer_role_assignment_id`
- `review_status`
- `result`
- `checklist_summary`
- `issues_found`
- `next_action`
- `created_at`

#### 枚举建议

- `review_status`: `started`, `completed`
- `result`: `pending`, `passed`, `rejected`

#### 说明

用于支撑：

- `start_review`
- `reject_to_rework`
- `mark_ready_for_delivery` 的前置审核证明

---

### 8. `DeliverySummaryRecord`

代表进入 `Ready for Delivery` 前的交付摘要记录。

#### 最小字段建议

- `delivery_summary_record_id`
- `task_instance_id`
- `workflow_instance_id`
- `change_summary`
- `affected_scope`
- `validation_summary`
- `remaining_risks`
- `created_by_role_assignment_id`
- `created_at`

---

### 9. `ExecutionLog`

代表一次动作执行或关键状态变更的审计日志。

#### 最小字段建议

- `execution_log_id`
- `task_instance_id`
- `workflow_instance_id`
- `action_key`
- `actor_user_id`
- `actor_role_key`
- `previous_status`
- `current_status`
- `comment`
- `created_record_ids`
- `created_at`

---

## 三、视图投影对象

### 1. `CanvasNode`

用于前端画布展示的节点对象。

#### 最小字段建议

- `node_id`
- `node_type`
- `title`
- `subtitle`
- `status`
- `owner_label`
- `position`
- `related_entity_type`
- `related_entity_id`

---

### 2. `CanvasEdge`

用于前端画布展示的连线对象。

#### 最小字段建议

- `edge_id`
- `source_node_id`
- `target_node_id`
- `edge_type`
- `label`
- `status`

---

### 3. `NodeDetailViewModel`

用于右侧详情面板。

#### 最小字段建议

- `node_id`
- `title`
- `summary`
- `owner`
- `inputs`
- `outputs`
- `enabled_skills`
- `risks`
- `latest_logs`
- `latest_record_summaries`
- `available_actions`

---

### 4. `TaskActionProjection`

用于前端任务详情页或节点详情页展示当前可执行动作。

#### 最小字段建议

- `action_key`
- `label`
- `enabled`
- `disabled_reason`
- `requires_confirmation`
- `required_payload_type`

#### 说明

这是运行时投影，不是模板层对象。  
它由当前状态、guard、角色权限、前置记录共同计算得出。

---

## 四、对象关系

### 模板聚合关系

- 一个 `CompanyTemplate` 包含多个 `RoleTemplate`
- 一个 `CompanyTemplate` 包含多个 `WorkflowTemplate`
- 一个 `CompanyTemplate` 包含多个 `TaskTemplate`
- 一个 `RoleTemplate` 可默认挂多个 `SkillPackageTemplate`
- 一个 `WorkflowTemplate` 可声明多个默认角色与节点

### 实例化关系

- 一个 `CompanyTemplate` 可实例化为一个 `Workspace`
- 一个 `Workspace` 可包含多个 `RoleAssignment`
- 一个 `WorkflowTemplate` 可实例化为 `WorkflowInstance`
- 一个 `TaskTemplate` 可实例化为 `TaskInstance`
- 一个 `TaskInstance` 通常对应一个主 `WorkflowInstance`

### transition 记录关系

- 一个 `TaskInstance` 可产生多个 `DecisionRecord`
- 一个 `TaskInstance` 可产生多个 `HandoffRecord`
- 一个 `TaskInstance` 可产生多个 `ReviewRecord`
- 一个 `TaskInstance` 可产生多个 `DeliverySummaryRecord`
- 一个 `TaskInstance` 可产生多个 `ExecutionLog`
- 一个 `TaskInstance` 在进入 `Ready for Delivery` 前至少应有一个有效 `DeliverySummaryRecord`
- 一个 `TaskInstance` 在 `Waiting Decision` 阶段最多只允许有一个未解决的 active `DecisionRecord`

### 视图映射关系

- 一个 `WorkflowInstance` 可投影为一组 `CanvasNode` + `CanvasEdge`
- 每个 `CanvasNode` 关联一个真实领域对象
- `NodeDetailViewModel` 和 `TaskActionProjection` 由运行时对象拼装生成，不作为核心主存储对象

---

## 五、Phase 1 必须先落地的对象

为了让产品先跑起来，建议优先落地：

1. `CompanyTemplate`
2. `RoleTemplate`
3. `WorkflowTemplate`
4. `TaskTemplate`
5. `Workspace`
6. `RoleAssignment`
7. `TaskInstance`
8. `WorkflowInstance`
9. `DecisionRecord`
10. `HandoffRecord`
11. `ReviewRecord`
12. `DeliverySummaryRecord`
13. `ExecutionLog`
14. `CanvasNode`
15. `CanvasEdge`

以下可以第二阶段再增强：

- 更复杂的 `SkillPackageTemplate` 元数据
- 更细的 `ExecutionLog.payload`
- 跨 workflow 复合编排关系
- 更复杂的角色权限关系图

---

## 当前阶段统一结论

当前阶段最重要的不是继续加对象名词，  
而是把以下 3 个真相来源收死：

1. 任务业务状态真相：`TaskInstance`
2. transition 记录真相：`DecisionRecord` / `HandoffRecord` / `ReviewRecord` / `DeliverySummaryRecord`
3. 页面动作真相：运行时 `TaskActionProjection`

只要这 3 层清楚，Codex 才能少猜、少返工。
