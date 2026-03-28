# workflows/playbooks/hotfix-workflow.md

## 目标
定义紧急 hotfix 任务的处理 SOP，在高时效压力下仍保留最基本的责任边界、风险控制和记录。

## 适用范围
适用于：
- 正在影响线上用户的严重问题
- 收入、登录、支付、权限等关键路径异常
- 需要尽快止血的生产问题

## 参与角色
- CEO / 用户
- Tech Lead Agent
- Dev Agent
- QA / Review Agent
- Ops / Release Agent

## 核心原则
- 先止血，再补完备
- 可以压缩流程，不能取消记录
- 高风险生产动作前必须人工拍板
- 必须明确：当前是临时止血还是根因修复

## 主流程
1. 确认事故级别与影响范围
2. Tech Lead 给出最小止血方案
3. CEO / 用户拍板
4. Dev 实施 hotfix
5. QA / Review 做最小关键验证
6. CEO / 用户确认当前结果是否接受
7. Ops / Release 收口并创建 follow-up

## 必须停机点
- 生产数据修复
- 回滚或大范围降级
- 关闭核心功能
- 对外说明或承诺恢复时间
