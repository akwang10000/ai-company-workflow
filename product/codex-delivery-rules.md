# product/codex-delivery-rules.md

## 目标

定义 Codex / 自动化编码代理在当前阶段提交实现时的最小交付规则，确保每轮交付都与当前 Phase 1 主入口和进度真相一致。

---

## 默认实施入口（必须与项目其它入口一致）

每次进入实现前，默认按以下顺序读取：

1. `tasks/IMPLEMENTATION-PROGRESS.md`
2. `AGENTS.md`
3. `product/domain-model.md`
4. `product/task-status-guards.md`
5. `product/task-transition-api-and-actions.md`
6. `product/api-contracts.md`
7. `product/implementation-phases.md`
8. `governance/task-schema.md`
9. `governance/decision-gates.md`
10. 对应 `workflows/playbooks/*.md`
11. 对应 `roles/playbooks/*.md`
12. `governance/ready-for-delivery-checklist.md`

### 明确删除的旧入口口径

以下文档不再作为当前默认主入口：

- `module-breakdown.md`
- `implementation-roadmap.md`
- `canvas-ui-spec.md`
- `screens-and-flows.md`

若它们存在，也只能作为补充参考，不能盖过当前 product 主真相。

---

## 每轮交付必须包含的内容

### 0. Progress file update

必须同步更新：

- `tasks/IMPLEMENTATION-PROGRESS.md`

至少写清：

- 当前 phase / subphase
- 最近完成项
- 当前正在做什么
- 当前阻塞 / 未决问题
- 下一步最该做什么

### 1. Changed files

列出本轮改动文件。

### 2. What changed

说明：

- 做了什么
- 为什么这么做
- 当前属于哪个 phase

### 3. Validation

说明：

- 本轮做了哪些验证
- 哪些地方还没验证
- 结果如何

### 4. Current status

说明：

- 当前做到哪
- 哪条主链路已通
- 哪条还没通

### 5. Risks / open questions

说明：

- 当前剩余风险
- 是否存在需要进入 `Waiting Decision` 的问题

---

## 交付边界要求

- 不得在一轮实现里同时重开主模型、主状态机、主 API 三大口径
- 若发现主真相文档存在歧义，应先修正文档，再实现
- 若发现实现范围过大，优先缩小当前交付面，不要扩总纲

---

## 当前阶段统一结论

当前阶段的 Codex 交付规则只有一个目标：

> 让每轮实现都严格挂在同一套主入口与进度真相上，而不是再长出第二套施工顺序。
