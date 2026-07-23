---
tags: [phase-1, interview, standardized-answers, tools, routing, provider]
created: 2026-07-22
updated: 2026-07-22
status: complete
question_range: Q17-Q40
---

# Q17-Q40：Tool、Router 与 Provider 标准答案

## Q17. 一个生产 Tool 契约至少有哪些字段？

- **来源映射**：`INT-NC-B03` 报告 Tool 调用流程；`INT-CS-C01` 用作趋势校验。
- **30 秒答案**：至少要有稳定 ID/版本、用途与禁用场景、输入/输出 Schema、错误模型、超时、幂等与副作用等级、权限、成本/限流和审计字段。
- **2 分钟标准答案**：ToolSpec 不只是函数名和描述。模型看到的是可选择能力，Runtime 还需要 actor/tenant 权限、参数约束、超时和 deadline、retryable 错误、幂等键、补偿或状态查询接口、敏感字段脱敏、并发上限、版本兼容和 owner。结果统一为结构化 ToolResult，区分成功、业务拒绝、系统失败和未知结果。版本变化要能灰度和回放，不能悄悄改变同名工具语义。
- **深挖**：读工具和写工具是否同一标准？写工具要求更强的审批、幂等和补偿；描述写给谁？同时服务模型选择与工程治理；Schema 合法就能执行吗？还要业务与权限校验。
- **项目映射**：ESGQA 缺少统一 Registry，证据 `P1-ESG-GAP-TOOL-REGISTRY-001`；agentHub 有权限事件，但不等于完整 ToolSpec。
- **评分/危险**：只答 name/description/parameters 是 SDK 级答案，不是生产契约。

## Q18. Tool 参数应该按什么顺序验证？

- **来源映射**：`INT-NC-A02` 的工具错误题与本题互补。
- **30 秒答案**：先解析与 Schema，再做规范化和业务不变量，再绑定 actor/tenant 做授权，最后检查预算、幂等和执行前条件；任何修正后都要重新校验与授权。
- **2 分钟标准答案**：我会按 parse -> type/schema -> canonicalize -> semantic/business -> authz -> policy/risk -> budget/deadline -> idempotency/precondition 执行。顺序很重要：路径要先 canonicalize 才能做目录授权；金额和币种要规范化后再验证额度；审批必须绑定工具版本和参数摘要，不能批准后允许模型替换参数。默认值也属于真实参数，应出现在审计记录中。
- **深挖**：为什么授权不放第一步？需要先得到安全规范化的资源标识；模型能否自动修参数？可提议修复，但修后重走全链路；Pydantic 是否足够？只保证结构，不保证业务语义。
- **项目映射**：`P1-ESG-STRUCTURED-OUTPUT-001` 证明结构化输出，不证明通用工具参数安全；`P1-HUB-PERMISSION-001` 的 glob 限制说明路径规范化仍是缺口。
- **评分/危险**：授权后再偷偷补默认参数是典型 TOCTOU/审批替换风险。

## Q19. ToolResult 如何回注模型？

- **来源映射**：真实面经常追问工具结果处理；`INT-NC-B04` 提及工具返回。
- **30 秒答案**：用稳定、结构化、限长且带 provenance 的 Observation 回注，区分状态、数据、错误、可重试性和 artifact 引用，不直接拼接任意 stdout/HTML。
- **2 分钟标准答案**：ToolResult 至少包含 execution id、tool/version、status、typed payload、error code、retryable、side-effect state、source/time 和 artifact refs。大结果写入对象存储或工作区，只回摘要和引用；不可信网页或文档必须标记 data，不得被提升为 instruction。向模型展示的信息还要脱敏、截断和保留关键错误字段；同时把完整原始结果保存在审计域供排障。Runtime 根据 status 决定继续、修参、审批或终止，而不是让模型从自由文本猜。
- **深挖**：工具结果包含 Prompt Injection 怎么办？标记不可信来源并做控制/数据隔离；截断会丢信息怎么办？保留 artifact 和可分页读取；二进制怎么回注？元数据和受控解析结果。
- **项目映射**：agentHub 统一 Provider 事件可作为归一化思路，证据 `P1-HUB-PROVIDER-001`；不能外推所有 ToolResult 已满足上述契约。
- **评分/危险**：直接把 shell 输出完整拼进 system prompt 风险极高。

