---
tags: [phase-1, interview, standardized-answers, agent-runtime, architecture]
created: 2026-07-22
updated: 2026-07-22
status: complete
question_range: Q1-Q16
---

# Q1-Q16：Agent Runtime 与编排架构标准答案

> [!tip] 口述方法
> 30 秒先下定义；2 分钟按“机制 -> 失败 -> 权衡 -> 项目证据”展开。面经来源只证明题目被公开报告过，技术答案以 [[technical-source-register]] 和 [[project-evidence-map]] 为准。

## Q1. 什么是 Agent？它与一次 LLM 调用有何区别？

- **公开面经映射**：`INT-NC-A03`、`INT-NC-B02`；直接/汇总面经都出现过 Agent 定义或相对直接模型调用的区别。
- **30 秒答案**：Agent 是围绕目标运行的受约束状态系统，Runtime 反复读取状态、让策略选择候选动作、校验授权、执行工具、接收环境观察并决定继续或终止。LLM 通常只是策略或生成组件；一次 LLM 调用没有天然的持久状态、外部行动、错误恢复和安全边界。
- **2 分钟标准答案**：我会从八个要素定义 Agent：目标、状态、观察、策略、动作、环境、终止和治理。用户输入进入 Runtime 后，模型可以提出调用工具或直接回答，但真实动作必须经过 Schema、权限、预算和风险校验；执行结果以结构化 Observation 写回状态，下一步再基于新状态决策。它与 Chatbot、Workflow 不是按“有没有调用模型”二分：单次模型调用可以嵌入 Workflow，固定流程也可以有 Tool；只有形成可观察、可行动、可终止的反馈闭环，才具有 Agent 运行时特征。生产上还必须有 Trace、幂等、取消、HITL 和失败原因，否则只是演示级循环。
- **深挖追问**：一次调用能否是 Agent？可以是一个 Agent step，但外层仍需承担状态与控制；LLM 能否自己授权？不能，授权属于可信 Runtime；为什么不以“自主性”单指标定义？自主性不可审计，工程上要拆成具体决策权。
- **项目映射**：ESGQA 用 `P1-ESG-STATEGRAPH-001` 证明领域状态闭环；agentHub 用 `P1-HUB-RUNTIME-001` 证明生命周期、事件、队列和权限。两者都不能外推成“模型拥有全部控制权”。
- **评分/危险答案**：高分必须讲状态、环境反馈和终止；“会 Function Calling 的聊天机器人就是 Agent”只够术语分。

## Q2. Workflow、Agent 和 Multi-Agent 如何区分？

- **公开面经映射**：`INT-NC-A05` 直接报告 Agent 与普通 Workflow；`INT-NC-B01/B02` 报告单/多 Agent 选型。
- **30 秒答案**：Workflow 的路径主要由代码预先定义；Agent 在受控动作集合中根据上下文动态决策；Multi-Agent 进一步引入多个独立角色、能力、权限或上下文域，以及通信和协调成本。
- **2 分钟标准答案**：我用四个维度判断。第一是控制流：固定分支和状态转换优先 Workflow，长尾语义决策才交给 Agent。第二是风险：支付、权限修改等高风险动作保持确定性流程，模型只能做建议或局部分类。第三是可验证性：Workflow 容易覆盖分支，Agent 需要评测集、Trace 和失败回放。第四是主体分离：只有能力、权限、上下文或并行责任确需分开时才上 Multi-Agent；把同一模型换几个 persona 并不会自动更可靠。生产系统通常是混合架构：确定性外壳管理入口、预算、权限和提交，局部 Agent 处理开放决策。
- **深挖追问**：节点多是否更 Agent？不是；工作流中有模型是否变 Agent？取决于控制权；Multi-Agent 如何证明有价值？与单 Agent baseline 比成功率、成本、延迟和可维护性。
- **项目映射**：ESGQA 是显式领域状态图，证据为 `P1-ESG-STATEGRAPH-001`；agentHub 更接近多 Provider/多 Agent 的运行时基础，但不能仅凭 Agent 数量声称效果更强。
- **评分/危险答案**：高分答案必须允许混合形态；“Workflow 不智能、Agent 全自主、Multi-Agent 最先进”是典型扣分点。

