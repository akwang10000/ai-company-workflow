# product/domain-model.md

## 目标
定义 AI Company Workflow 第一阶段（研发团队版）的核心领域模型，确保后端建模、前端状态结构、模板实例化和 Codex 实施都围绕同一套对象体系推进。

这份文档解决的问题是：
- 系统里到底有哪些核心对象
- 这些对象之间是什么关系
- 模板与实例怎么区分
- 哪些对象是第一阶段必须落地的

---

## 核心原则
- 先把核心对象收死，再谈前后端细节
- 模板对象与运行时实例对象必须分开
- workflow、task、角色、技能不要混成一个大对象
- 第一阶段先满足研发团队版，不急着一次抽象全行业终态

---

## 第一阶段核心对象总览
建议第一阶段至少包含以下对象：

### 模板层对象
- `RoleTemplate`
- `WorkflowTemplate`
- `TaskTemplate`
- `SkillPackageTemplate`
- `CompanyTemplate`

### 运行时对象
- `Workspace`
- `WorkflowInstance`
- `TaskInstance`
- `RoleAssignment`
- `DecisionRecord`
- `HandoffRecord`
- `ExecutionLog`

### 视图层对象
- `CanvasNode`
- `CanvasEdge`
- `NodeDetailViewModel`

---

## 模板层对象

### 1. `RoleTemplate`
代表一个可复用角色模板。

#### 最小字段建议
- `role_template_id`
- `name`
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
它描述的是“这个角色应该是什么样”，不是某一次任务里谁正在执行。

---

### 2. `WorkflowTemplate`
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
它描述的是 feature / bugfix / hotfix 这种流转骨架，不是具体某个任务实例本身。

---

### 3. `TaskTemplate`
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

### 4. `SkillPackageTemplate`
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

### 5. `CompanyTemplate`
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
它是第一阶段最关键的聚合模板对象。

---

## 运行时对象

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

### 2. `WorkflowInstance`
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
- `current_owner`
- `next_owner`
- `acceptance_criteria`
- `definition_of_done`
- `known_risks`
- `created_at`
- `updated_at`

---

### 4. `RoleAssignment`
代表某个角色模板在某个 workspace / workflow 中被分配后的实际角色实例。

#### 最小字段建议
- `role_assignment_id`
- `workspace_id`
- `role_template_id`
- `display_name`
- `bound_user_id`
- `bound_agent_id`
- `enabled_skill_package_ids`
- `status`

---

### 5. `DecisionRecord`
代表一次决策门记录。

#### 最小字段建议
- `decision_record_id`
- `task_instance_id`
- `workflow_instance_id`
- `decision_type`
- `reason`
- `options`
- `recommended_option`
- `approver`
- `decision_result`
- `created_at`

---

### 6. `HandoffRecord`
代表一次角色交接记录。

#### 最小字段建议
- `handoff_record_id`
- `task_instance_id`
- `from_role_assignment_id`
- `to_role_assignment_id`
- `completed_items`
- `unfinished_items`
- `known_risks`
- `next_actions`
- `created_at`

---

### 7. `ExecutionLog`
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

---

## 视图层对象

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
- `available_actions`

---

## 对象关系（第一阶段）

### 模板聚合关系
- 一个 `CompanyTemplate` 包含多个 `RoleTemplate`
- 一个 `CompanyTemplate` 包含多个 `WorkflowTemplate`
- 一个 `CompanyTemplate` 包含多个 `TaskTemplate`
- 一个 `RoleTemplate` 可以默认挂多个 `SkillPackageTemplate`
- 一个 `WorkflowTemplate` 可以声明多个默认角色与节点

### 实例化关系
- 一个 `CompanyTemplate` 可实例化为一个 `Workspace`
- 一个 `Workspace` 可包含多个 `RoleAssignment`
- 一个 `TaskTemplate` 可实例化为 `TaskInstance`
- 一个 `WorkflowTemplate` 可实例化为 `WorkflowInstance`
- 一个 `TaskInstance` 通常对应一个主 `WorkflowInstance`
- 一个 `TaskInstance` 可产生多个 `DecisionRecord`、`HandoffRecord`、`ExecutionLog`

### 视图映射关系
- 一个 `WorkflowInstance` 可投影为一组 `CanvasNode` + `CanvasEdge`
- 每个 `CanvasNode` 关联一个真实领域对象
- `NodeDetailViewModel` 由运行时对象拼装生成，不作为核心主存储对象

---

## 第一阶段必须先落地的对象
为了让产品先跑起来，建议优先落地：
1. `CompanyTemplate`
2. `RoleTemplate`
3. `WorkflowTemplate`
4. `TaskInstance`
5. `WorkflowInstance`
6. `RoleAssignment`
7. `DecisionRecord`
8. `HandoffRecord`
9. `CanvasNode`
10. `CanvasEdge`

以下可以第二阶段再增强：
- 更复杂的 `SkillPackageTemplate` 元数据
- 更细的 `ExecutionLog.payload`
- 跨 workflow 复合编排关系

---

## Codex 实施提示
如果 Codex 依据本文档开始实现，建议顺序是：
1. 先建模板层对象
2. 再建运行时对象
3. 再建画布投影对象
4. 最后补对象间转换与实例化逻辑

不要一上来就从画布 UI 反推领域模型。

---

## 一句话总结
这套系统的核心不是“一个大而全的任务对象”，而是：
**公司模板聚合角色、workflow、任务、技能；运行时再实例化成 workspace、task、workflow 和画布视图。**