## Q20. Function Calling、Tool Registry、MCP 有何区别？

- **来源映射**：`INT-DY-C01`、`INT-CS-C01` 是趋势材料；A/B 级面经也多次出现 Function Calling/MCP。
- **30 秒答案**：Function Calling 是模型产出结构化调用意图的机制；Tool Registry 是应用运行时维护工具元数据、实例和策略的目录；MCP 是客户端与外部 Server 交换工具/资源/Prompt 的协议。它们可组合但互不替代。
- **2 分钟标准答案**：Function Calling 解决“模型怎样表达调用哪个函数和参数”，不执行函数也不授权。Registry 解决“当前 actor 可见哪些工具、版本、Schema、成本和执行器”，属于应用控制面。MCP 解决跨进程发现和调用工具/资源的互操作，但接入 MCP Server 后仍需本地鉴权、租户隔离、参数校验、出站策略和审计。典型链路是 MCP/本地适配器注册 ToolSpec，Runtime 按 actor 过滤，模型用 Function Calling 提议，Runtime 校验后执行。
- **深挖**：有 MCP 还要 Registry 吗？要，Registry 承担本地治理；有 Function Calling 就是 Agent 吗？不是；MCP Server 可信么？按外部依赖做零信任校验。
- **项目映射**：ESGQA 没有通用 Registry/MCP；agentHub 有工具与权限事件，但冻结证据不能说明完整 MCP 治理。
- **评分/危险**：把三者都叫“工具调用协议”会丢失层次。

## Q21. 写操作超时后能否直接重试？

- **来源映射**：`INT-NC-B02` 的失败重试与资金安全题。
- **30 秒答案**：不能。超时只说明客户端没收到确定结果，服务端可能已提交；应先用 idempotency key 或查询接口核验，再决定补偿、等待或重试。
- **2 分钟标准答案**：写操作有三种结果：明确失败、明确成功、未知结果。只有明确失败且服务端承诺未执行时才安全重试；未知结果先按 operation id 查询真实状态。设计上用业务幂等键、唯一约束、状态机和 outbox，必要时用 saga 补偿，但补偿也要幂等。重试必须沿用同一业务 id，不能生成新支付单。用户界面显示“处理中/待核验”，不要用模型文本猜成功。
- **深挖**：GET 一定幂等吗？通常语义如此，但仍看实现；补偿等于回滚吗？不一定，只是新的业务动作；如何处理迟到成功？状态机按版本和终态拒绝覆盖。
- **项目映射**：现有项目没有资金业务运行证据；这里只能作为通用设计，Q107 再做场景化。
- **评分/危险**：“超时重试三次”在写操作中可能造成重复扣款。

## Q22. 如何做 Tool 选择与权限分离？

- **来源映射**：`INT-NC-A01` 报告工具选择和限制。
- **30 秒答案**：模型只在经 actor/tenant/policy 过滤后的工具集合中选择候选；Runtime 在执行前对规范化参数做最终授权，选择权不等于执行权。
- **2 分钟标准答案**：先由能力目录按任务、角色、租户、环境和风险生成最小可见集合，再把简化后的 Schema 给模型。模型返回 candidate action 后，Runtime 使用真实身份、工具版本和参数重新做 authz/policy；高风险动作进入参数绑定的 HITL。即使模型通过 Prompt 请求隐藏工具，也没有执行器句柄。权限决定还应写入 Trace，包括规则版本和拒绝理由。
- **深挖**：隐藏工具是否等于安全？不是，执行层必须拒绝；能否用模型 confidence 跳过审批？不能；委派 Agent 怎么办？重新绑定新 actor，不继承超范围权限。
- **项目映射**：`P1-HUB-PERMISSION-001` 展示角色工具和写路径检查，也暴露 custom default 与 glob 风险。
- **评分/危险**：把完整工具列表给模型，再要求 Prompt“不要乱用”不是权限控制。

