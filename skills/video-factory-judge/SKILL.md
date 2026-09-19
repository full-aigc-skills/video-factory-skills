---
name: video-factory-judge
description: Use when a rough or final video needs deterministic media gates, advisory semantic review, and explicit human acceptance or rework labels.
---

# Judge a Video Factory artifact

## When to use · 使用方法

对粗剪、同步审阅版或终版执行 `bin/video-factory evaluate <artifact> <plan.json>`。先读取回执和
确定性证据，再做语义评价，最后记录人工标签；三者不得混成一个模型分数。

## Workflow · 判定顺序

1. 硬门：文件、哈希、完整解码、视频流、时长、尺寸、帧率、必需音轨、时间线和来源。
2. 策略门：黑帧、冻结、静音、字幕越界；无证据写 `SKIPPED` 或 `NOT_RUN`。
3. 语义建议：叙事、节奏、镜头职责、声画同步、重复镜头和视觉一致性。
4. 人工决定：approved、rejected 或 rework；人工 rejected 永远不能被模型覆盖。

每个失败项必须说明证据、影响和修复方式。确定性失败直接 FAIL；艺术性策略提示默认进入 review，
由用户决定是否接受。不得凭肉眼声称哈希、编解码或精确时长已经通过。

## Capability boundaries · 能力边界

能做技术门与结构化审阅；需要成片、计划和回执；不替代版权、品牌、无障碍或人工播放判断，也不
修改成片。缺少语义或视觉证据时保持 `NOT_RUN`，不能用模型猜测补齐。

## Validation and gotchas · 校验与陷阱

硬门失败不能人工放行；策略门误报可由人工接受但必须保留警告；人工 rejected 永远优先。不要把
同步审阅版当终版，不要把 `SKIPPED` 当通过，不要用 HTTP 200、文件存在或模型高分替代完整解码。

参见本技能内的 [操作与质量门](references/operations.md) 和
[示例与 FAQ](references/examples-and-faq.md)。

<!-- QUALITY_BASELINE_V1 -->
## When to use（什么时候使用）

当用户需要 **基于可验证证据进行质量、安全或交付审查** 时加载本技能。先从请求中提取目标、输入、约束、交付格式和验收标准；描述摘要为：Use when a rough or final video needs deterministic media gates, advisory semantic review, and explicit human acceptance or rework labels.。

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

冻结审查对象、验收标准、证据时间和版本标识；任一关键条件未知时停止在只读阶段。
### Step 3：形成计划

列出将调用的工具、会改变的对象、成功标准以及失败后的安全退出方式。
### Step 4：执行动作

逐项判定 PASS、FAIL、SKIPPED 或 BLOCKED，并记录依据；每个外部调用均保留可关联的状态或回执。
### Step 5：验证交付

输出发现、严重度、证据位置、修复建议和剩余风险，并把事实、推断和未验证项分开陈述。

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

审查结果不是修改授权；不伪造运行证据，也不把缺失证据标为通过。 如果请求需要别的技能，不复制其正文；按技能名称进行交接，并保留当前任务上下文。

## Progressive disclosure

- 需要确定输入/输出、状态和授权点时，读取 `references/workflow-contract.md`。
- 需要交付前自检时，读取 `references/validation-checklist.md`。
- 遇到超时、部分成功或恢复场景时，读取 `references/error-recovery.md`。
- 首次运行、拒绝越权和失败恢复分别参考 `examples/happy-path.md`、`examples/boundary-refusal.md`、`examples/failure-recovery.md`。
