# product/role-skill-mapping.md

## 目标
把角色、技能包、workflow 节点三者的关系收成一张可落地的映射规则，确保：
- 技能系统不只停留在概念层
- 模板实例化后，角色默认拥有哪些能力是明确的
- 节点详情页知道该展示哪些技能
- Codex 在实现角色与技能页时有明确依据

---

## 核心原则
- 角色是职责容器
- 技能包是默认能力组合
- workflow 节点声明“当前阶段最相关的技能”
- 第一阶段优先做默认挂载和展示，不急着做复杂动态编排

---

## 第一阶段默认角色与技能包映射

### 1. PM Agent
#### 默认技能包
- `skill_pkg_pm_basic`
- `skill_pkg_requirement_clarification`
- `skill_pkg_acceptance_definition`
- `skill_pkg_task_breakdown`

#### 主要作用
- 澄清需求
- 补任务字段
- 定义验收标准
- 生成任务拆解

---

### 2. Tech Lead Agent
#### 默认技能包
- `skill_pkg_tech_design`
- `skill_pkg_risk_analysis`
- `skill_pkg_module_breakdown`
- `skill_pkg_dependency_check`

#### 主要作用
- 输出技术方案
- 识别风险
- 拆模块
- 识别依赖与边界

---

### 3. Dev Agent
#### 默认技能包
- `skill_pkg_implementation`
- `skill_pkg_self_test_summary`
- `skill_pkg_change_summary`

#### 主要作用
- 实现功能 / 修复问题
- 输出自测结果
- 输出变更说明

---

### 4. QA / Review Agent
#### 默认技能包
- `skill_pkg_review_check`
- `skill_pkg_regression_check`
- `skill_pkg_rework_feedback`

#### 主要作用
- 审核验收
- 回归检查
- 输出返工意见

---

### 5. Ops / Release Agent
#### 默认技能包
- `skill_pkg_delivery_summary`
- `skill_pkg_release_note`
- `skill_pkg_postmortem_init`

#### 主要作用
- 交付摘要
- 发布说明
- 复盘初始化

---

## workflow 节点与技能展示映射

### feature workflow
#### PM 节点
展示技能：
- requirement clarification
- acceptance definition
- task breakdown

#### Tech Lead 节点
展示技能：
- tech design
- risk analysis
- dependency check

#### Dev 节点
展示技能：
- implementation
- self test summary
- change summary

#### QA 节点
展示技能：
- review check
- regression check

#### Ops 节点
展示技能：
- delivery summary
- release note

---

### bugfix workflow
#### Tech Lead 节点
展示技能：
- root cause reasoning（可归入 tech design）
- risk analysis
- regression scope check

#### Dev 节点
展示技能：
- implementation
- self test summary
- fix explanation

#### QA 节点
展示技能：
- regression check
- rework feedback

---

### hotfix workflow
#### Tech Lead 节点
展示技能：
- hotfix strategy
- risk analysis
- rollback awareness

#### Dev 节点
展示技能：
- hotfix implementation
- quick validation summary

#### QA 节点
展示技能：
- critical path verification

#### Ops 节点
展示技能：
- release note
- incident summary
- postmortem init

---

## 技能包展示规则

### 在角色与技能页展示
- 技能包名称
- 一句话说明
- 是否默认启用
- 适用角色

### 在 workflow 节点详情页展示
- 当前节点相关技能
- 当前是否启用
- 简短输入 / 输出说明

### 不在第一阶段默认展示
- timeout
- retry policy
- fallback policy
- tool binding 细节
- prompt 片段

---

## 默认启用与可选启用

### 默认启用
- 每个角色的基础技能包
- 与主 workflow 强相关的技能包

### 可选启用
- 扩展技能包
- 某些只在特定场景使用的技能包

### 第一阶段建议
- 先实现“默认启用 + 可见”
- 再实现“可选开关”
- 最后再做更细粒度配置

---

## 模板实例化时的技能挂载规则
从 `CompanyTemplate` 实例化时：
1. 先创建 `RoleAssignment`
2. 再为每个角色绑定默认 `SkillPackage`
3. 节点详情页按 workflow 类型筛出当前相关技能
4. 用户可后续在角色与技能页做启停微调

---

## Codex 实施建议
如果 Codex 按本文档实现，建议顺序：
1. 先把默认角色 -> 默认技能包映射写死到配置或种子数据
2. 再做角色与技能页展示
3. 再做节点详情里的技能展示
4. 最后补技能启用 / 禁用接口

---

## 一句话总结
role-skill-mapping 文档的作用，是把“角色有能力”这件事从抽象概念，落成：
**这个角色默认带哪些技能，这个节点该展示哪些技能。**