## Q23. ESGQA 为什么不能说已有通用 Tool 系统？

- **来源性质**：项目证据边界题。
- **30 秒答案**：因为冻结代码中的检索、计算、Web 等能力由特定图节点直接调用，没有统一 ToolSpec、Registry、标准 ToolResult、按工具权限和副作用元数据。
- **2 分钟标准答案**：能调用外部能力只能证明“节点集成了依赖”，不等于开放式 Tool Runtime。通用 Tool 系统要支持发现、Schema、版本、统一执行、错误、权限、预算、审计和动态可见性。ESGQA 的 Pydantic 模型主要约束 LLM 结构化输出，也不能替代 Tool 契约。若改造，我会先包装现有 retriever/calculator/web 为 typed adapter，再建 Registry 和 Policy，保持现有 StateGraph 路径不变，逐步迁移。
- **深挖**：节点函数能否注册成 Tool？可以作为建议，但当前未实现；为什么不直接上 MCP？先解决本地契约和权限，再考虑协议；改造代价？错误语义、测试矩阵和节点迁移。
- **项目映射**：`P1-ESG-GAP-TOOL-REGISTRY-001`、`P1-ESG-STRUCTURED-OUTPUT-001`。
- **评分/危险**：把 LangChain 依赖中的 Tool 能力说成项目已接通属于证据越界。

## Q24. agentHub 权限配置是否足以证明 Tool 安全？

- **来源性质**：项目安全评审题。
- **30 秒答案**：不足。它证明有角色级工具和写路径检查，但默认配置、路径 canonicalization、symlink/TOCTOU、执行器强制和容器边界仍需验证。
- **2 分钟标准答案**：代码检查顺序包括 forbiddenTools、allowedTools 和写路径 patterns，这是积极证据；但未知 Agent 使用较宽默认，空 allowlist 的语义不是禁止全部，glob 转正则也不等于真实路径授权。真正安全还需在执行层用 canonical path、workspace root、打开后校验、最小挂载、非 root、网络出站和审计；权限事件还要绑定 actor、tool version 和参数摘要。
- **深挖**：有 Docker 是否足够？不够；symlink 如何绕过？授权路径和实际打开目标不一致；如何测？负向 contract tests、路径穿越、链接替换和工具别名。
- **项目映射**：`P1-HUB-PERMISSION-001`、`P1-HUB-SANDBOX-001`、`P1-HUB-DOC-SECURITY-GAPS-001`。
- **评分/危险**：“有白名单所以绝对安全”忽略执行层和默认策略。

## Q25. Router 为什么不只是意图分类器？

- **来源映射**：A/B 面经反复追问模型怎样选工具和 Agent/RAG 选型。
- **30 秒答案**：生产 Router 还要考虑能力、权限、风险、成本、延迟、Provider 健康、上下文和 fallback；意图只是输入特征之一。
- **2 分钟标准答案**：分类器回答“用户大概想做什么”，Router 回答“在当前 actor、资源和 SLA 下允许走哪条路径”。它可能先用规则阻断高风险或无权限请求，再用模型处理长尾语义，最后根据 Provider/Tool 健康、预算和策略选择执行路径。输出应是 typed route decision，包括目标、理由码、置信信息、fallback 和策略版本，而不是只返回标签字符串。
- **深挖**：规则和模型谁先？安全硬规则优先，语义长尾再模型；如何处理 OOD？reject/clarify/fallback；Router 能否直接授权？不能替代 authz。
- **项目映射**：ESGQA `P1-ESG-ROUTING-001` 是结构化分类加固定映射；agentHub `P1-HUB-ROUTING-001` 是事件规则路由，两者路由对象不同。
- **评分/危险**：只讲 softmax 分类准确率没有运行时维度。

## Q26. 规则路由和模型路由如何组合？

