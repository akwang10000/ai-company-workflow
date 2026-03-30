# product/domain-model.md

## 目标
定义 AI Company Workflow 在 **Phase 1 MVP（软件公司模板 / 研发团队版）** 的核心领域模型，确保后端建模、前端核心状态结构、模板实例化关系和 Codex 施工都围绕同一套对象体系推进。

这份文档主要回答：
- 系统里到底有哪些核心对象
- 模板对象与运行时对象如何区分
- task transition 相关记录应该如何建模
- 哪些对象是 Phase 1 必须先落地的

---

## 核心原则
- 先把核心对象收死，再谈实现细节
- 模板对象与运行时实例对象必须分开
- workflow、task、角色、技能不要混成一个大对象
- task transition 相关记录必须显式建模，不要全部塞进日志
- Phase 1 先满足软件公司模板，不急着一次抽象全行业终态

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
它描述的是“这个角色应该是什么样”，不是某次任务里谁正在执行。

---

### 3. `WorkflowTemplate`
代表一个可复用工作流模板。

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
它描述的是 feature / bugfix / hotfix 的流转骨架，不是具体任务实例本身。

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
- `current_owner_role_assignment_id`
- `next_owner_role_assignment_id`
- `acceptance_criteria`
- `definition_of_done`
- `known_risks`
- `created_at`
- `updated_at`

#### 补充建议字段
- `current_workflow_instance_id`
- `available_action_keys`（可选缓存 / 投影字段）
- `latest_review_result`
- `latest_delivery_readiness`

---

### 4. `WorkflowInstance`
代表某个模板 workflow 在某个 workspace 中实例化后的运行对象。

#### 最小字段建议
- `workflow_instance_id`
- `workflow_template_id`
- `workspace_id`
- `task_instance_id`
- `current_status`
- `current_node_id`
- `current_owner_role_assignment_id`
- `started_at`
- `updated_at`

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
- `created_at`
- `resolved_at`

#### 说明
用于支撑 `request_decision` / `resume_after_decision` 这类 transition。

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
- `result`
- `checklist_summary`
- `issues_found`
- `next_action`
- `created_at`

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

#### 说明
用于支撑 `mark_ready_for_delivery`。

---

### 9. `ExecutionLog`
代表任务和 workflow 的执行日志。

#### 最小字段建议
- `execution_log_id`
- `task_instance_id`
- `workflow_instance_id`
- `actor_type`
- `actor_id`
- `event_type`
- `summary`
- `payload`
- `created_at`

#### 说明
日志不是 review / handoff / decision / delivery summary 的替代品。
它用于审计追踪和 timeline 展示，不应承担全部业务语义。

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
- 一个 `TaskInstance` 可产生多个 `ExecutionLog`
- 一个 `TaskInstance` 在进入 `Ready for Delivery` 前至少应有一个有效 `DeliverySummaryRecord`

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

## 六、与其他文档的关系
- `product/domain-model.md`：定义对象与对象关系
- `product/api-contracts.md`：定义这些对象如何通过 API 暴露
- `product/task-status-guards.md`：定义哪些状态转换合法
- `product/task-transition-api-and-actions.md`：定义 action 如何触发记录创建与状态推进
- `product/template-instantiation-flow.md`：定义模板对象如何实例化为运行时对象

---

## 七、当前阶段统一结论
Phase 1 里最容易被做虚的一块，就是把 decision / handoff / review / delivery summary 都混成“日志”。

当前统一口径是：
- 这些记录都应是显式业务对象
- 日志只做审计与事件流，不替代一等业务记录

---

## 一句话总结
这份文档的作用，是把 Phase 1 的对象体系收成一套：

**模板、运行时实例、transition 记录、视图投影都能对齐，并足够支撑 Codex 继续施工的领域模型。**
