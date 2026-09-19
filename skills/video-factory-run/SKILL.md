---
name: video-factory-run
description: Use when a validated VideoPlan has an explicit matching rough-cut or final-cut approval and should be rendered, collected, and recorded without automatic retry.
---

# Run an approved edit

## When to use · 前置条件

只在 `VideoPlan` 已校验、当前阶段已有匹配批准时执行。粗剪批准不能用于终版，旧 revision 的批准
不能用于新计划。先运行 `probe`；FFmpeg/ffprobe 缺失时指出具体缺项和安装责任，不要自行安装。

## Workflow · 执行

1. 重算 plan/edit hash 并核对 stage、round、quote revision。
2. 在授权根内重新解析普通文件并核对每个 SHA-256。
3. 只渲染台账中 Pending 的内容寻址分段，不重做回执仍有效的分段。
4. 以封闭枚举编译 argv，`shell:false`；不接受用户自定义 filter graph 或网络协议。
5. 粗剪和终版渲染完成后都进入 `ReviewReady`；显式人工 `accept` 后才进入 `Completed`。
6. 对媒体执行 ffprobe、完整解码、二次哈希和策略门，原子发布成片并写入台账。

失败只记录一次，不自动重试。超时、缺能力、哈希变化、批准不匹配或质量硬门失败时，返回“缺少/失败
的具体项 + 如何补齐”，保留分段供恢复。不得覆盖已采用产物。

## Capability boundaries · 能力边界

能做本地 FFmpeg 粗剪与终版；需要有效计划、对应批准和本机 FFmpeg/ffprobe；不调用外部生成服务、
不安装软件、不接受 URL 或任意命令。终版渲染后保持 `ReviewReady`，用户显式 accept 才 Completed。

## Validation and gotchas · 校验与陷阱

每一步重算哈希并验证回执。Chrome 缺失只影响同步审阅；素材缺失时生成 requirements；Failed 需要
新 round；进程中断只恢复 Pending；字幕默认内嵌轨，不谎称已经烧录或完成安全区视觉检查。

参见本技能内的 [操作与恢复](references/operations.md) 和
[契约与安全](references/contracts-and-safety.md)。

<!-- QUALITY_BASELINE_V1 -->
## When to use（什么时候使用）

当用户需要 **在明确输入、预算和交付约束后执行生成或写入操作** 时加载本技能。先从请求中提取目标、输入、约束、交付格式和验收标准；描述摘要为：Use when a validated VideoPlan has an explicit matching rough-cut or final-cut approval and should be rendered, collected, and recorded without automatic retry.。

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