- **来源性质**：生成训练题，真实面经中的 Agent/Tool 选择会追到该层。
- **30 秒答案**：硬约束、权限、显式命令和高风险场景用规则；开放语义和长尾意图用模型；模型结果再经过规则校验并配置 fallback。
- **2 分钟标准答案**：典型顺序是输入净化 -> hard deny/allow -> 显式规则 -> 模型分类或打分 -> 策略后处理 -> typed decision。规则保证可审计和稳定上界，模型提高语义覆盖。冲突时安全规则不可被模型覆盖；低置信或 OOD 进入澄清/默认 Workflow。评测要按规则命中、模型命中、冲突、fallback 分桶，避免总准确率掩盖高风险错误。
- **深挖**：规则多了如何维护？优先级、冲突检测、版本和命中统计；模型输出 JSON 就安全吗？仍需校验；默认路由怎样选？按最小权限和最小副作用。
- **项目映射**：ESGQA 的 intent 结构化输出与程序路由是可引用例子，但未知值默认 calculation 是待改进点。
- **评分/危险**：让模型覆盖安全规则是控制面倒置。

## Q27. 模型 confidence 能直接当概率吗？

- **来源性质**：生成训练题。
- **30 秒答案**：不能。自报 confidence、logit 或采样一致性通常未校准，分布变化后更不等于真实正确概率。
- **2 分钟标准答案**：先明确 confidence 来源：模型文本自评最弱；分类 head/logprob 也需在目标分布上校准。用留出集做 reliability diagram、ECE/Brier、温度缩放或 isotonic，并按任务、语言、风险分桶。路由阈值由误路由成本决定，高风险宁可 reject/HITL，不能用统一 0.8。线上监控覆盖率、选择性风险和漂移，定期重校准。
- **深挖**：多次采样一致是否可信？可作特征但不等于正确；阈值怎样定？成本矩阵和容量约束；LLM 自评何时有用？辅助排序，不做唯一授权信号。
- **项目映射**：两个项目没有置信校准证据，不应编造阈值效果。
- **评分/危险**：“模型说 90% 就是九成概率”错误。

## Q28. 如何评测 Router？

- **来源性质**：生成训练题；与 `INT-NC-B03` 的模型评估相关。
- **30 秒答案**：离线看每类 precision/recall、混淆、拒答覆盖和成本加权错误；线上看任务成功、fallback、延迟、成本和高风险误路由，并按分布切片。
- **2 分钟标准答案**：先构建包含正常、歧义、OOD、攻击和高风险样本的版本化 Golden Set。指标不能只看 accuracy，要有 macro-F1、关键类 recall、误授权率、clarification rate、coverage-risk curve。再做端到端回放，因为路由标签对不代表任务完成。上线通过 shadow/canary 比较旧新策略，并记录 route reason、模型/规则版本和后续结果用于归因。
- **深挖**：类别不均衡怎么办？分层抽样和每类指标；标签歧义？多标注者与 adjudication；线上成功能反推路由正确吗？只能作为弱标签，需防下游补救掩盖。
- **项目映射**：ESGQA 无路由评测数据；回答应提出测法，不声称已完成。
- **评分/危险**：只有总体准确率不足以评估高风险路由。

## Q29. fallback、retry、reject、HITL 有何区别？

- **来源映射**：`INT-NC-B02` 的失败、中断和金融安全题。
- **30 秒答案**：retry 是同路径处理可恢复失败；fallback 换能力或降级；reject 明确不执行；HITL 把决策或批准交给人，四者触发条件和风险不同。
- **2 分钟标准答案**：瞬时网络错误且副作用安全可 retry；Provider 不可用可 fallback 到能力兼容且安全策略等价的 Provider；请求非法、越权或超范围应 reject；高风险、歧义或不可逆动作进入 HITL。它们都需要原因码、预算和终止语义。Fallback 不能 silently 改变隐私地域、工具能力或质量承诺；HITL 审批要绑定参数和版本。
- **深挖**：模型拒答属于 reject 吗？是策略结果但仍由 Runtime记录；fallback 会重放 Prompt 吗？先脱敏和能力适配；HITL 超时？安全终止而非默认允许。
- **项目映射**：ESGQA 有 fallback 节点；agentHub 有权限事件和 Manager abort，但不代表四类策略完整实现。
- **评分/危险**：把所有异常统一“换模型重试”会扩大事故。

## Q30. 如何处理多条规则同时命中？