## Q3. 为什么说 Runtime 比 Prompt 更接近 Agent 的工程核心？

- **来源性质**：生成训练题；C 级题单反复出现 Prompt、Tool、Runtime 边界，但不归因到公司。
- **30 秒答案**：Prompt 只影响模型的候选输出，Runtime 才拥有状态、资源、权限、预算、取消、重试、队列和终止，因此可靠性边界在 Runtime。
- **2 分钟标准答案**：Prompt 可以描述“先检查权限”“最多重试三次”，但它不能原子地更新数据库、阻止越权、杀死超时进程，也不能判断超时请求是否已产生副作用。Runtime 要把模型输出解析成 typed decision，验证工具与参数，绑定 actor 和租户，传播 deadline，执行并记录 ToolResult，再根据错误类别决定重试、降级、审批或终止。即便模型完全不遵守 Prompt，Runtime 也必须守住不可突破的不变量。Prompt 是策略上下文的一部分，Runtime 才是可信控制面。
- **深挖追问**：Prompt 是否不重要？重要，但负责引导而非强制；为什么 `temperature=0` 仍不可靠？确定性不等于正确性；如何测试 Runtime？用 Provider/Tool contract tests、事件序列、迟到事件、取消与权限反例。
- **项目映射**：`P1-HUB-RUNTIME-001` 展示队列、Provider 生命周期、权限事件和 idle 回收；`P1-HUB-PERMISSION-001` 同时暴露默认配置与路径语义仍需加固。
- **评分/危险答案**：只谈“系统 Prompt 写得好”没有解释资源所有权，最多 2 分。

## Q4. Agent 状态应该包含什么，不应该包含什么？

- **来源性质**：生成训练题；与 `INT-NC-B02` 的状态机、记忆追问相关。
- **30 秒答案**：状态应包含完成任务必需的业务事实、执行进度、工具产物引用、预算、错误和终止信息；连接对象、密钥、原始大文件和不可信日志不应直接进入模型可见状态。
- **2 分钟标准答案**：我会拆成 durable state、working state 和 model-visible context。durable state 保存可恢复的任务、版本、审批和提交结果；working state 保存当前计划、步骤、错误和临时 artifact；model-visible context 是从前两者投影、裁剪和脱敏后的有限视图。每个字段要有 Schema、owner、更新规则和序列化策略，并区分追加、覆盖、集合合并等 reducer。敏感信息只保留引用，工具凭证由执行层注入。并发时使用版本号或 compare-and-swap，不能让多个节点随意改同一个 dict。
- **深挖追问**：为什么日志不全塞进状态？Token、污染和敏感泄露；状态和记忆区别？状态服务当前任务，记忆跨任务复用且需要写入策略；Schema 合法是否代表语义正确？不代表，还需业务不变量。
- **项目映射**：ESGQA `CBAMState` 由 `P1-ESG-STATEGRAPH-001` 支持；agentHub `ContextBus` 由 `P1-HUB-CONTEXT-001` 支持。两者均没有证据证明状态设计最优。
- **评分/危险答案**：高分要讲模型可见投影和敏感边界；“所有内容放一个全局字典方便共享”是危险答案。

## Q5. 如何定义 Agent 的完成与停止？

- **公开面经映射**：`INT-NC-B01` 报告过“如何判断信息足够”和“死循环怎么处理”。
- **30 秒答案**：完成是业务验收条件已满足；停止还包括拒绝、需要澄清、预算耗尽、deadline、人工中止、不可恢复错误和最大步数。模型说 `done` 只能是候选信号。
- **2 分钟标准答案**：先把成功条件写成可检查的 acceptance，例如证据覆盖、输出 Schema、必需工具成功和业务状态确认。每个循环结束都由 Runtime 评估：是否成功、是否仍有可执行动作、是否有进展、剩余预算是否足够、是否触发风险审批。停止后返回结构化 terminal reason、已完成步骤、未完成项、可恢复 checkpoint 和用户可采取的下一步。防循环不仅靠 max_steps，还检测重复动作、重复状态、同一错误和连续低进展。高风险任务即使生成了答案，也必须在真实提交状态确认后才能标成功。
- **深挖追问**：信息充分性如何量化？必需槽位、证据覆盖和 verifier；预算耗尽是否算失败？是安全终止，不应伪装成功；部分结果如何处理？标注不完整和 provenance。
- **项目映射**：ESGQA 显式 `END/fallback` 由 `P1-ESG-STATEGRAPH-001` 支持；agentHub Planner/Manager 的有限动作与 abort 由 `P1-HUB-PLANNER-001` 支持。
- **评分/危险答案**：高分需区分 success 与 safe stop；“直到模型满意”没有工程终止语义。

