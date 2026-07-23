---
tags: [phase-1, interview, question-bank, project-evidence, review]
created: 2026-07-22
updated: 2026-07-23
status: complete
question_count: 84
---

# Phase 1 大模型 Agent 面试题库

> [!important] 题库口径
> 以下是依据 Phase 1 知识、公开技术资料和两个冻结项目源码生成的训练题，不声称是任何公司的真实原题。项目回答必须遵守 [[project-evidence-map]]：只把源码闭环标为已实现，把设计补全标为改进建议。
>
> [!important] 114 题扩展入口
> 本文件保留 Q1-Q84 精简题卡；公开面经来源、Q85-Q114 和全部详细答案从 [[interview-question-bank-expanded]] 进入。

## 1. 使用方法与统一评分

每题依次练三档：30 秒先给结论；2 分钟给结构、权衡与项目证据；深挖时给失败模式、指标和改造。不要逐字背诵，先独立口述再对照。

统一 5 分制：

- **1 分**：只会术语或框架 API。
- **2 分**：能讲基本流程，但无边界和失败处理。
- **3 分**：有状态、契约、指标和至少一个真实项目证据。
- **4 分**：能比较方案、解释权衡，并准确区分已实现与建议。
- **5 分**：能应对反例，给出验证方案、生产不变量和源码级证据。

## 2. Lesson 1.1：Agent 定义与第一性原理

### Q1. 什么是 Agent？它与一次 LLM 调用有何区别？
- **30 秒**：Agent 是在 Runtime 中围绕目标反复感知状态、选择动作、执行并观察结果的受约束系统；LLM 通常只是策略组件。
- **2 分钟/深挖**：拆成目标、状态、策略、动作、环境、观察、终止和治理；补充 Tool/Router/Memory/Policy/Trace。讨论一次调用也可在外层形成 Agent step，但不能因用了模型就称 Agent。
- **评分点**：状态闭环、环境反馈、终止与安全；能引用 `P1-ESG-STATEGRAPH-001` 或 `P1-HUB-RUNTIME-001`。
- **危险答案**：会调用函数的 Chatbot 就是 Agent；模型自己负责所有权限。

### Q2. Workflow、Agent 和 Multi-Agent 如何区分？
- **30 秒**：Workflow 的控制流主要由代码预定义；Agent 在受控候选中动态决策；Multi-Agent 增加角色、通信和协调，不自动更智能。
- **2 分钟/深挖**：用确定性、可测试性、开放性和成本比较；高风险固定流程优先 Workflow，语义长尾才交给模型；Multi-Agent 要付出路由、上下文、死锁与责任归因成本。
- **评分点**：不做二元划分，能解释混合架构；ESGQA 是领域图工作流，不应夸成任意自治 Agent。
- **危险答案**：节点越多越 Agent；Multi-Agent 一定比单 Agent 强。

### Q3. 为什么说 Runtime 比 Prompt 更接近 Agent 的工程核心？
- **30 秒**：Prompt 提供决策上下文，Runtime 才维护状态、调工具、处理错误、预算、权限和终止。
- **2 分钟/深挖**：说明 Prompt 不拥有资源、不保证重试幂等、不处理取消和迟到事件；以 agentHub Provider 事件、busy queue、permission、compression 生命周期举证。
- **评分点**：能画模型与 Runtime 边界；提到策略否决和 trace。
- **危险答案**：只要系统 Prompt 足够长就能实现可靠 Agent。

### Q4. Agent 状态应该包含什么，不应该包含什么？
- **30 秒**：包含完成目标所需的业务状态、执行状态、错误、预算和引用；连接对象、密钥等本地依赖不应直接进入模型可见状态。
- **2 分钟/深挖**：区分 durable state、working state、model-visible context；说明 reducer、并发合并、版本和敏感字段；用 `CBAMState` 举例并讨论字段膨胀。
- **评分点**：能说明 Schema、所有权和序列化边界。
- **危险答案**：把所有对象塞进一个 dict；日志和密钥也写进 Prompt 便于排障。

### Q5. 如何定义 Agent 的完成与停止？
- **30 秒**：完成需满足业务验收；停止还包括拒绝、预算耗尽、deadline、不可恢复错误、人工中止和最大步数。
- **2 分钟/深挖**：定义 terminal reason、未完成输出、补偿和审计；区分成功终止与安全终止；防循环用 hop、重复状态和 progress detector。
- **评分点**：明确终态与原因码；不把生成最终文本等同任务成功。
- **危险答案**：模型说 done 就结束；异常统一继续尝试。

### Q6. 设计 Agent 时第一性原理的顺序是什么？
- **30 秒**：先目标与验收，再环境和动作，再状态与反馈，最后选择模型和框架。
- **2 分钟/深挖**：列资产/风险、可确定流程、最小能力、失败恢复、可观测指标；只有不可枚举语义决策才引入模型。
- **评分点**：需求和风险先于框架；能提出 non-goal。
- **危险答案**：先选 LangGraph/某 SDK，再找业务场景。

### Q7. 用 ESGQA 证明你理解 Agent，但不能夸大什么？
- **30 秒**：它有 `CBAMState`、显式节点和条件边，可证明领域状态化编排；不能据此声称通用 Tool Registry、多 Provider 或任意自治 ReAct。
- **2 分钟/深挖**：讲 `START -> memory -> intent -> route -> domain nodes -> END`，补充结构化输出和错误边；再列 `P1-ESG-GAP-TOOL-REGISTRY-001`、`P1-ESG-GAP-PROVIDER-001`、`P1-ESG-GAP-SANDBOX-001` 三类缺口。
- **评分点**：调用链与边界成对出现。
- **危险答案**：把 LangGraph 的能力全部说成项目已实现。