- **来源性质**：生成训练题；可用 agentHub 事件路由作源码评审。
- **30 秒答案**：先定义显式优先级和冲突策略，区分 first-match、all-match、互斥组与 deny-overrides，并记录全部候选和最终决策。
- **2 分钟标准答案**：规则模型要有稳定排序、唯一 ID、版本、scope 和组合语义。安全拒绝通常 `deny-overrides`；通知类可 all-match；互斥业务路由应在发布前做静态冲突检测，运行时若同优先级冲突则进入保守 fallback。Trace 保存命中集合、字段、优先级和结果，测试覆盖边界值与规则顺序变化。
- **深挖**：优先级相同怎么办？禁止发布或确定性 tie-break；first-match 有何风险？新规则遮蔽旧规则；怎样监控？命中率、遮蔽率、空路由率。
- **项目映射**：`P1-HUB-ROUTING-001` 按 priority 排序并可能多目标输出；不能声称冲突已形式化解决。
- **评分/危险**：依赖数组偶然顺序而不记录原因，难以审计。

## Q31. ESGQA 的路由能证明什么？

- **来源性质**：项目证据题。
- **30 秒答案**：能证明结构化意图输出进入程序化条件边，且错误优先走 fallback；不能证明置信校准、策略引擎、HITL 或所有未知意图安全处理。
- **2 分钟标准答案**：`node_intent_parser()` 通过 Pydantic `IntentOutput` 写入 `task_type`，`route_intent()` 映射到固定节点，各阶段 `error_msg` 转 fallback。这是“模型分类 + 程序路由”的混合例子。限制是路由函数没有再次做严格枚举校验，未知值落到 calculation/retriever 默认路径，可能把 OOD 误当计算任务。改进应加入显式 unknown、策略前置、路由 Trace 和离线分桶评测。
- **深挖**：为什么默认 calculation 有风险？副作用和错误答案；怎样兼容旧客户端？新增 unknown 不破坏已知枚举；是否支持动态工具选择？没有证据。
- **项目映射**：`P1-ESG-ROUTING-001`。
- **评分/危险**：不能把“有条件边”夸成企业策略平台。

## Q32. agentHub 的事件路由如何评审？

- **来源性质**：项目源码评审题。
- **30 秒答案**：检查规则 Schema、排序、匹配字段、多目标语义、空命中、冲突、重放确定性和 ID 稳定性，再看路由结果是否越过权限边界。
- **2 分钟标准答案**：代码按 priority 降序匹配 eventType/toolName/sender/exitCode/path/result，并为 notify target 生成 InboxEntry。积极点是规则显式、字段可审计、支持默认和自定义；风险是空数组表示隐式无动作，时间+随机 ID 不适合严格重放，多个规则可同时命中，路径 glob 和下游权限仍需分开验证。测试要覆盖同优先级、字段缺失、恶意路径、重复事件和配置热更新。
- **深挖**：路由与授权关系？路由只决定通知/流向，不自动授权动作；如何 exactly-once？需事件 ID、去重存储和事务边界；空命中怎么办？按事件级默认策略显式处理。
- **项目映射**：`P1-HUB-ROUTING-001`。
- **评分/危险**：只复述代码 happy path，不算完成评审。

## Q33. 最小 Provider 接口为什么不能只有 chat？

- **来源映射**：真实面经中的模型选择、推理框架和流式输出会追到 Provider 语义。
- **30 秒答案**：Agent Runtime 还需要生命周期、流式事件、工具/权限、取消、Usage、会话恢复、能力发现和错误语义；单一 `chat()` 无法可靠托管长任务。
- **2 分钟标准答案**：最小可用接口应包含 capability、start/resume、send、stream/onEvent、cancel/stop、usage、health 和统一错误；工具调用和权限请求通过 typed event 表达。消息转换要保留 role、tool call/result、artifact 和 provenance。接口不必追求所有厂商最小公倍数，而应显式声明 optional capability，调用前协商。
- **深挖**：为什么不把所有能力塞 complete()？无法取消和增量观测；流式文本够吗？还需工具、usage、终态；会话 ID 属于谁？Runtime 管逻辑会话，Provider 适配底层 ID。
- **项目映射**：`P1-HUB-PROVIDER-001` 的接口、Factory 和事件是直接例子。
- **评分/危险**：`def chat(prompt): str` 只能支持最简模型调用。

