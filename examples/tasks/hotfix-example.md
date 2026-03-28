# examples/tasks/hotfix-example.md

## 目标
提供一条可直接复用的 **hotfix 任务单样例**。

## 示例任务
- task_id: `HOTFIX-001`
- title: `紧急修复支付回调重复入账导致订单状态异常`
- task_type: `bugfix`
- scenario: `engineering`
- goal: `先止血支付回调重复入账问题，阻止新订单继续异常`
- current_owner: `Tech Lead Agent`
- next_owner: `CEO`

## 验收标准示例
- 同一支付单号重复回调时不会重复入账
- 新订单支付主路径可继续完成
- 当前 hotfix 是止血还是根因修复已明确说明
- follow-up 治理任务已提出

## 推荐流转
Tech Lead -> CEO / 用户拍板 -> Dev -> QA / Review -> CEO / 用户 -> Ops / Release