### Q8. 用 agentHub 证明 Runtime 能力，并说明风险。
- **30 秒**：它以 Provider 事件驱动会话，维护队列、权限、压缩和 idle 生命周期；但不同 Provider 语义和安全配置仍需契约测试。
- **2 分钟/深挖**：从 Factory/start/send/onEvent 到队列与终态；讨论 busy 时入队、压缩 pending prompt、权限事件和失败恢复。
- **评分点**：`P1-HUB-RUNTIME-001`；能说出事件顺序与并发风险。
- **危险答案**：EventEmitter 自动保证事件有序、恰好一次。

## 3. Lesson 1.2：Agent Loop 与编排模式

### Q9. 写出生产级 Agent Loop 的基本状态机。
- **30 秒**：observe -> decide/plan -> validate -> authorize -> act -> observe -> update -> terminate/retry。
- **2 分钟/深挖**：为每步补 input/output、deadline、错误、trace；模型只产生候选动作，Runtime 校验授权；终止包含成功、拒绝、取消、预算与失败。
- **评分点**：不是裸 while；有策略门和停止原因。
- **危险答案**：无限循环直到模型输出 final。

### Q10. ReAct、Plan-and-Execute、ReWOO 如何选择？
- **30 秒**：ReAct 每步适应观察；Plan-and-Execute 先分解再执行；ReWOO 倾向把规划与观察执行解耦，减少重复推理。
- **2 分钟/深挖**：按环境动态性、依赖 DAG、成本、可恢复性比较；开放环境用短步反馈，稳定可并行子任务用显式计划；引用 `SRC-REACT-001/SRC-REWOO-001`，不外推论文性能。
- **评分点**：能讲 trade-off 和混合使用。
- **危险答案**：背论文名，不说明状态和失败恢复。

### Q11. 为什么 Agent Loop 必须有 step budget、token budget 和 deadline？
- **30 秒**：分别限制动作次数、模型消耗和墙钟时间，防循环、成本失控和用户超时。
- **2 分钟/深挖**：绝对 deadline 向下传播；每次决策前做剩余预算检查；终止时返回部分结果与原因；监控重复状态和无进展。
- **评分点**：能区分三类预算并说明组合。
- **危险答案**：只设最大重试 3 次就足够。

### Q12. Loop 中工具失败如何处理？
- **30 秒**：按参数、权限、瞬时、业务、不可恢复分类；仅可重试错误有界重试，副作用需幂等或补偿。
- **2 分钟/深挖**：参数错误可回模型修正；权限拒绝终止或审批；限流尊重 retry-after；未知错误保守终止；ToolResult 结构化回注。
- **评分点**：错误 taxonomy、幂等和 trace。
- **危险答案**：捕获 Exception 后把字符串发回模型无限修。

### Q13. 如何并行执行 Agent 子任务？
- **30 秒**：先构建依赖 DAG，只并行无依赖且资源/副作用不冲突的节点，再确定性合并结果。
- **2 分钟/深挖**：调度 ready set、并发上限、取消传播、部分失败策略、join reducer；说明并行会增加上下文合并和资源竞争。
- **评分点**：引用 `SRC-LLMCOMPILER-001` 只支持架构思想；不声称项目已有任意 DAG 编译器。
- **危险答案**：Promise.all 所有 Tool 即可。

### Q14. 状态图相比隐式 Loop 有何优势？
- **30 秒**：节点、边、状态与终止显式，便于可视化、单测、恢复和审计。
- **2 分钟/深挖**：讨论条件边、checkpoint、reducer、子图、不可达节点和循环上限；代价是图定义与状态迁移维护。
- **评分点**：用 ESGQA `build_cbam_graph` 举证。
- **危险答案**：使用 LangGraph 就天然可靠和可恢复。

### Q15. Planner 输出错误怎么办？
- **30 秒**：Schema 验证、业务不变量校验、有界修复重试；持续失败转失败审查、澄清或终止。
- **2 分钟/深挖**：校验工具存在、依赖无环、参数可得、权限/预算；把 validation errors 结构化反馈，不重放全部不可信历史。
- **评分点**：`P1-HUB-PLANNER-001` 的有限重试；说明未证明计划最优。
- **危险答案**：让另一个 LLM 判断即可，无需确定性校验。

### Q16. ESGQA 图与 agentHub Runtime Loop 的根本差异？
- **30 秒**：ESGQA 以领域状态图和条件边驱动；agentHub 以 Provider 事件、队列和会话生命周期驱动。
- **2 分钟/深挖**：比较控制权、状态位置、扩展点、错误与压缩；说明二者都不是简单 while，但不能互相借用未实现能力。
- **评分点**：双项目证据准确、边界清楚。
- **危险答案**：两个项目都实现了同一种 ReAct Loop。

## 4. Lesson 1.3：Tool-use 设计

### Q17. 一个生产 Tool 契约至少有哪些字段？
- **30 秒**：名称/描述、输入 Schema、输出/错误、权限、风险、副作用、幂等、timeout、审批和可观测元数据。
- **2 分钟/深挖**：ToolSpec、ToolRequest、ToolResult、ToolError 四层；区分模型可见描述与 Runtime 私有策略；版本兼容和弃用。
- **评分点**：不仅是函数和 JSON Schema。
- **危险答案**：函数签名加装饰器就是完整 Tool。

### Q18. Tool 参数应该按什么顺序验证？
- **30 秒**：先限制体积和基础解析，再 Schema/类型，再主体与资源授权，再业务不变量和风险审批，最后执行。
- **2 分钟/深挖**：避免在授权前解析昂贵或危险对象；canonicalize 与 authz 的顺序需防歧义；审批绑定规范化参数。
- **评分点**：考虑 DoS、路径与 TOCTOU。
- **危险答案**：Pydantic 通过就直接执行。

### Q19. ToolResult 如何回注模型？
- **30 秒**：结构化状态、必要字段、来源和可恢复错误；大对象存外部句柄，不回注秘密和原始噪声。
- **2 分钟/深挖**：关联 call ID、成功/失败枚举、retryable、用户可见与模型可见分离；外部内容标记不可信，不把网页指令提升为系统指令。
- **评分点**：上下文预算和注入风险同时考虑。
- **危险答案**：异常 `str(e)` 和完整日志全部塞回 Prompt。

