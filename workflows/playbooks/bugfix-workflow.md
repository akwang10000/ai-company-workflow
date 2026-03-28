# workflows/playbooks/bugfix-workflow.md

## 目标
定义常规 bug 修复任务从发现、定位、修复到验证、收口的完整 SOP。

## 适用范围
适用于：
- 已知问题修复
- 可复现的功能异常
- 明确范围的回归问题

不直接覆盖：
- 正在扩散的生产事故
- 需要分钟级响应的线上紧急问题

## 参与角色
- CEO / 用户
- PM Agent（按需）
- Tech Lead Agent
- Dev Agent
- QA / Review Agent
- Ops / Release Agent

## 主流程
1. 登记问题与影响范围
2. 补齐复现条件和验收标准
3. Tech Lead 定位并给出修复路径
4. Dev 实施修复并验证
5. QA / Review 做回归检查
6. CEO / 用户确认是否接受当前修复范围
7. Ops / Release 收口归档

## 关键停机点
- 根因不明但影响面过大
- 修复方案可能引入高风险兼容性问题
- 需要直接改生产数据