## Q6. 设计 Agent 时第一性原理的顺序是什么？

- **来源性质**：生成训练题，用于纠正框架先行。
- **30 秒答案**：先定义业务目标和验收，再识别环境、可用动作和风险；随后设计状态、反馈、失败恢复和观测，最后才选择模型、Prompt 与框架。
- **2 分钟标准答案**：我的顺序是：目标/non-goal -> 成功与失败标准 -> 资产和威胁 -> 确定性可完成的部分 -> 必须交给模型的语义不确定部分 -> Tool 契约与副作用 -> 状态和终止 -> 预算、权限、恢复和评测 -> Provider/框架。这样能回答“为什么需要 Agent”而不是把 LangGraph、MCP 或 ReAct 当需求。最小实现先做单 Agent 或 Workflow baseline，再用离线集和真实 Trace 证明增加 Planning、Reflection 或 Multi-Agent 的收益。
- **深挖追问**：什么情况下不需要 LLM？规则可枚举且风险高；non-goal 有什么价值？限制自治范围和测试面；为什么先定义验收？没有 verifier 就无法判断优化是否有效。
- **项目映射**：ESGQA 适合解释“业务图优先”；agentHub 适合解释“Runtime 能力底座”。两者是不同问题域，不能套同一架构结论。
- **评分/危险答案**：先报框架栈、后补业务理由，通常会被继续追问到失守。

## Q7. 用 ESGQA 证明你理解 Agent，但不能夸大什么？

- **来源性质**：项目定制题；回答必须以冻结提交为准。
- **30 秒答案**：ESGQA 在冻结提交中实现了 `CBAMState`、显式节点、条件边、结构化 LLM 输出、记忆和检索，可证明领域状态化编排；不能说它已实现通用 ReAct、Tool Registry、多 Provider 或命令沙箱。
- **2 分钟标准答案**：我会先讲源码闭环：`build_cbam_graph()` 从 memory、intent 进入固定业务分支，各节点更新 `CBAMState`，错误优先进入 fallback，最终到 `END`；关键模型结果通过 Pydantic 结构化。随后主动说明边界：外部能力嵌在特定节点，没有统一 ToolSpec/Registry；多处直接构造 `ChatOllama`，没有 Provider Factory；API 层 JWT、限流和 Celery 不等于工具执行沙箱。改进时才使用将来时：可在确定性图外壳内局部引入有界 ReAct、统一 Tool 协议和 Provider adapter。
- **深挖追问**：状态图等于 Agent 吗？不自动等于；混合检索效果有数字吗？当前只有源码证据，没有运行指标；项目最大风险？未知分类默认路由、记忆污染、危险反序列化与能力安全缺口。
- **项目映射**：`P1-ESG-STATEGRAPH-001`、`P1-ESG-STRUCTURED-OUTPUT-001`、`P1-ESG-MEMORY-001`、`P1-ESG-RETRIEVAL-001`；缺口使用三个 `P1-ESG-GAP-*` 证据卡。
- **评分/危险答案**：能主动讲未实现能力比包装成“企业级全自治 Agent”更可信。

## Q8. 用 agentHub 证明 Runtime 能力，并说明风险。

