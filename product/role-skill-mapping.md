# product/role-skill-mapping.md

## 目标
把“角色是职责容器、技能是能力模块、workflow 是编排骨架”这套原则，收成第一阶段可直接落地的映射表，确保：
- 模板默认能力包清楚
- 角色与技能挂载关系清楚
- workflow 节点和技能展示不靠猜
- Codex 与前后端实现时有明确依据

---

## 核心原则
- 角色负责职责，技能负责能力
- 默认先挂基础技能包，不要求新用户从零配
- 第一阶段先做“默认可用”，不做复杂动态编排
- 技能细节放详情层，不把主画布炸掉

---

## 第一阶段角色 -> 默认技能包映射

### 1. PM Agent
#### 默认技能包
- `skill_pkg_pm_requirement_clarify`
- `skill_pkg_pm_task_breakdown`
- `skill_pkg_pm_acceptance_criteria`

#### 主要用途
- 澄清需求
- 定义范围 / 非范围
- 写验收标准
- 给 Tech Lead 做 handoff

---

### 2. Tech Lead Agent
#### 默认技能包
- `skill_pkg_tl_solution_design`
- `skill_pkg_tl_risk_assessment`
- `skill_pkg_tl_module_split`

#### 主要用途
- 输出技术方案
- 识别风险
- 拆模块
- 决定修复路径 / 止血路径

---

### 3. Dev Agent
#### 默认技能包
- `skill_pkg_dev_implementation`
- `skill_pkg_dev_self_validation`
- `skill_pkg_dev_change_summary`

#### 主要用途
- 完成功能实现 / 缺陷修复
- 留下自测说明
- 留下变更摘要

---

### 4. QA / Review Agent
#### 默认技能包
- `skill_pkg_qa_acceptance_review`
- `skill_pkg_qa_regression_check`
- `skill_pkg_qa_rework_feedback`

#### 主要用途
- 对照验收标准审核
- 做最小回归检查
- 给出返工意见或通过说明

---

### 5. Ops / Release Agent
#### 默认技能包
- `skill_pkg_ops_delivery_summary`
- `skill_pkg_ops_archive`
- `skill_pkg_ops_postmortem_init`

#### 主要用途
- 汇总交付说明
- 归档产物
- 初始化 postmortem / follow-up

---

## workflow 节点 -> 技能展示映射

### feature workflow
- PM 节点：需求澄清 / 任务拆解 / 验收标准生成
- Tech Lead 节点：技术方案 / 风险评估 / 模块拆分
- Dev 节点：实现修改 / 自测摘要 / 变更说明
- QA 节点：验收检查 / 回归检查 / 返工建议
- Ops 节点：交付摘要 / 归档整理

---

### bugfix workflow
- PM 或 Tech Lead 节点：复现补全 / 影响范围判断
- Tech Lead 节点：修复路径 / 风险识别
- Dev 节点：修复实现 / 复现验证 / 变更摘要
- QA 节点：回归检查 / 返工建议
- Ops 节点：修复结果摘要 / follow-up 记录

---

### hotfix workflow
- Tech Lead 节点：事故判断 / 最小止血方案
- Dev 节点：hotfix 实施 / 临时修补执行
- QA 节点：最小关键验证
- Ops 节点：hotfix 收口 / follow-up 初始化

---

## 角色详情页建议展示内容
在角色详情页，默认展示：
- 角色名
- 职责摘要
- 默认技能包列表
- 当前启用技能包
- 适用 workflow

第一阶段不强求默认展示：
- schema 细节
- tool binding
- retry / timeout / fallback

---

## 画布节点详情建议展示内容
在 workflow 节点详情中，展示：
- 当前角色
- 当前节点启用技能
- 输入
- 输出
- 风险
- 最近执行记录

不要在主画布节点卡片上直接铺所有技能名。

---

## 默认启用策略
### 第一阶段建议
- 模板实例化后，角色基础技能包默认启用
- 用户可在角色与技能页手动关闭少数技能包
- 不要求第一阶段支持复杂条件触发启用

---

## 第一阶段暂不强求
- 技能运行时自动切换策略
- 节点内多技能编排 DSL
- 复杂技能版本兼容矩阵
- 普通用户可视化编辑 skill internals

---

## 与模板实例化的关系
模板实例化后：
- `RoleAssignment` 自动绑定默认技能包
- 画布节点详情可根据角色与 workflow 映射显示技能
- 新用户无需手工给每个角色装配能力

---

## Codex 实施建议
如果 Codex 依据本文档实现，建议顺序：
1. 先把默认技能包数据结构建好
2. 再把 RoleTemplate -> SkillPackage 的绑定做出来
3. 再把节点详情里的技能展示接上
4. 最后再补启用 / 禁用接口

---

## 一句话总结
角色-技能映射文档的目标，是把“技能系统”从概念层拉到产品层：
**让每个角色默认就有一组能工作的能力包，而不是让用户从零装配。**