## Q34. 如何设计 capability matrix？

- **来源映射**：`INT-NC-A01/B02` 的模型选型题，详细项目答法见 Q85。
- **30 秒答案**：按能力维度记录支持级别、限制和验证版本，如 tool calling、structured output、vision、streaming、context、cancel、usage、region 和数据策略，而不是厂商布尔开关。
- **2 分钟标准答案**：每个 Provider/模型条目包含 model/version、能力、参数限制、事件语义、最大上下文、速率、价格、数据驻留和已通过的 contract suite。支持级别可为 native/emulated/unsupported，防止“能模拟”被写成原生。路由根据任务需求过滤，再按质量、成本和健康度选择。能力变化要版本化并持续探测。
- **深挖**：上下文长度写官方数字够吗？还要实测有效能力和输出预留；工具调用支持怎么验证？Schema、并行、错误和多轮 contract tests；价格能否硬编码？应配置和带时间戳。
- **项目映射**：agentHub Provider 数量有限，不能外推完整 matrix；ESGQA 当前单 Ollama Provider。
- **评分/危险**：大量 `if provider == ...` 会形成不可维护条件树。

## Q35. 如何统一流式事件？

- **来源映射**：B 级汇总出现流式通信；Q86 会处理部署层。
- **30 秒答案**：定义递增 sequence、event type、payload、timestamp、usage、terminal reason 和 correlation id，覆盖 text/tool/permission/error/completed/cancelled，而不是只统一文本 chunk。
- **2 分钟标准答案**：Provider adapter 把各 SDK 事件映射为内部 envelope：session/turn/execution id、seq、type、payload、provider metadata。Runtime 保证每 turn 只有一个终态，忽略或隔离终态后的迟到事件；工具调用与结果用 call id 关联；Usage 可增量或终态校正。前端通过 SSE/WebSocket 消费时要处理重连、last-event-id、去重和背压。
- **深挖**：能否保证全局顺序？通常只保证会话/turn 内；增量 usage 和最终值冲突？终态校正并保留来源；Provider 漏 completed？Runtime deadline 生成失败终态。
- **项目映射**：`P1-HUB-PROVIDER-001` 证明统一事件类别，但不证明所有顺序不变量。
- **评分/危险**：把不同 Provider 的字符串 chunk 直接透传会泄漏语义差异。

## Q36. Provider 错误如何分类？

- **来源性质**：生成训练题。
- **30 秒答案**：至少分输入/Schema、认证授权、限流、瞬时网络、容量、模型拒绝/安全、上下文超限、能力不支持、取消和未知结果，并带 retryable 与 side-effect 语义。
- **2 分钟标准答案**：adapter 把供应商错误码转换为稳定内部 taxonomy，同时保留原始 code 和 trace。认证失败不重试；限流按 retry-after；网络错误在 deadline 内退避；上下文超限触发裁剪/压缩；能力不支持触发兼容 fallback；安全拒绝不能换弱策略模型绕过。错误分类要驱动指标和用户信息，但不得泄露密钥或内部 Prompt。
- **深挖**：HTTP 500 都可重试吗？不一定；流中断属于什么？需判断是否已有副作用和是否可续传；未知错误？保守终止并保存 trace。
- **项目映射**：agentHub 有具体 Provider/Runtme 异常处理证据，但没有完整统一 taxonomy 的证明。
- **评分/危险**：所有错误统一 Exception 会使重试策略失真。

## Q37. 多 Provider fallback 的安全不变量？