### Q20. Function Calling、Tool Registry、MCP 有何区别？
- **30 秒**：Function Calling 是模型 API 产生结构化调用；Registry 是应用内能力注册与执行控制；MCP 是 Host/Client/Server 之间发现和调用能力的协议。
- **2 分钟/深挖**：解释三者如何组合及信任边界；MCP Server 提供工具不代表调用已授权；引用 `SRC-MCP-SPEC-001`。
- **评分点**：层次正确，不把协议当执行沙箱。
- **危险答案**：用了 MCP 就自动安全、自动跨模型兼容。

### Q21. 写操作超时后能否直接重试？
- **30 秒**：不能；超时表示结果未知，需幂等键、操作状态查询或补偿，确认未提交后再重试。
- **2 分钟/深挖**：区分 at-most-once、at-least-once 与 effectively-once；Tool call ID、业务 idempotency key、outbox/对账。
- **评分点**：副作用状态而非网络状态决定重试。
- **危险答案**：指数退避可以解决重复写。

### Q22. 如何做 Tool 选择与权限分离？
- **30 秒**：模型从候选工具中提出调用；Policy Engine 根据 actor/resource/action 决定 allow/deny/approval；Runtime 执行。
- **2 分钟/深挖**：候选集合先按 capability 与权限预过滤，但最终仍逐次授权；高风险参数变化重新审批；拒绝结果不可被 Prompt 覆盖。
- **评分点**：Confused Deputy 意识。
- **危险答案**：模型 confidence 高就可跳过审批。

### Q23. ESGQA 为什么不能说已有通用 Tool 系统？
- **30 秒**：它有领域节点和外部调用，但没有发现通用 Registry、统一 Tool Schema、权限、生命周期和任意模型选工具闭环。
- **2 分钟/深挖**：用 `P1-ESG-STRUCTURED-OUTPUT-001` 说明可复用基础，用 `P1-ESG-GAP-TOOL-REGISTRY-001` 说明缺口；建议先抽纯函数和只读检索。
- **评分点**：节点不等于 Tool；建议与事实分离。
- **危险答案**：每个 LangGraph node 都是完整 Tool。

### Q24. agentHub 权限配置是否足以证明 Tool 安全？
- **30 秒**：不能；它证明有角色工具和写路径配置，还需核验空列表语义、真实路径、审批、沙箱和运行时执行链。
- **2 分钟/深挖**：引用 `P1-HUB-PERMISSION-001`；说明 glob、未知角色默认、symlink/TOCTOU 和参数绑定问题。
- **评分点**：能从配置追到条件分支。
- **危险答案**：有 allowlist 文件就等于默认拒绝。

## 5. Lesson 1.4：Router 与决策

### Q25. Router 为什么不只是意图分类器？
- **30 秒**：它还决定能力、策略和故障路径，并输出可审计决策。
- **2 分钟/深挖**：规则、模型、Policy、Runtime 四层；`RouteDecision` 包含 reason、confidence、rule、policy、fallback、version、trace。
- **评分点**：执行权不在裸标签。
- **危险答案**：取 logits 最大类别即可。

### Q26. 规则路由和模型路由如何组合？
- **30 秒**：高确定性与高风险条件先规则/策略，模型补语义长尾，低置信度澄清，最终安全默认。
- **2 分钟/深挖**：候选过滤、优先级、specificity、tie-breaker、terminal/composition；模型输出再做业务与策略验证。
- **评分点**：分层和冲突语义。
- **危险答案**：所有规则都写进 Prompt 让模型决定。

### Q27. 模型 confidence 能直接当概率吗？
- **30 秒**：不能；需要自有标注数据校准和按业务风险设阈值。
- **2 分钟/深挖**：reliability/ECE/Brier、分桶漂移；高风险动作即使高置信度仍需策略；低置信度澄清或只读 fallback。
- **评分点**：校准与业务成本。
- **危险答案**：temperature=0 后 confidence 就可靠。

### Q28. 如何评测 Router？
- **30 秒**：per-route precision/recall、校准、拒绝/澄清率、业务成本、P95 延迟与线上结果。
- **2 分钟/深挖**：成本矩阵区分误路由损失；规则冲突、未知输入、候选不可用、策略拒绝、fallback 测试；版本化回放。
- **评分点**：不只总体 accuracy。
- **危险答案**：离线准确率 95% 就可以上线。

### Q29. fallback、retry、reject、HITL 有何区别？
- **30 秒**：fallback 换受约束替代路径；retry 重做同类动作；reject 明确不执行；HITL 暂停等待授权或判断。
- **2 分钟/深挖**：结合错误可恢复性、权限、幂等和风险；fallback 不得扩大权限；HITL 参数变化失效。
- **评分点**：能给四个实际例子。
- **危险答案**：失败统一切更强模型。

### Q30. 如何处理多条规则同时命中？
- **30 秒**：显式 priority、specificity、稳定 tie-breaker、terminal 和 action composition。
- **2 分钟/深挖**：默认规则最后；冲突输出 trace；配置加载时做静态检查，测试覆盖同优先级、互斥动作和无匹配。
- **评分点**：确定性重放。
- **危险答案**：依赖数组当前顺序，不记录版本。

### Q31. ESGQA 的路由能证明什么？
- **30 秒**：能证明结构化意图驱动条件边，以及节点错误状态驱动后续分支。
- **2 分钟/深挖**：讲 `route_intent`、`check_*` 与 `build_cbam_graph`；说明没有通用 Router 服务、置信度校准和成本评测证据。
- **评分点**：`P1-ESG-ROUTING-001`。
- **危险答案**：项目已实现任意 Tool/Provider/Agent 的统一路由。

### Q32. agentHub 的事件路由如何评审？
- **30 秒**：按 priority 匹配 eventType、toolName、sender、exitCode、path、result 并产生通知目标。
- **2 分钟/深挖**：检查无命中空数组、默认规则、自定义替换、同优先级稳定性、ID 可重放和目标不可用；引用 `P1-HUB-ROUTING-001`。
- **评分点**：从规则定义讲到失败语义。
- **危险答案**：有 priority 就不存在冲突。

