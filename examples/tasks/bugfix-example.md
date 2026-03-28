# examples/tasks/bugfix-example.md

## 目标
提供一条可直接复用的 **bugfix 任务单样例**。

## 示例任务
- task_id: `BUG-001`
- title: `修复后台用户列表页切换分页后筛选条件丢失`
- task_type: `bugfix`
- scenario: `engineering`
- goal: `修复后台用户列表页在切换分页后丢失筛选条件的问题`
- current_owner: `PM Agent`
- next_owner: `Tech Lead Agent`

## 验收标准示例
- 选择筛选条件后切换分页，条件不会丢失
- 切换每页条数后筛选条件仍保持
- 返回列表页时筛选条件按设计保留
- 不影响现有排序和分页能力

## 推荐流转
PM / Tech Lead -> Dev -> QA / Review -> CEO / 用户 -> Ops / Release