- **来源性质**：项目定制题；回答以归档提交 `815f645` 为准。
- **30 秒答案**：agentHub 的 `AgentRuntime` 管 Provider 创建与切换、busy queue、权限事件、压缩和 idle 生命周期，能证明运行时编排；风险在 Provider 语义差异、默认权限、路径语义、容器加固和迟到事件。
- **2 分钟标准答案**：调用链是 prompt 进入 Runtime，按配置创建 Provider；忙时排队，空闲时发送；Provider 输出统一事件，Runtime 处理工具、权限、文本、usage 和完成；Usage 超阈值触发压缩状态机，终态后发送下一条或进入 idle timer。这个证据说明 Runtime 对生命周期和控制面负责，但底层模型的推理/工具循环部分仍由 Provider SDK 承担。安全上不能只说“有 Docker 和权限配置”：自定义 Agent 默认较宽，glob 不是路径 canonicalization，威胁模型还记录 seccomp/AppArmor、只读根文件系统和 Docker socket 等缺口。
- **深挖追问**：事件是否恰好一次？没有证据；Provider 接口是否完全抹平差异？不会；压缩阈值 70% 是否最佳？只是当前硬阈值，缺离线质量证明。
- **项目映射**：`P1-HUB-RUNTIME-001`、`P1-HUB-PROVIDER-001`、`P1-HUB-COMPRESSION-001`、`P1-HUB-SANDBOX-001`、`P1-HUB-PERMISSION-001`。
- **评分/危险答案**：高分答案同时讲能力与残余风险；“EventEmitter 自动保证顺序和 exactly-once”错误。

## Q9. 写出生产级 Agent Loop 的基本状态机。

- **公开面经映射**：`INT-NC-B01/B03` 报告循环流程和 Tool 调用流程。
- **30 秒答案**：`observe -> decide/plan -> validate -> authorize -> act -> observe -> update -> verify -> terminate/retry`，每个转换都有 deadline、错误类型、Trace 和终止原因。
- **2 分钟标准答案**：输入先转换成可信状态和不可信上下文；策略层产生 typed candidate action；Runtime 验证工具存在、参数、业务不变量、预算和权限；执行层用超时与幂等键调用工具；ToolResult 归一化为 success/error/unknown outcome Observation；状态更新后 verifier 判断验收和进展，再决定完成、澄清、审批、重试、重规划或失败。取消和 deadline 要向下传播，迟到结果不能覆盖已终止状态。关键是模型只提议，Runtime 决定是否执行。
- **深挖追问**：为什么 validation 和 authorization 分开？输入正确不代表 actor 有权；unknown outcome 怎么办？查询真实状态，不能盲重试；Loop 如何恢复？持久 checkpoint、step id 和幂等提交。
- **项目映射**：ESGQA 是显式领域图版本；agentHub 是事件驱动 Runtime 版本。二者都不是简单 `while True`。
- **评分/危险答案**：缺 validate/authorize/terminate 的“Thought-Action-Observation”只够论文概念分。

## Q10. ReAct、Plan-and-Execute、ReWOO 如何选择？

- **公开面经映射**：`INT-NC-A01` 报告 ReAct；`INT-NC-B03` 报告 Plan 模式。
- **30 秒答案**：环境反馈频繁、路径难预知时用有界 ReAct；任务依赖较清晰、需要全局分解时用 Plan-and-Execute；依赖可提前变量化、希望减少每步重新推理时考虑 ReWOO/DAG。
- **2 分钟标准答案**：ReAct 每步依据 Observation 调整，适应性强，但模型调用多、易循环；Plan-and-Execute 先拆目标，再逐步执行，可审计、可并行，但前提变化会造成计划漂移；ReWOO 用变量引用未来工具结果，把 Planner、Worker、Solver 解耦，适合依赖清晰的任务，但对中途变化较不敏感。生产中常混合：确定性外壳管理权限和提交，Planner 生成 typed DAG，局部不确定节点用短步 ReAct，失败时只重规划未完成子图。
- **深挖追问**：选择指标？成功率、模型调用、关键路径延迟、重规划率；ReWOO 是否等于所有 Planner/Executor？不是；能否在支付上 ReAct？只做建议，真实提交仍固定流程。
- **项目映射**：两个项目没有证据证明完整采用这些论文范式；ESGQA 可提出局部 Hybrid 建议，agentHub 可提出可插拔 DecisionPolicy，必须用将来时。
- **评分/危险答案**：只背论文名不讲环境动态性、依赖和失败恢复，得分有限。

## Q11. 为什么 Agent Loop 必须有 step budget、token budget 和 deadline？