## 6. Lesson 1.5：Provider 抽象

### Q33. 最小 Provider 接口为什么不能只有 chat？
- **30 秒**：还需 capability、start/stream/cancel/resume/stop、事件、Usage 和错误规范化。
- **2 分钟/深挖**：消息、工具、会话、流式、终态和原生扩展；稳定核心加 capability matrix，无法无损映射时显式降级。
- **评分点**：生命周期与抽象泄漏。
- **危险答案**：OpenAI-compatible 端点都完全一样。

### Q34. 如何设计 capability matrix？
- **30 秒**：声明 streaming、tool calling、structured output、resume、cancel、usage、context 等能力并带版本。
- **2 分钟/深挖**：required capability 由请求声明；Router 只选满足者；能力不可仅由模型名猜测，缓存要有有效期。
- **评分点**：能力发现驱动路由。
- **危险答案**：在业务代码散落 provider name 判断。

### Q35. 如何统一流式事件？
- **30 秒**：统一 run ID、单调 seq、typed event 和唯一 terminal，并保留 native metadata。
- **2 分钟/深挖**：started/delta/tool/usage/completed|failed|cancelled；验证 completed 后 delta、重复 tool complete、迟到事件；背压和取消传播。
- **评分点**：事件不变量。
- **危险答案**：EventEmitter 发出来就算统一。

### Q36. Provider 错误如何分类？
- **30 秒**：认证、权限、限流、瞬时网络、不可用、非法请求、上下文溢出、内容策略、工具协议和未知。
- **2 分钟/深挖**：哪些可重试、retry-after、deadline、熔断维度；保留 provider code/request ID；权限错误不能换 Provider 绕过。
- **评分点**：错误驱动 Runtime 决策。
- **危险答案**：4xx 不重试、5xx 全重试。

### Q37. 多 Provider fallback 的安全不变量？
- **30 秒**：权限、工具、指令、输出契约、数据地域、副作用和预算不能被放宽。
- **2 分钟/深挖**：Provider B 不支持 strict output 时高风险任务应终止/排队；已产生 Tool 副作用时不得重新生成重复动作。
- **评分点**：降级不是无条件可用性优化。
- **危险答案**：失败就轮询所有模型直到成功。

### Q38. 如何证明一个新 Provider 真正接通？
- **30 秒**：有 Adapter、Factory 注册、Runtime 调用、事件映射、会话/工具/取消/Usage/错误和 contract tests 闭环。
- **2 分钟/深挖**：正常、限流、认证失败、取消、迟到事件、工具审批、上下文溢出、队列重放；不是 package 依赖或单次 API 返回。
- **评分点**：完整证据链。
- **危险答案**：SDK 已安装且能 import。

### Q39. ESGQA 的 Provider 现状如何表达？
- **30 秒**：业务节点直接使用特定模型集成，未形成通用 Provider 接口和多 Provider Runtime。
- **2 分钟/深挖**：先盘点结构化输出、流式等真实依赖，再抽最小 Gateway，以当前 Adapter 和第二实现的 contract test 验证。
- **评分点**：`P1-ESG-GAP-PROVIDER-001`。
- **危险答案**：LangChain 支持很多模型，所以项目已支持。

### Q40. agentHub 是否支持 Codex Provider？
- **30 秒**：冻结提交不支持；Factory 只注册 claude-code、opencode、test，没有 Codex Provider 文件。
- **2 分钟/深挖**：依赖存在不等于运行时实现；新增时对齐接口、事件、会话、工具、权限、取消和 Usage，并加 contract tests。
- **评分点**：`P1-HUB-GAP-CODEX-PROVIDER-001`。
- **危险答案**：package 里有 Codex SDK，所以已经支持。

## 7. Lesson 1.6：Context Engineering

### Q41. 本地上下文和模型可见上下文有何区别？
- **30 秒**：本地上下文是 Runtime 依赖和状态；只有序列化进消息、工具或输入的内容才被模型看到并消耗窗口。
- **2 分钟/深挖**：再区分外部记忆和执行状态；密钥/连接不进入模型；引用 `SRC-OPENAI-CONTEXT-001`。
- **评分点**：层次和安全边界。
- **危险答案**：Agent context 对象里的字段模型都会知道。

### Q42. 写出上下文预算方程。
- **30 秒**：window >= fixed + history + memory + retrieval + tools + input + output reserve + safety margin。
- **2 分钟/深挖**：固定区超限应失败；实际 tokenizer/Usage 优先；先预留输出；按 pinned/optional 分配。
- **评分点**：能手算并说明估算误差。
- **危险答案**：输入塞满后让 API 自动截断。

### Q43. 上下文超限时先删什么？
- **30 秒**：先去重和噪声、裁剪 ToolResult、按权限检索，再摘要和压缩关键历史。
- **2 分钟/深挖**：保护系统约束、当前请求、未完成计划和关键事实；大对象外存句柄；摘要保留来源。
- **评分点**：便宜减量顺序。
- **危险答案**：只保留最近十轮。

### Q44. 长窗口为何仍需要检索与排序？
- **30 秒**：容量不等于均匀利用，还涉及成本、延迟、权限、更新、重复和位置效应。
- **2 分钟/深挖**：引用 `SRC-LOST-MIDDLE-001`；关键证据靠近决策点，但位置不替代事实和权限校验。
- **评分点**：研究结论边界，不声称所有模型相同。
- **危险答案**：窗口足够大就不用 RAG。

### Q45. 记忆系统的生命周期是什么？
- **30 秒**：写入要有来源/主体/置信度；读取按相关性和权限；支持冲突、TTL、修订、删除、归档与审计。
- **2 分钟/深挖**：episodic/semantic/procedural/working；跨租户隔离、持久注入与用户可控性。
- **评分点**：向量库只是检索组件。
- **危险答案**：把所有聊天 embedding 后就是长期记忆。

