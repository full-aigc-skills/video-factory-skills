---
name: video-factory-plan
description: Use when local images, clips, audio, or ReelBench shot evidence must be converted into a validated rough-cut or final-cut EditDecision before rendering.
---

# Plan a Video Factory edit

## When to use · 何时使用

当用户已有创作目标与本地素材，需要把自然语言剪辑要求变成可执行、可批准、可版本化的
`EditDecision` 和 `VideoPlan` 时使用。单纯拉片用 `video-shots`，已有计划的渲染用 run Skill。

## Workflow · 计划流程

1. 为每项素材登记稳定 ID、本地相对路径、SHA-256、类型、来源与授权信息。
2. 使用有理 timebase 与整数 ticks，创建稳定 `C01...` clip ID。
3. 明确源入点/出点、时间线位置、轨道、转场、运动和增益；禁止悬空素材和重叠。
4. 输出画幅仅选 16:9、9:16 或 1:1；终版音频/字幕用对应 Asset ID 引用。
5. 执行 `bin/video-factory validate-plan video-plan.json`。
6. 执行 `bin/video-factory quote video-plan.json --stage rough|final`，等待当前阶段批准。

信息不足时先给“假设版计划”，明确假设的时长、画幅和节奏，再列出缺少的素材或选择；不要只说
“请提供更多信息”。缺素材时输出 `asset-requirements.json`，不得虚构文件或哈希。

## Capability boundaries · 能力边界

能做：从授权素材和 ReelBench 证据形成可版本化时间线。需要条件：素材路径、哈希、时长和输出
规格必须可验证。不做：不生成缺失媒体、不猜测哈希、不启动渲染、不调用其他插件私有模块。

## Validation · 输出与门禁

输出必须是闭合 Schema，不含未知字段、URL、凭据或任意滤镜。素材、顺序、规格或 revision 改变后，
旧批准失效。自然语言反馈生成新 revision 和结构化 diff，不覆盖旧版本。

## Gotchas · 常见陷阱

禁止浮点时间线、重复 clip ID、越界裁切、悬空素材、同轨重叠、未知字段和旧批准复用。遇到信息
不足时先给带明确假设的草案，并逐项列出用户需要补充的素材或选择。

参见本技能内的 [契约与安全](references/contracts-and-safety.md) 和
[端到端示例](references/examples-and-faq.md)。

<!-- QUALITY_BASELINE_V1 -->
## When to use（什么时候使用）

当用户需要 **在明确输入、预算和交付约束后执行生成或写入操作** 时加载本技能。先从请求中提取目标、输入、约束、交付格式和验收标准；描述摘要为：Use when local images, clips, audio, or ReelBench shot evidence must be converted into a validated rough-cut or final-cut EditDecision before rendering.。

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

校验输入、模型/工具能力、输出路径、预算上限和审批状态；任一关键条件未知时停止在只读阶段。
### Step 3：形成计划

列出将调用的工具、会改变的对象、成功标准以及失败后的安全退出方式。
### Step 4：执行动作

按一次批准执行并记录请求标识；模糊结果先查询而不是重提；每个外部调用均保留可关联的状态或回执。
### Step 5：验证交付

验证产物存在性、格式、哈希/标识、成本状态和质量门禁，并把事实、推断和未验证项分开陈述。

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

付费、发布、覆盖、上传或外部写入必须使用当前任务的显式授权；不自动扩大次数和预算。 如果请求需要别的技能，不复制其正文；按技能名称进行交接，并保留当前任务上下文。

## Progressive disclosure

- 需要确定输入/输出、状态和授权点时，读取 `references/workflow-contract.md`。
- 需要交付前自检时，读取 `references/validation-checklist.md`。
- 遇到超时、部分成功或恢复场景时，读取 `references/error-recovery.md`。
- 首次运行、拒绝越权和失败恢复分别参考 `examples/happy-path.md`、`examples/boundary-refusal.md`、`examples/failure-recovery.md`。