- **公开面经映射**：`INT-NC-B01` 的死循环追问；`INT-NC-A01` 的工具调用限制。
- **30 秒答案**：三者分别限制动作次数、模型消耗和墙钟时间，防止无进展循环、成本失控和用户超时；它们不可互相替代。
- **2 分钟标准答案**：step budget 防反复调用同一工具；token budget 控制上下文和生成成本；deadline 控制真实 SLA，并且要以绝对时间向 Provider、Tool 和子任务传播。每步前先判断剩余预算是否足以完成动作，执行后记录消耗和 progress。达到阈值时返回结构化部分结果和 terminal reason，而不是假装成功。对长任务还可按计划阶段分配子预算，并保留全局硬上限。
- **深挖追问**：为什么只设 retry=3 不够？可能在不同动作间循环；超时后结果回来怎么办？按 execution id 丢弃或只用于审计；Token 预算如何预测？历史 token + 预留输出 + tool schema + 安全余量。
- **项目映射**：agentHub 有 Usage 压缩入口，但没有证据说明存在统一三预算体系；回答应区分代码现状和理想设计。
- **评分/危险答案**：高分需讲向下传播与终止语义；“超时就再试一次”可能重复副作用。

## Q12. Loop 中工具失败如何处理？

- **公开面经映射**：`INT-NC-B02` 报告 Agent 失败/中断与重试安全；`INT-NC-A02` 的工具问题与 Q89 互补。
- **30 秒答案**：先分类为参数、权限、限流/瞬时、业务拒绝、不可恢复和未知结果；只有明确可重试且副作用安全的错误才有界重试。
- **2 分钟标准答案**：参数错误返回结构化 validation issues，让策略修正一次；权限错误终止或进入审批，不能换工具绕过；限流按 `retry-after` 和剩余 deadline 退避；业务拒绝作为正常结果交给上层；不可恢复错误失败；写操作超时属于 unknown outcome，先用幂等键或查询接口核验真实状态。ToolResult 至少包含 status、error code、retryable、side-effect state、artifact references 和 trace id。重试要有上限、抖动、熔断和补偿策略。
- **深挖追问**：工具返回 200 但语义错误？业务 verifier；模型能否决定 retryable？不能单独决定；多个失败如何聚合？保留根因和每步结果，避免只剩最后异常。
- **项目映射**：`P1-HUB-RUNTIME-001` 有失败清理与队列处理；`P1-HUB-PLANNER-001` 有失败审查；不能据此声称所有副作用均幂等。
- **评分/危险答案**：`catch Exception -> 发回模型 -> 无限修` 是典型不合格实现。

## Q13. 如何并行执行 Agent 子任务？

- **来源性质**：生成训练题；与 `INT-NC-B01` 的 Multi-Agent 并发追问相关。
- **30 秒答案**：先构建依赖 DAG，只并行无数据依赖、无资源冲突且副作用可交换或幂等的节点，再按确定性 reducer 合并。
- **2 分钟标准答案**：Planner 输出任务、输入 artifact、依赖、资源、deadline 和验收；调度器维护 ready set 和并发上限。父 deadline 和取消信号传给子任务；失败策略按业务选择 fail-fast、best-effort 或 quorum。结果合并必须基于 typed artifact、版本和确定性 reducer，不能把多段聊天直接拼接后期望模型消歧。关键路径决定理论延迟下界，过度并行会触发 Provider 限流、数据库锁竞争和 Token 复制。
- **深挖追问**：两个只读查询能否并行？通常可以，但要考虑共享限流；两个写操作呢？需事务、幂等或串行化；一个失败是否取消全部？取决于依赖和部分结果价值。
- **项目映射**：`SRC-LLMCOMPILER-001` 只支持架构思想；两个项目没有任意 DAG 编译器的已实现证据。
- **评分/危险答案**：`Promise.all` 不是并行设计的完整答案。

## Q14. 状态图相比隐式 Loop 有何优势？