### Q46. 压缩为什么要状态机和迟滞？
- **30 秒**：压缩期间新输入要暂存；单阈值会反复触发；摘要失败需可恢复。
- **2 分钟/深挖**：active/pending/compressing/restarting；75% 触发、45% 目标仅为例；冷却和最小新增 Token；保真探针。
- **评分点**：并发竞态与质量评测。
- **危险答案**：超过 70% 总结即可且不会丢信息。

### Q47. ESGQA 上下文实现如何评价？
- **30 秒**：有最近对话、长期画像、FAISS 兜底和混合检索；缺显式 Token 预算、记忆治理和实测质量证据。
- **2 分钟/深挖**：讲 SQLite 最近 4 条 -> 画像 -> FAISS；Redis -> BM25/FAISS -> rerank；指出固定 user ID 与索引信任风险。
- **评分点**：`P1-ESG-MEMORY-001/P1-ESG-RETRIEVAL-001`。
- **危险答案**：已有完整企业级记忆，检索准确率很高。

### Q48. agentHub ContextBus 与压缩有什么亮点和缺口？
- **30 秒**：结构化条目按类型、时间和引用在预算内选取；Usage 超 70% 进入压缩状态机并暂存新 Prompt。权重、阈值和摘要保真仍是缺口。
- **2 分钟/深挖**：版本、归档/GC、字符截断；summary 后 Provider 重启；提出 token-aware、迟滞、pinned facts、质量门。
- **评分点**：`P1-HUB-CONTEXT-001/P1-HUB-COMPRESSION-001`。
- **危险答案**：70% 是行业最佳阈值。

## 8. Lesson 1.7：Agent 安全

### Q49. 如何从零做 Agent threat model？
- **30 秒**：列资产、主体、入口、信任边界、滥用路径，再为每条威胁设计预防、检测和响应。
- **2 分钟/深挖**：User/外部内容 -> API -> Runtime/Policy -> Tool Broker -> Sandbox -> 外部系统；数据、凭证、主机、预算和可用性。
- **评分点**：不是只背 OWASP 类别。
- **危险答案**：装一个内容过滤器即可。

### Q50. 为什么 Prompt Injection 不能只靠 Prompt 解决？
- **30 秒**：自然语言指令和数据边界模糊，模型可能误判；真正权限必须由确定性 Runtime 强制。
- **2 分钟/深挖**：直接、间接、持久、跨 Agent 注入；内容标签降低风险，Tool authz、最小权限、HITL、沙箱限制后果。
- **评分点**：防诱导与限制影响两条线。
- **危险答案**：system prompt 写得足够严格可彻底消灭。

### Q51. authn、authz、sandbox 有何区别？
- **30 秒**：认证是谁，授权能否对资源做动作，沙箱限制即使获准执行后的影响范围。
- **2 分钟/深挖**：JWT、业务资源授权、Tool 参数授权、容器 mount/network/resource；任何一层不能替代另一层。
- **评分点**：用 ESGQA 入口安全举反例。
- **危险答案**：有 JWT 就能防文件越权。

### Q52. 文件 Tool 如何防路径遍历、symlink 和 TOCTOU？
- **30 秒**：受控根下规范化、拒绝绝对/越界、打开时不跟随链接、基于目录句柄操作、精确 mount，并在使用时而非只在字符串阶段校验。
- **2 分钟/深挖**：审批绑定 canonical action；临时文件原子替换；并发/重解析点；glob 只是策略表达，不是完整安全原语。
- **评分点**：多层防护与竞态意识。
- **危险答案**：检查字符串以 workspace 开头即可。

### Q53. 使用 Docker 后还要检查什么？
- **30 秒**：用户/root、mount、read-only rootfs、capabilities、no-new-privileges、seccomp/LSM、network、resource、image、Docker socket。
- **2 分钟/深挖**：共享内核、daemon 权限、rootless、egress、PID/disk/deadline；配置测试与运行审计。
- **评分点**：至少七个面并能解释作用。
- **危险答案**：容器天然等于虚拟机级安全。

### Q54. HITL 如何避免审批疲劳和参数替换？
- **30 秒**：按风险分层，仅高风险审批；展示具体 diff/目标；批准绑定 actor、参数、资源、有效期，变化即失效。
- **2 分钟/深挖**：本次/会话/范围授权、撤销与审计；Trust 模式约束；低风险只读可预授权。
- **评分点**：人审是策略状态，不是弹窗装饰。
- **危险答案**：所有动作都弹一次允许。

### Q55. ESGQA 的安全能力边界？
- **30 秒**：有 JWT、限流、Celery 隔离长任务和指标；没有证据表明有 Agent 文件/命令沙箱。
- **2 分钟/深挖**：入口身份与频率不等于资源授权和 Tool isolation；审计 secret、tenant、日志；引用 `P1-ESG-API-SECURITY-001/P1-ESG-GAP-SANDBOX-001`。
- **评分点**：不贬低已实现，也不扩大。
- **危险答案**：Celery worker 就是安全沙箱。

### Q56. agentHub 的权限与沙箱最值得追问什么？
- **30 秒**：空 `allowedTools` 实际不等于 deny all；glob 不能证明真实路径安全；threat model 还列出 rootfs、seccomp/AppArmor、Docker socket 和写边界缺口。
- **2 分钟/深挖**：per-session Docker 和权限配置是基础；优先修显式默认拒绝、精确 mount/socket、非 root/cap/seccomp/network，再做配置和负向测试。
- **评分点**：`P1-HUB-SANDBOX-001/PERMISSION-001/DOC-SECURITY-GAPS-001`。
- **危险答案**：项目有 Docker 和 allowlist，所以已经生产安全。

## 9. 跨课系统设计题

### Q57. 设计一个企业知识问答 Agent。
- **30 秒**：显式工作流：鉴权 -> 意图路由 -> 权限过滤检索 -> 生成 -> 证据校验 -> 回答；只读工具最小化，Context 有预算，完整 trace。
- **2 分钟/深挖**：Provider capability、混合检索/重排、拒答、租户过滤、Prompt Injection、Usage/成本、RAG 评测；用 ESGQA 作证据但不捏造线上指标。
- **评分点**：七课形成闭环。
- **危险答案**：把所有文档放长上下文即可。

