---
name: video-factory-recover
description: Use when a Video Factory analysis, rough cut, review render, or final render was interrupted and the durable ledger must determine the only legal next action.
---

# Recover a Video Factory job

## When to use · 何时恢复

仅在分析、粗剪、同步审阅或终版任务被中断，且已有可读取的持久台账或证据回执时使用。没有台账
时先报告缺失项，不猜测已经完成的步骤。

## Workflow · 恢复流程

1. 执行 `bin/video-factory status <ledger.json>`，确认 state、revision、stage 和历史。
2. 执行 `bin/video-factory recover <ledger.json>`，只读取恢复前沿，不直接重新渲染。
3. 重新校验 capability、plan/edit hash、stage、round、素材哈希和已有分段回执。
4. 仅当 nextAction 为 `resume_pending` 且幂等键一致时，用同一计划和批准继续 run。
5. Failed 项必须修复原因并创建新 round；不得用原参数自动重试。

`Completed` 分段不会重做；回执缺失或哈希变化的分段不能冒充完成。旧粗剪、同步审阅版、终版和
EditDecision 均保留。若台账损坏，报告具体文件和最后可验证回执，不猜测状态、不删除工作目录。
恢复完成的候选只能进入 `ReviewReady`；只有明确的人工复核才能从 `ReviewReady` 进入
`Completed` 或 `ReworkReady`，模型评分不得推进这道状态门。

常见恢复：进程中断 → 继续 Pending；Chrome 缺失 → 保留普通粗剪并跳过增强审阅；素材变化 →
旧批准失效并创建新 round；确定性质量失败 → 修复计划后重新报价。

## Capability boundaries · 能力边界

能恢复同一幂等键下的 Pending 工作；需要原计划、批准、素材和回执仍有效；不重试 Failed、不覆盖
旧产物、不删除工作目录、不从损坏台账臆造状态。

## Validation and gotchas · 校验与陷阱

任何 Failed 优先返回 `new_round_required`，即使还有 Pending；只有 Running + Pending 才返回
`resume_pending`。恢复前重算素材和分段哈希，回执被篡改则不能复用。

参见本技能内的 [操作与恢复](references/operations.md) 和
[示例与 FAQ](references/examples-and-faq.md)。

<!-- QUALITY_BASELINE_V1 -->
## When to use（什么时候使用）

当用户需要 **从已记录状态恢复中断任务，避免重复提交或重复计费** 时加载本技能。先从请求中提取目标、输入、约束、交付格式和验收标准；描述摘要为：Use when a Video Factory analysis, rough cut, review render, or final render was interrupted and the durable ledger must determine the only legal next action.。

## Rules

- 先读后写：先确认当前状态与真实能力，再执行会改变外部状态的动作。
- 权限最小化：只使用完成当前步骤所需的文件、工具、账户与网络范围。
- 证据优先：运行结果、资源 ID、版本、哈希或测试输出缺失时，明确标记为 `NOT_VERIFIED`。
- 幂等优先：保留请求标识与阶段状态；结果不明确时先查询，不进行盲目重试。
- 隐私安全：日志、示例、回执和错误信息不得包含 token、cookie、密钥或个人敏感数据。

## Workflow

### Step 1：澄清意图

确认本技能是否匹配目标；若只是相邻需求，交给更精确的技能。
### Step 2：执行预检

核对任务标识、最后成功阶段、远端状态、预算和授权范围；任一关键条件未知时停止在只读阶段。
### Step 3：形成计划

列出将调用的工具、会改变的对象、成功标准以及失败后的安全退出方式。
### Step 4：执行动作

仅继续尚未完成且可证明安全的阶段；每个外部调用均保留可关联的状态或回执。
### Step 5：验证交付

返回复用结果、新执行步骤、未恢复项和下一人工决策点，并把事实、推断和未验证项分开陈述。

## Validation checklist

- [ ] 技能触发条件与用户意图一致，没有把相邻任务误路由到本技能。
- [ ] 输入、目标对象、版本和输出位置均已明确，且没有使用猜测值替代必填值。
- [ ] 所有写入、付费、发布或不可逆动作都在用户授权范围内。
- [ ] 结果已用独立检查验证；仅有“命令成功”或“文件存在”不算完整验收。
- [ ] 输出包含实际证据、失败/跳过项、剩余风险和可执行的下一步。

## Gotchas

1. **把计划当结果**：文档或提示词不等于真实执行；必须标明实际运行层级。
2. **错误重试**：超时或响应丢失可能已经产生远端状态，先查询再决定是否重试。
3. **隐式扩大范围**：批量、全量、发布、覆盖和付费不是普通读写的自然延伸。
4. **版本漂移**：引用外部资源时记录版本、tag 或提交；不要把可变分支当发布证据。
5. **证据过期**：缓存、旧截图和历史测试不能证明当前环境；在交付前刷新关键证据。

## 不适用与边界

没有幂等键、远端状态或用户授权时不重提任务；恢复不扩大原批准范围。 如果请求需要别的技能，不复制其正文；按技能名称进行交接，并保留当前任务上下文。

## Progressive disclosure

- 需要确定输入/输出、状态和授权点时，读取 `references/workflow-contract.md`。
- 需要交付前自检时，读取 `references/validation-checklist.md`。
- 遇到超时、部分成功或恢复场景时，读取 `references/error-recovery.md`。
- 首次运行、拒绝越权和失败恢复分别参考 `examples/happy-path.md`、`examples/boundary-refusal.md`、`examples/failure-recovery.md`。