- **来源映射**：模型选型与可用性追问常见；Q85 负责选择维度。
- **30 秒答案**：fallback 不能扩大工具权限、数据地域、上下文泄露、日志保留或安全策略，也不能静默降低结构化输出和业务验收。
- **2 分钟标准答案**：候选 Provider 必须通过 capability 和合规过滤；切换时只发送完成任务所需的脱敏上下文，重新计算 token budget，重新绑定 tool schema，并保持 actor/policy 不变。若目标 Provider 不支持 required capability，应明确降级或失败，不能用文本模拟后继续高风险动作。Trace 记录原 Provider、原因、目标、上下文摘要和结果，便于成本与质量归因。
- **深挖**：安全拒绝能否换模型？不能用于绕过策略；不同地域 API 呢？先过数据驻留规则；fallback 后能否沿用会话？需语义兼容和状态迁移验证。
- **项目映射**：`P1-HUB-PROVIDER-001` 支持多实现，不能证明上述所有不变量已实现。
- **评分/危险**：“失败就轮询所有模型直到有答案”会造成合规和一致性问题。

## Q38. 如何证明一个新 Provider 真正接通？

- **来源性质**：生成训练题；也用于识别“安装 SDK 就算支持”的包装。
- **30 秒答案**：通过 contract suite 验证生命周期、消息、流式、工具、权限、取消、Usage、错误、会话恢复和终态，而不是只看一次文本响应。
- **2 分钟标准答案**：测试矩阵包含正常文本、结构化输出、单/多工具、工具错误、并行调用、权限拒绝、取消、超时、上下文超限、流断开、Usage 和恢复。记录真实命令、版本和输出，区分 mock、沙箱和生产。再做与 Runtime 的集成测试，检查事件顺序、队列和迟到事件。最后以 capability matrix 发布，不支持项明确标记。
- **深挖**：单测过了为什么还要端到端？SDK 和事件组合会出错；如何测取消？长任务发起后取消并验证无终态覆盖；如何证明费用？保存 provider usage 与内部统计对账。
- **项目映射**：agentHub 冻结提交无 Codex Provider，证据 `P1-HUB-GAP-CODEX-PROVIDER-001`；依赖存在不算接通。
- **评分/危险**：一次“hello world”成功不能证明 Agent Provider 完成。

## Q39. ESGQA 的 Provider 现状如何表达？

- **来源性质**：项目事实题。
- **30 秒答案**：冻结代码多处直接构造 `ChatOllama`，是 Ollama 单 Provider 集成；没有 Provider Protocol、Factory、能力发现和统一事件。
- **2 分钟标准答案**：我会说“当前模型调用与业务节点耦合，已经能使用 Ollama 完成特定结构化任务，但多模型切换是未实现的改造方向”。改造先抽最小 adapter，保留现有 Pydantic 输出，再增加 error/usage/capability 和 contract tests；只有新 Provider 通过完整测试后才宣称支持。不能把 LangChain 的模型生态当成项目已实现能力。
- **深挖**：为何不一开始做大而全接口？避免过度抽象；结构化输出跨 Provider 怎么办？能力协商与兼容层；迁移风险？Prompt/Schema 语义和错误差异。
- **项目映射**：`P1-ESG-GAP-PROVIDER-001`。
- **评分/危险**：“换一行 model name 就支持所有厂商”不成立。

## Q40. agentHub 是否支持 Codex Provider？

- **来源性质**：项目证据核验题。
- **30 秒答案**：冻结提交 `815f645` 不支持。Factory 只注册 `claude-code`、`opencode` 和 `test`，归档中没有 Codex Provider；依赖清单出现 SDK 也不能算实现。
- **2 分钟标准答案**：支持的判定必须落到 Factory 注册、Provider 实现、事件适配和 contract suite。当前证据只说明项目可能准备了依赖或未来方向。若增加 Codex，需要对齐会话、工具调用、权限请求、取消、Usage、压缩恢复和错误语义，再在 Runtime 上做集成测试。在完成前只能说“有明确改造路径”，不能说“已支持”。
- **深挖**：为什么依赖不算？没有运行入口和语义适配；HTTP 能返回文本算不算？只算最小调用，不算 Provider；如何防接口抽象泄漏？optional capability 和专用 metadata。
- **项目映射**：`P1-HUB-GAP-CODEX-PROVIDER-001`、`P1-HUB-PROVIDER-001`。
- **评分/危险**：面试中主动承认该缺口比虚构实现更可信。