### Q58. 设计一个能改代码的 Agent Runtime。
- **30 秒**：Planner/Loop 产生候选修改，Tool Broker 授权，per-session sandbox 执行，测试与 diff 门，HITL 合并，Provider 事件可取消，Context 压缩可恢复。
- **2 分钟/深挖**：路径/命令/network、workspace mount、幂等、重试、副作用、审计、kill switch；agentHub 能证明部分 Runtime 与沙箱基础，安全缺口如实说明。
- **评分点**：模型不直接拥有宿主 shell。
- **危险答案**：给模型 shell，再靠 Prompt 限制目录。

### Q59. 如何做 Agent 全链路可观测性？
- **30 秒**：一个 trace 关联 route、model/provider、prompt version、tool call、policy/approval、context/usage、sandbox、terminal reason 和业务结果。
- **2 分钟/深挖**：结构化事件、seq、span、成本、P95、错误类、路由版本；脱敏与日志权限；离线 replay 和线上告警。
- **评分点**：业务、模型、工具、安全四类指标。
- **危险答案**：保存完整 Prompt 和 stdout 就够。

### Q60. Agent 质量下降时如何系统排障？
- **30 秒**：先按版本和 trace 复现，再分 Router、Context/Retrieval、Provider、Tool、Policy、Loop/终止，不先盲调 Prompt。
- **2 分钟/深挖**：对比输入、候选、选择、执行、观察每步；检查数据漂移、检索、校准、Usage、错误和 fallback；最小实验定位责任层。
- **评分点**：分层归因与证据。
- **危险答案**：换更大模型或提高 temperature 试试。

### Q61. 如何控制 Agent 成本与延迟？
- **30 秒**：预算、候选工具缩减、检索去重、缓存、短路、并行无依赖任务、模型/Provider 路由和压缩。
- **2 分钟/深挖**：质量约束下优化，记录每 step/provider/tool token 与延迟；fallback 与缓存注意一致性；P50/P95/P99 和成本分位。
- **评分点**：不以牺牲安全和答案质量换低成本。
- **危险答案**：所有任务切最便宜模型。

### Q62. 如何做 Agent 评测体系？
- **30 秒**：任务成功、步骤/工具正确、事实与证据、成本延迟、鲁棒性、安全违规和恢复能力多维评测。
- **2 分钟/深挖**：golden tasks、模拟工具、故障注入、对抗样本、路由混淆矩阵、压缩探针、权限负测；线上 shadow/canary 与人工复核。
- **评分点**：组件指标和端到端指标并存。
- **危险答案**：只让另一个 LLM 打分。

### Q63. 如果面试官问项目中没实现的能力，怎么答？
- **30 秒**：先明确冻结版本未实现，再给最小改造路径、关键风险和验证标准，不把建议包装成经历。
- **2 分钟/深挖**：例：agentHub Codex Provider；沿 AbstractProvider/Factory/Runtime 事件接入，补会话、工具、权限、取消、Usage 和 contract tests。
- **评分点**：诚实、技术完整、可执行。
- **危险答案**：用框架或依赖存在来暗示已经做过。

