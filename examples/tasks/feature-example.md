# examples/tasks/feature-example.md

## 目标
提供一条可直接复用的 **feature 任务单样例**。

## 示例任务
- task_id: `FEAT-001`
- title: `新增后台用户封禁记录查询页`
- task_type: `feature`
- scenario: `engineering`
- goal: `新增后台用户封禁记录查询页，降低运营人工查库支持成本`
- current_owner: `PM Agent`
- next_owner: `Tech Lead Agent`

## 验收标准示例
- 支持按用户 ID 查询封禁记录
- 支持按时间范围筛选
- 展示封禁原因、操作人、操作时间
- 无权限用户不可访问
- 页面结果展示正常

## 推荐流转
PM -> Tech Lead -> Dev -> QA / Review -> CEO / 用户 -> Ops / Release