- **公开面经映射**：`INT-NC-B02` 报告状态机能力；`INT-NC-B02` 的 LangGraph 追问延伸到 Q94。
- **30 秒答案**：状态图把节点、边、状态转换、循环和终止显式化，更容易可视化、单测、检查点恢复、人工介入和审计。
- **2 分钟标准答案**：隐式 Loop 常把策略、状态更新、错误和工具调用揉在一个函数里。状态图能为每个节点定义输入输出 Schema，用条件边表达合法转换，用 reducer 管并发更新，用 checkpoint 恢复，并检查不可达节点或无终止环。代价是图和状态版本迁移、节点粒度选择及框架耦合；图框架本身不保证业务正确、权限安全或恢复语义，仍要显式实现。
- **深挖追问**：图越细越好吗？不，过细增加状态噪声；LangGraph 是否等于 Graph of Thoughts？不是；循环边怎么安全？上限、进展检测和终态原因。
- **项目映射**：ESGQA `P1-ESG-STATEGRAPH-001` 是直接源码证据；不能因使用 StateGraph 就声称 checkpoint、HITL 等框架能力均已配置。
- **评分/危险答案**：“用了 LangGraph 所以天然可靠”会被追问到实现证据。

## Q15. Planner 输出错误怎么办？

- **公开面经映射**：`INT-NC-B03` 的 Plan 模式；与真实项目计划验证追问高度相关。
- **30 秒答案**：先做 Schema 和业务不变量校验，有界修复；持续失败就澄清、降级、人工审查或终止，不能直接执行自然语言计划。
- **2 分钟标准答案**：校验至少包括：工具是否存在、参数来源是否可得、依赖是否无环、输出是否被下游消费、权限是否允许、预算/deadline 是否可行、写操作是否有幂等/补偿。验证错误以结构化列表反馈给 Planner，限制修复轮数，并保留原计划与每次差异。执行中如果前提失效，只重规划未完成子图，已验证产物保持不可变。解析或调用异常默认保守终止，不让模型用任意文本绕过动作集合。
- **深挖追问**：另一个 LLM 审核够吗？不够，确定性不变量优先；DAG 有环怎么处理？拒绝或要求重写；计划最优如何证明？通常不证明最优，只验证可行并比较指标。
- **项目映射**：`P1-HUB-PLANNER-001` 有 JSON 校验、有限重试和 Manager 的 continue/replan/abort；不能说 Planner 一定正确。
- **评分/危险答案**：高分要包含能力、权限和副作用校验；“让 Planner 再想一次”不完整。

## Q16. ESGQA 图与 agentHub Runtime Loop 的根本差异？

- **来源性质**：项目对比题；此处修正原主库中答案与题目错位的问题。
- **30 秒答案**：ESGQA 是把特定 ESG/CBAM 业务步骤固化为状态图的领域编排；agentHub `AgentRuntime` 是承载不同 Provider、会话、事件、队列、权限和压缩的通用运行时控制层。前者回答“该业务下一步去哪”，后者回答“Agent 进程和交互如何被可靠托管”。
- **2 分钟标准答案**：ESGQA 的主要控制单元是 `CBAMState`、业务节点和条件边，流程路径在图中显式定义，适合审计固定领域 Pipeline；外部检索、计算和模型调用嵌在节点内。agentHub 的主要控制单元是会话、Provider 实例和统一事件，Runtime 负责 busy queue、权限请求、Usage 压缩、终态和 idle 回收，底层推理/Tool 细节可由 Provider SDK 承担。两者可以组合：领域图可运行在 Runtime 之上，Runtime 也可托管不同编排策略；但当前冻结代码没有证据表明两项目已集成。比较时不能说一个“更 Agent”，而要看它们处在不同抽象层。
- **深挖追问**：谁更适合恢复？取决于 checkpoint 和事件持久化实现，不能从框架名推断；谁负责权限？ESGQA 只有 API 入口安全证据，agentHub 有角色权限但仍有缺口；能否互借实现？只能提出设计，不能改写为当前事实。
- **项目映射**：ESGQA：`P1-ESG-STATEGRAPH-001/P1-ESG-ROUTING-001`；agentHub：`P1-HUB-RUNTIME-001/P1-HUB-PROVIDER-001/P1-HUB-COMPRESSION-001`。
- **评分/危险答案**：若回答成通用“Agent 六层架构”而不比较两个项目，就是答非所问；这正是原卡需要修正的地方。


