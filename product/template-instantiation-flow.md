# product/template-instantiation-flow.md

## 目标
定义“第一个软件公司模板”如何被实例化成一个可运行 workspace，让产品、后端、前端、Codex 都清楚：
- 用户点下“从模板启动”后发生什么
- 系统会生成哪些对象
- 用户如何进入第一条任务
- 哪些步骤自动完成，哪些步骤留给用户确认

---

## 核心原则
- 模板启动必须简单，不要求用户先手搓所有角色和 workflow
- 先生成一套可运行骨架，再允许微调
- 用户第一步看到的应是“能工作的系统”，不是一堆空白配置
- 模板实例化结果必须可回查、可继续配置、可继续推进

---

## 实例化前提
用户至少需要：
- 选中一个 `CompanyTemplate`
- 提供 workspace 名称
- 确认拥有者 / 创建者身份

第一阶段不强求：
- 起步时就逐项细调角色和技能
- 起步时就逐项编辑画布节点

---

## 实例化流程（主路径）

### Step 1：用户选择模板
#### 页面
- 模板首页
- 模板详情页

#### 用户动作
- 选择“第一个软件公司模板”
- 点击“从模板启动”

#### 系统动作
- 校验模板状态可用
- 拉取模板详情

---

### Step 2：填写最小启动信息
#### 用户输入
- workspaceName
- ownerUserId（或当前登录用户默认带入）

#### 第一阶段不强求
- 起步时逐个改角色名
- 起步时逐个选技能包

---

### Step 3：创建 Workspace
#### 系统生成
- `Workspace`
- 基础元数据
- 初始 dashboard 数据结构

#### 产出
- 一个可进入的工作空间

---

### Step 4：实例化角色模板
#### 系统生成
基于 `CompanyTemplate.included_role_template_ids`，生成：
- PM Agent
- Tech Lead Agent
- Dev Agent
- QA / Review Agent
- Ops / Release Agent

#### 生成对象
- `RoleAssignment`

#### 默认行为
- 角色默认启用基础技能包
- 角色初始状态为 enabled

---

### Step 5：实例化 workflow 模板
#### 系统生成
基于 `CompanyTemplate.included_workflow_template_ids`，生成可选 workflow 骨架：
- feature workflow
- bugfix workflow
- hotfix workflow

#### 说明
第一阶段这里不一定立刻生成真实运行中的 workflow instance，
也可以先生成“可用于任务启动的 workflow 配置骨架”。

---

### Step 6：注入任务样例与起步引导
#### 系统生成或展示
- feature task example
- bugfix task example
- hotfix task example
- 新手引导卡片

#### 目的
让用户知道：
- 接下来能创建什么任务
- 不需要从零理解 schema

---

### Step 7：进入工作空间首页
#### 页面重点
- 当前任务数
- 当前待拍板数
- 当前推荐第一步
- 快捷入口：新建任务 / 看画布 / 看模板内容

---

## 实例化后默认结果
执行一次“从模板启动”后，系统应默认具备：
- 1 个 `Workspace`
- 5 个 `RoleAssignment`
- 3 个 workflow 骨架
- 3 类任务样例可参考
- 1 组基础 dashboard 卡片
- 默认技能包已挂载到角色

---

## 推荐的首次启动路径
实例化完成后，优先引导用户：
1. 查看工作空间首页
2. 查看模板生成了哪些角色
3. 新建第一条 feature / bugfix / hotfix 任务
4. 进入任务详情与 workflow 画布

不要默认把用户直接丢进复杂模板配置页。

---

## 自动完成 vs 用户确认

### 自动完成
- 创建 workspace
- 生成默认角色实例
- 生成 workflow 骨架
- 挂载默认技能包
- 准备任务样例入口

### 用户后续可确认 / 微调
- 改 workspace 名称
- 改角色显示名
- 启用 / 禁用部分技能包
- 调整模板说明
- 创建第一条真实任务

---

## 与页面流转的关系
- 模板首页 -> 模板详情页 -> 启动弹窗 -> 工作空间首页
- 工作空间首页 -> 任务列表页 / 画布页 / 角色与技能页
- 第一次任务创建后 -> 任务详情页 -> workflow 画布页

---

## Codex 实施建议
如果 Codex 要按本文档实现模板启动流程，建议顺序：
1. 先实现 `POST /templates/{id}/instantiate`
2. 再实现 workspace 首页
3. 再实现角色实例与默认技能展示
4. 最后接任务样例与新手引导

---

## 一句话总结
模板实例化流程的目标不是“复制一堆配置”，而是：
**让用户点一下，就拥有一个能立刻开始跑第一条任务的软件公司骨架。**