### Q64. 用 5 分钟总结 Phase 1 的 Agent 工程观。
- **30 秒**：ESGQA 是把特定 ESG/CBAM 业务步骤固化为状态图的领域编排；agentHub `AgentRuntime` 是承载 Provider、会话、事件、队列、权限和压缩的通用运行时控制层。
- **2 分钟/深挖**：ESGQA 的控制单元是 `CBAMState`、业务节点和条件边，回答“该业务下一步去哪”；agentHub 的控制单元是会话、Provider 和统一事件，回答“Agent 生命周期如何被托管”。两者可在设计上组合，但冻结代码没有集成证据，也不能比较成谁“更 Agent”。详见 [[interview-answers/01-agent-runtime-and-architecture#Q16. ESGQA 图与 agentHub Runtime Loop 的根本差异？|Q16 详细答案]]。
- **评分点**：准确比较两个抽象层，并分别引用 `P1-ESG-STATEGRAPH-001` 与 `P1-HUB-RUNTIME-001`。
- **危险答案**：只罗列框架名、论文名和模型名。

## 10. 通用 Agent 架构专项题

> [!note] 项目边界
> Q65-Q84 考查通用架构，不要求两个项目已经采用。回答先讲标准机制，再用 [[general-agent-architecture-patterns]] 和 [[project-evidence-map]] 说明项目映射。

### Q65. ReAct 的完整循环是什么，为什么不等于 while-loop？
- **30 秒**：ReAct 让模型在决策/推理、Action 和 Observation 之间交错，根据最新环境反馈修正下一步；while-loop 只是控制结构。
- **2 分钟/深挖**：画 `decide -> validate -> authorize -> execute -> observe -> update -> terminate`；模型只产生候选动作，Runtime 控制 Schema、Policy、HITL、预算和终止。说明结构化 reason code 可观测，但不应把完整私有推理当稳定 API。
- **评分点**：Observation 真正影响后续决策；能区分策略、控制流与 Runtime；引用 `SRC-REACT-001`。
- **危险答案**：只要 Prompt 输出 Thought/Action，就是生产 ReAct；`while True` 会自动形成可靠 Agent。

### Q66. 如何把论文式 ReAct 改造成生产 Runtime？
- **30 秒**：使用 typed decision、Tool Registry、参数验证、actor-bound Policy、有界预算、幂等、结构化 Observation、进展检测和 terminal reason。
- **2 分钟/深挖**：补 Provider/Tool timeout、重试分类、pending approval、checkpoint、Context 压缩、Prompt Injection、防重复副作用和逐步 trace。高风险动作不能因模型 confidence 高而跳过策略。
- **评分点**：至少覆盖控制、失败、安全、可观测和成本五面。
- **危险答案**：调低 temperature，再设 max_steps 就能上线。

### Q67. CoT、Function Calling 与 ReAct 有何区别？
- **30 秒**：CoT 是推理轨迹；Function Calling 是结构化动作请求机制；ReAct 是推理/决策和环境动作/观察交错的策略模式。
- **2 分钟/深挖**：ReAct 可通过 Function Calling 实现 Action，也可用文本动作协议；Function Calling 本身不规定是否多步、不提供权限；CoT 没有环境反馈闭环。
- **评分点**：层次清楚，能说明三者如何组合。
- **危险答案**：开启 tool calling 就自动采用 ReAct；有 CoT 就有 Agent。

### Q68. 什么任务不应该使用 ReAct？
- **30 秒**：步骤可枚举、错误代价高、延迟上界严格或副作用不可逆的任务，优先确定性 Workflow。
- **2 分钟/深挖**：举支付审批、权限变更、固定 RAG 流程；开放检索可局部使用有界 ReAct，但入口认证、工具权限、结果验证和真实提交保持确定性。
- **评分点**：能从动态性、风险、成本和审计选型，而非追架构热点。
- **危险答案**：ReAct 是最通用架构，所有 Agent 都应使用。

### Q69. Plan-and-Execute 中的计划漂移如何处理？
- **30 秒**：每步比较 expected output 与 Observation；前提失效时只重规划未完成子图，保留已验证产物并限制 replan 次数。
- **2 分钟/深挖**：Plan 需 version、assumption、dependency、acceptance；区分局部修参、剩余计划重构、目标变化确认和无进展终止；重规划前重新做能力、权限和预算校验。
- **评分点**：不丢成功工作、有 replan trigger 和 termination。
- **危险答案**：遇到任何错误都让 Planner 从头生成完整计划。

### Q70. Plan-and-Execute 与 ReWOO 的本质差异？
- **30 秒**：二者都先计划；ReWOO 突出用变量引用未来工具结果，由 Planner、Worker、Solver 解耦，减少每个 Observation 后重复推理。
- **2 分钟/深挖**：Plan-and-Execute 更泛化，可每步重规划；ReWOO 预先变量化，适合依赖清晰和并行，但中间适应性弱。生产实现把变量编译成 typed artifact DAG。
- **评分点**：变量依赖、三角色和适应性权衡；引用 `SRC-REWOO-001`。
- **危险答案**：任何有 Planner 和 Executor 的系统都是 ReWOO。

### Q71. Reflexion 与 Self-Refine 有何区别？
- **30 秒**：Reflexion 把任务反馈转成语言反思并写入 episodic memory，影响后续 trial；Self-Refine 对同一输出进行 feedback -> revision。
- **2 分钟/深挖**：比较反馈时间尺度、记忆、验收和成本；Reflexion 风险是错误经验持久化，Self-Refine 风险是同模型自评偏差。两者都不等于参数训练。
- **评分点**：跨 trial vs 输出内多轮；`SRC-REFLEXION-001/SRC-SELF-REFINE-001`。
- **危险答案**：二者都是让模型再想一次，所以完全相同。

### Q72. CRITIC 为什么强调外部工具反馈？
- **30 秒**：模型可用搜索、解释器或测试获得可验证证据，再据此批评和修订，减少纯内在自评缺少新信息的问题。
- **2 分钟/深挖**：举代码测试和事实核验；工具结果仍可能错误或被注入，需来源、权限和结构化 Result；修订前后用外部 verifier 选择，不默认最后一版最好。
- **评分点**：外部反馈的价值和新的 Tool 安全面；`SRC-CRITIC-001`。
- **危险答案**：只要调用任何工具，模型的最终修订就一定正确。

### Q73. 为什么自我反思可能让答案更差？
- **30 秒**：若模型没有新证据，generator 与 critic 共享知识缺口和偏差，可能把正确答案改坏或强化错误。
- **2 分钟/深挖**：引入独立 rubric、外部 verifier、测试/检索/人类反馈、最大轮次、improvement threshold 和保留 best-so-far；记录 draft/feedback/revision。
- **评分点**：引用 `SRC-SELF-CORRECTION-LIMIT-001`，不极端化为反思永远无效。
- **危险答案**：模型生成一段 critique 就等于获得新知识。

### Q74. Tree of Thoughts 的四个核心组件是什么？
- **30 秒**：thought state、候选生成器、评价器和搜索/剪枝策略，外加最终 verifier。
- **2 分钟/深挖**：说明 BFS/DFS/beam、branching/depth/token 预算、回溯、等价状态缓存；thought 必须是可评估中间单元，不只是随意切句。
- **评分点**：搜索状态、生成、评价和终止完整；`SRC-TOT-001`。
- **危险答案**：一次生成多个答案再多数投票就是完整 ToT。

### Q75. ToT 最关键的薄弱点为什么是 evaluator？
- **30 秒**：搜索质量取决于能否给好路径更高分；偏差评价器会系统剪掉正确分支，更多搜索反而扩大错误。
- **2 分钟/深挖**：同模型 generator/evaluator 错误相关；可用外部 verifier、过程约束、校准、多评价器和 best-so-far。指标需分解 generation recall、ranking accuracy、search cost 和 task success。
- **评分点**：能解释搜索不是免费正确率提升。
- **危险答案**：增加分支数总能覆盖并找到正确答案。

### Q76. Graph of Thoughts 与 LangGraph/StateGraph 有何区别？
- **30 秒**：GoT 是 thought 的生成、依赖、聚合和反馈推理范式；StateGraph 是表达任意工作流状态和边的控制框架。
- **2 分钟/深挖**：GoT 节点是候选思想或部分结果，可合并/变换；StateGraph 节点可以是 API、规则或业务步骤。可以用 StateGraph 实现 GoT，但用了图框架不等于采用 GoT。
- **评分点**：范式与实现工具分层；引用 `SRC-GOT-001`。
- **危险答案**：ESGQA 使用 StateGraph，所以项目已经实现 Graph of Thoughts。

### Q77. LATS 如何把推理、行动和规划结合？
- **30 秒**：LATS 在树搜索中用 LM 生成动作、与环境交互、用 value/self-reflection 评估，并把价值反馈到祖先后继续选择。
- **2 分钟/深挖**：类比 MCTS 的 selection/expansion/evaluation/backpropagation；强调真实环境不可随意回滚，高风险动作应在模拟器或 sandbox 分支，最终只提交一次。
- **评分点**：环境反馈、value、reflection、树搜索和安全可回滚；`SRC-LATS-001`。
- **危险答案**：在真实数据库上并行尝试多条写路径，再选最好结果。

### Q78. DAG 并行执行需要满足什么条件？
- **30 秒**：无数据依赖、资源不冲突、副作用可交换/幂等、deadline 可传播、结果可确定性合并。
- **2 分钟/深挖**：ready set、并发上限、关键路径、fail-fast/best-effort/quorum、取消和补偿；理论延迟下界由关键路径决定，实际受限流与资源竞争影响。
- **评分点**：依赖与副作用同时考虑；`SRC-LLMCOMPILER-001`。
- **危险答案**：把所有工具调用放进 `Promise.all` 就是 LLMCompiler。

### Q79. Supervisor/Worker Multi-Agent 如何设计？
- **30 秒**：Supervisor 分解、分派和验收，Worker 按专业能力、工具或权限执行；共享 typed artifact 而非复制整段聊天。
- **2 分钟/深挖**：任务契约、owner、deadline、权限、返回 Schema、retry、worker health、合并和终止；Supervisor 是瓶颈和单点错误，可分层或用确定性 scheduler。
- **评分点**：角色真正对应能力/上下文差异，引用 `SRC-AUTOGEN-001/SRC-CAMEL-001` 但不外推性能。
- **危险答案**：给同一模型三个不同 persona 就自然形成可靠团队。

### Q80. 什么时候 Multi-Agent 值得，什么时候只是过度设计？
- **30 秒**：只有任务确需不同能力、权限、上下文隔离或并行责任时值得；能用函数、Tool 或单 Agent 子任务解决时不必增加 Agent。
- **2 分钟/深挖**：比较 Token 复制、通信延迟、错误转述、责任归因和安全面；先做单 Agent baseline，再以成功率/成本/延迟和可维护性证明增益。
- **评分点**：给出引入门槛和量化验证。
- **危险答案**：Agent 数量越多，推理能力越强。

### Q81. 如何为 ESGQA 设计不夸大的 Hybrid 演进？
- **30 秒**：保留现有 StateGraph 作为确定性外壳，只在开放证据缺口子图引入有界 ReAct；报告任务可用 Plan/DAG，最终仍做 RAG 证据和安全校验。
- **2 分钟/深挖**：列 Tool allowlist、步数/Token、检索来源、progress detector、失败出口和评测；明确当前冻结代码未实现这些通用策略。
- **评分点**：`P1-ESG-STATEGRAPH-001` 与未实现边界同时出现。
- **危险答案**：把现有条件图直接改名 ReAct/GoT，作为项目亮点。

### Q82. 如何在 agentHub 中增加可插拔 Agent 策略？
- **30 秒**：在现有事件 Runtime 上定义 typed `DecisionPolicy`，分别实现 bounded ReAct、Plan 或 Search；Runtime 继续负责 Provider、权限、Context、队列和终止。
- **2 分钟/深挖**：统一 decision/event/terminal 协议，Factory 注册策略，contract suite 验证动作、观察、取消、预算、HITL、压缩和迟到事件；具体论文算法需独立证据。
- **评分点**：复用 `P1-HUB-RUNTIME-001/P1-HUB-PLANNER-001`，建议使用将来时。
- **危险答案**：agentHub 有 Planner 和事件循环，所以已经同时实现 ReAct、ReWOO 和 Reflexion。

### Q83. 项目没用 ReAct，面试时怎样回答才不吃亏？
- **30 秒**：先完整讲 ReAct 机制和生产化，再解释项目为何选择显式 Workflow，并给出受限引入场景；诚实说明未实现。
- **2 分钟/深挖**：从业务可预测性、风险、审计、延迟选择 StateGraph；展示你理解 ReAct 的 typed action、Observation、循环终止和安全；提出局部 Hybrid 与验证计划。
- **评分点**：知识完整、架构判断和项目诚信同时成立。
- **危险答案**：项目没用所以没研究；或者把相似循环包装成 ReAct。

### Q84. 面对新任务，如何系统选择 Workflow、ReAct、Planning、Reflection、Search 或 Multi-Agent？
- **30 秒**：依次问步骤可枚举性、Observation 动态性、依赖并行、verifier、环境可回滚、主体是否真需分离，以及风险/预算上限。
- **2 分钟/深挖**：可枚举先 Workflow；动态探索局部 ReAct；长依赖任务 Plan/DAG；有可信反馈再 Reflection；可回溯且有 evaluator 才 Search；能力/权限/上下文确需分离才 Multi-Agent。生产常用确定性外壳加局部策略。
- **评分点**：从最简单基线开始，有升级条件、失败成本和 Hybrid。
- **危险答案**：脱离任务直接推荐当前最热门架构。

## 11. 最后自检

- 是否先给结论，再给机制、权衡、证据和边界？
- 是否能在 30 秒、2 分钟、5 分钟三档切换？
- 是否把源码路径证明与个人设计建议分开？
- 是否避免虚构性能数字、线上规模、公司原题和未实现能力？
- 是否能对每个 happy path 补一个 failure path、一个指标和一个安全风险？
- 是否能解释为什么这样设计，而不只会报 API 名称？

关联笔记：[[lessons/1.1-agent-definition-and-first-principles]]、[[lessons/1.2-agent-loop-patterns]]、[[lessons/1.3-tool-use-design]]、[[lessons/1.4-routing-and-decision]]、[[lessons/1.5-provider-abstraction]]、[[lessons/1.6-context-engineering]]、[[lessons/1.7-agent-security-boundaries]]。




