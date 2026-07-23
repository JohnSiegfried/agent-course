---
tags: [phase-1, interview, standardized-answers, react, planning, reflection, search, multi-agent]
created: 2026-07-23
updated: 2026-07-23
status: complete
question_range: Q65-Q84
---

# Q65-Q84：通用 Agent 架构模式标准答案

> [!important] 项目边界
> 这些题考通用知识。两个项目没有使用某论文范式时，先讲标准机制，再用“现状/缺口/演进”映射，绝不能把相似循环重命名为已实现的 ReAct、ReWOO、Reflexion 或 LATS。

## Q65. ReAct 的完整循环是什么，为什么不等于 while-loop？

- **来源映射**：`INT-NC-A01` 直接报告字节 Agent 面试询问 ReAct 架构与原理。
- **30 秒答案**：ReAct 让模型在决策/推理、Action 和 Observation 间交错，用真实环境反馈修正下一步；while-loop 只是一种控制结构，没有规定动作语义、反馈利用和终止。
- **2 分钟标准答案**：生产表达为 `observe -> decide -> validate -> authorize -> execute -> observe -> update -> verify/terminate`。模型给 typed candidate action，Runtime 检查 Tool、参数、权限、预算和风险；ToolResult 作为结构化 Observation 进入下一轮。ReAct 的价值在于中间观察能改变路径，代价是调用多、状态漂移和循环。公开论文机制不自动包含幂等、HITL、取消、审计和 Prompt Injection 防御，这些必须由 Runtime 补齐。
- **深挖**：必须输出 Thought 吗？不必，不应依赖私有 CoT；一次 Tool call 算 ReAct 吗？未必，需后续观察影响决策；终止谁决定？Runtime 验收优先。
- **项目映射**：ESGQA 的固定图不是通用 ReAct；agentHub 的事件循环也不能直接命名为 ReAct。
- **评分/危险**：`while True` 加 Thought/Action 文本不是生产 ReAct。

## Q66. 如何把论文式 ReAct 改造成生产 Runtime？

- **来源映射**：`INT-NC-A01` 的 ReAct 与工具限制追问。
- **30 秒答案**：把自由文本动作改为 typed decision，并补 Tool Registry、参数/权限、预算、幂等、结构化 Observation、进展检测、checkpoint、Trace 和终态。
- **2 分钟标准答案**：模型层只负责在最小工具集合中提议；控制层验证 Schema、业务不变量、actor-bound authz 和高风险审批；执行层带 deadline、幂等键和资源限制；观察层归一化错误、来源和副作用状态；状态层做版本、恢复和上下文预算；终止层检查验收、无进展和硬预算。安全上把外部内容标记为不可信数据，权限不可由 Prompt 改写。评测按 Tool 选择/参数、循环长度、任务成功、成本和安全分解。
- **深挖**：max_steps 是否够？不够，还要重复状态和 deadline；模型修参几次？按错误类型有界；高风险 Tool 怎么办？固定 Workflow 或 HITL。
- **项目映射**：只能作为 ESGQA/agentHub 的演进建议。
- **评分/危险**：降 temperature 不能补齐运行时控制。

## Q67. CoT、Function Calling 与 ReAct 有何区别？

- **来源映射**：`INT-CS-C01` 趋势题；Function Calling 在 A/B 面经中反复出现。
- **30 秒答案**：CoT 是推理轨迹或提示策略；Function Calling 是结构化表达动作请求的接口；ReAct 是决策、行动和环境观察交错的多步策略范式。
- **2 分钟标准答案**：CoT 可以完全不接触环境；Function Calling 可以只调用一次，也不自带权限、执行和循环；ReAct 可以用 Function Calling 表达 Action，也可用其他 typed protocol。三者可组合：模型内部推理后输出 function call，Runtime 执行并回注 Observation，形成 ReAct；但生产系统不应依赖展示完整私有 CoT 来审计，而应记录 decision reason、Tool 和结果。
- **深挖**：JSON Mode 等于 Function Calling 吗？不是，前者只约束输出结构；开启 Tool Calling 就是 Agent 吗？不是；如何可观测推理？记录可公开的理由码和状态转换。
- **项目映射**：ESGQA 有 Pydantic 结构化输出，但没有通用 Function Calling/ReAct 证据。
- **评分/危险**：把三者都解释成“让模型一步步想”会失分。

## Q68. 什么任务不应该使用 ReAct？

- **来源映射**：面经中的 Agent/Workflow 和单/多 Agent 选型。
- **30 秒答案**：步骤可枚举、错误代价高、延迟上界严格、合规路径固定或副作用不可逆的任务，优先确定性 Workflow；ReAct 只用于局部开放探索。
- **2 分钟标准答案**：支付提交、权限变更、数据删除和监管报送应由状态机、事务和审批控制；模型可做意图识别、材料检查和建议，但不能动态改变真实提交路径。纯格式转换、固定 ETL 和简单 CRUD 也不值得承担多步模型成本。开放搜索、诊断和信息收集可使用有界 ReAct，并在真实副作用前回到确定性门。
- **深挖**：知识问答需要 ReAct 吗？固定 RAG 通常 Workflow 足够；什么情况下升级？问题需要动态选择多数据源/澄清/工具；如何证明？与固定 baseline 比。
- **项目映射**：ESGQA 选择显式 StateGraph 是合理领域判断，不是“技术落后”。
- **评分/危险**：“ReAct 最通用，所以所有 Agent 都用”缺少风险意识。

## Q69. Plan-and-Execute 中的计划漂移如何处理？

- **来源映射**：`INT-NC-B03` 报告 Plan 模式；`INT-NC-B02` 报告失败重试。
- **30 秒答案**：每步用 expected output/assumption 对比 Observation；前提失效时只重规划未完成子图，保留已验证产物，并限制 replan 次数和预算。
- **2 分钟标准答案**：计划节点包含 dependency、input artifact、acceptance、assumption、owner 和 deadline。执行后检查产物是否满足契约、外部状态是否变化。轻微偏差可修参数，局部依赖变化重构剩余 DAG，目标变化需用户确认；连续无进展则终止。Plan 版本化，已成功且仍有效的 artifact 不重复执行，写操作尤其不能从头重放。
- **深挖**：何时全量重规划？目标或核心约束变化且旧产物不可复用；如何防重规划循环？次数、成本和 progress threshold；谁判断 assumption 失效？确定性 verifier 优先。
- **项目映射**：agentHub Manager 有 continue/replan/abort，证据 `P1-HUB-PLANNER-001`，不证明完整 DAG 漂移治理。
- **评分/危险**：每次错误都从头生成计划会重复副作用和浪费成本。

## Q70. Plan-and-Execute 与 ReWOO 的本质差异？

- **来源性质**：通用架构题，答案依据 `SRC-REWOO-001`。
- **30 秒答案**：Plan-and-Execute 是泛化的先计划后执行；ReWOO 强调用变量引用未来工具结果，把 Planner、Worker、Solver 解耦，减少每个 Observation 后重复调用 Planner。
- **2 分钟标准答案**：ReWOO 计划形如工具步骤和 `#E1/#E2` 依赖，Worker 执行并填变量，Solver 汇总。优点是依赖显式、可并行、模型调用少；缺点是计划阶段看不到真实中间反馈，动态环境适应性弱。生产上应把变量编译为 typed artifact DAG，验证无环、参数来源、权限和失败策略；必要时在失败节点局部重规划。
- **深挖**：任意 Planner/Executor 都是 ReWOO 吗？不是，变量化和角色解耦是关键；中途信息改变怎么办？局部 replan；为何不总用？计划错误会批量传播。
- **项目映射**：两个项目均无完整 ReWOO 证据。
- **评分/危险**：只说“ReWOO 更省 Token”不够。

## Q71. Reflexion 与 Self-Refine 有何区别？

- **来源性质**：通用论文题，依据 `SRC-REFLEXION-001/SRC-SELF-REFINE-001`。
- **30 秒答案**：Reflexion 把任务反馈转成语言反思并存入 episodic memory，影响后续 trial；Self-Refine 在同一输出上循环 feedback -> revision。
- **2 分钟标准答案**：Reflexion 的时间尺度跨尝试，需要环境反馈和记忆检索，适合可重复 trial；Self-Refine 聚焦单次产物，通过评价和修订改善输出。两者都不更新模型参数。风险分别是错误经验长期污染，以及 generator/critic 同源导致把正确答案改坏。生产中要有外部 verifier、最大轮次、best-so-far 和写入门槛。
- **深挖**：与 RL 有何区别？无参数优化；如何选择记忆？按任务相似、可信反馈和 TTL；Self-Refine 何时停止？改进阈值、验收或预算。
- **项目映射**：项目 Memory/Planner 不能自动命名为 Reflexion。
- **评分/危险**：“都是让模型再想一次”没有时间尺度和记忆差异。

## Q72. CRITIC 为什么强调外部工具反馈？

- **来源性质**：通用论文题，依据 `SRC-CRITIC-001`。
- **30 秒答案**：外部搜索、解释器或测试能提供新且可验证的信息，打破纯自评共享知识缺口的问题。
- **2 分钟标准答案**：模型先生成，再选择工具核验事实、计算或代码，通过结果形成 critique 并修订。价值不在“多一轮模型”，而在引入外部证据。ToolResult 仍可能错误、过期或被注入，因此要有来源、权限、Schema 和 verifier；修订前后比较，保留 best-so-far，不默认最后一版最好。
- **深挖**：搜索结果就是真相吗？不是；代码任务最可靠反馈？测试、类型检查和静态分析组合；工具不可用？降级并标注未验证。
- **项目映射**：ESGQA 的检索/评估节点可类比外部反馈，但不能称已实现 CRITIC。
- **评分/危险**：“调用工具后一定正确”错误。

## Q73. 为什么自我反思可能让答案更差？

- **来源性质**：通用研究边界题，依据 `SRC-SELF-CORRECTION-LIMIT-001`。
- **30 秒答案**：没有新证据时，生成器和批评器共享知识缺口与偏差，可能把正确答案改坏、放大幻觉或产生迎合式修改。
- **2 分钟标准答案**：改进依赖反馈质量而非轮数。控制方法是明确 rubric、独立/异构 verifier、测试或检索证据、保留原稿和 best-so-far、最大轮次和 improvement threshold。评测要比较首次、最佳和最终版本，观察 regression rate，而不是只展示成功案例。高风险事实不能靠模型自信度决定。
- **深挖**：不同模型互评就独立吗？仍可能共享训练偏差；多数投票可靠么？错误可相关；何时值得反思？存在可验证反馈且修订成本可接受。
- **项目映射**：两个项目无自反思效果证据。
- **评分/危险**：“让模型检查一次就更可靠”缺乏证据。

## Q74. Tree of Thoughts 的四个核心组件是什么？

- **来源性质**：通用论文题，依据 `SRC-TOT-001`。
- **30 秒答案**：thought state、候选生成器、评价器、搜索/剪枝策略，生产上还要最终 verifier 和预算。
- **2 分钟标准答案**：把问题表示为可评估的中间状态；生成器扩展多个候选；评价器估计可行性/价值；BFS、DFS 或 beam 选择继续和回溯。需要 branching/depth/token 上限、状态去重和缓存，最终用外部 verifier 验收。它适合可分解、有中间评价且可回溯的任务，不适合真实不可逆动作直接分支试错。
- **深挖**：多采样就是 ToT 吗？没有状态搜索和剪枝不算；thought 粒度如何定？能独立评价且支持扩展；何时停止？找到通过 verifier 的解或预算耗尽。
- **项目映射**：StateGraph 不等于 ToT。
- **评分/危险**：只答“生成多个答案投票”不完整。

## Q75. ToT 最关键的薄弱点为什么是 evaluator？

- **来源性质**：通用架构深挖题。
- **30 秒答案**：搜索质量由 evaluator 的排序决定；若它系统偏向错误路径，更多分支会更快剪掉正确解并扩大成本。
- **2 分钟标准答案**：评价器需要区分“看起来合理”和“真实接近目标”。同模型 generator/evaluator 错误高度相关，价值分数也未必校准。可加入硬约束、外部测试、过程检查、多评价器和人工小样本校准；指标拆成 candidate recall、ranking accuracy、search success 和 cost。保留多样性，避免早期过度剪枝。
- **深挖**：增大 beam 一定好？成本上升且偏差仍在；如何校准 evaluator？标注中间状态和 pairwise ranking；无中间标签怎么办？用可执行 verifier 或最终结果反推，但注意信用分配。
- **项目映射**：无项目已实现证据。
- **评分/危险**：“搜索足够多总能找到正确答案”忽略评价瓶颈。

## Q76. Graph of Thoughts 与 LangGraph/StateGraph 有何区别？

- **来源映射**：`INT-NC-B02` 报告 LangGraph；GoT 为通用知识补充。
- **30 秒答案**：GoT 是把 thought 的生成、依赖、聚合和反馈表示成图的推理范式；LangGraph/StateGraph 是实现任意有状态工作流的控制框架。
- **2 分钟标准答案**：GoT 节点通常是候选思想或部分结果，边表示依赖/变换，支持合并多个 thought；StateGraph 节点可以是规则、API、Tool 或业务步骤。可以用 StateGraph 实现 GoT，但仅使用图框架不能证明采用 GoT。判断看运行语义：是否真的生成多个候选 thought、评价、聚合和反馈，而不是看类名。
- **深挖**：ESGQA 是 GoT 吗？不是，现有节点是领域阶段；LangGraph 的 conditional edge 是否等于搜索？不是；如何验证 GoT？保存候选图和评价/剪枝 Trace。
- **项目映射**：ESGQA 只引用 `P1-ESG-STATEGRAPH-001`。
- **评分/危险**：框架名与算法范式混淆是常见扣分点。

## Q77. LATS 如何把推理、行动和规划结合？

- **来源性质**：通用论文题，依据 `SRC-LATS-001`。
- **30 秒答案**：LATS 在树搜索中选择状态、生成动作、与环境交互、用 value/self-reflection 评价，再把价值回传祖先继续搜索。
- **2 分钟标准答案**：可类比 MCTS 的 selection、expansion、evaluation、backpropagation：LM 同时参与 proposal/value/reflection，真实 Observation 进入节点。它能探索多条交互轨迹，但成本高且 evaluator 偏差明显。生产上必须在可回滚 simulator/sandbox 中分支；数据库写入、付款等真实副作用不能并行试多条再选，最终只提交一次通过政策的动作。
- **深挖**：与 ToT 差异？LATS 更强调环境行动和价值回传；反思放哪？失败节点评价；何时不适用？不可回滚、低延迟或无可靠 value。
- **项目映射**：无项目已实现 LATS。
- **评分/危险**：在真实生产环境并行试错是严重安全问题。

## Q78. DAG 并行执行需要满足什么条件？

- **来源映射**：`INT-NC-B01` 的 Multi-Agent 并发；通用依据 `SRC-LLMCOMPILER-001`。
- **30 秒答案**：无数据依赖、资源不冲突、副作用可交换/幂等、deadline 可传播、结果可确定性合并。
- **2 分钟标准答案**：Planner 先验证 DAG 无环和输入来源，scheduler 只调度 ready set；设置 Provider/Tool/租户并发上限。每节点有 artifact Schema、deadline、取消和失败策略；join 用 reducer 合并，冲突时显式失败。关键路径决定理论延迟，下游共享数据库或限流可能让并行变慢。写操作除非有事务/幂等和一致性设计，通常串行。
- **深挖**：部分失败怎么办？fail-fast/best-effort/quorum 按任务；取消如何传播？父取消到未开始和在途节点；结果顺序不稳定？用 node id/版本确定性聚合。
- **项目映射**：不声称 agentHub 有任意 DAG 编译器。
- **评分/危险**：所有 Tool `Promise.all` 不满足依赖和副作用约束。

## Q79. Supervisor/Worker Multi-Agent 如何设计？

- **来源映射**：`INT-NC-B01/B02/B03` 多次报告 Multi-Agent 架构和编排。
- **30 秒答案**：Supervisor 分解、分派、监控和验收；Worker 依据真实能力/权限/上下文差异执行，双方通过 typed task/artifact 通信。
- **2 分钟标准答案**：任务契约包含 goal、input refs、required capability、owner、deadline、权限、output Schema 和 acceptance。Supervisor 不把整段聊天复制给每个 Worker，只给最小上下文；Worker 结果进入 artifact store，Supervisor 用确定性 verifier 或独立 reviewer 验收。要处理 worker health、重复派发、幂等、取消、死锁、消息风暴和责任归因。Supervisor 是单点和瓶颈，可把确定性调度从模型中抽出。
- **深挖**：persona 不同算 Multi-Agent 价值吗？不充分；共享 Memory 怎么办？按 scope 和版本；Supervisor 失败？持久任务状态和可恢复调度器。
- **项目映射**：agentHub 可提供 Runtime 基础，但效果与完整通信协议需独立证据。
- **评分/危险**：给同一模型三个名字不构成可靠协作。

## Q80. 什么时候 Multi-Agent 值得，什么时候只是过度设计？

- **来源映射**：`INT-NC-B01` 直接报告单/多 Agent 选择。
- **30 秒答案**：只有不同能力、权限、上下文隔离或可并行责任确需分开且指标证明收益时值得；可用函数、Tool 或单 Agent 子任务解决时不加 Agent。
- **2 分钟标准答案**：引入前做单 Agent baseline。Multi-Agent 的收益可能来自专业模型/工具、隔离敏感上下文、职责审查或并行；成本是 Token 复制、通信延迟、错误转述、调度、死锁和更大安全面。用 task success、P95、单位成功成本、错误归因和维护复杂度比较；如果收益只是 Prompt 更长或 persona 更丰富，优先单 Agent + typed tools。
- **深挖**：Reviewer Agent 值得吗？只有独立 rubric/工具和实测增益；并行总提升吗？受关键路径和限流；多模型是否更独立？仍需验证错误相关性。
- **项目映射**：不把 agentHub 的多个 Agent 当性能证明。
- **评分/危险**：“Agent 越多越智能”错误。

## Q81. 如何为 ESGQA 设计不夸大的 Hybrid 演进？

- **来源性质**：项目演进题。
- **30 秒答案**：保留 StateGraph 确定性外壳，只在开放证据缺口子图引入有界 ReAct；报告型任务可用 Plan/DAG，最终仍做证据、权限和业务校验。
- **2 分钟标准答案**：现有 memory/intent/domain/fallback 路径继续控制主流程。为“证据不足”增加最多 N 步的受限检索策略，只暴露搜索/检索只读 Tool，设置来源 allowlist、Token/step/deadline 和 progress detector；返回 typed evidence 后回到原图 evaluator。长报告可先产出带依赖的计划，并行只读检索，再确定性汇总。用固定问集比较证据覆盖、答案忠实、成本和循环率。所有内容是未来设计，冻结提交未实现。
- **深挖**：为何不整体改 ReAct？固定业务更可审计；风险动作？仍在图外壳；如何回滚？新策略 feature flag 和旧 Workflow fallback。
- **项目映射**：`P1-ESG-STATEGRAPH-001` 加三个 GAP 证据。
- **评分/危险**：把现有条件图直接改名 Hybrid/ReAct 属于包装。

## Q82. 如何在 agentHub 中增加可插拔 Agent 策略？

- **来源性质**：项目演进题。
- **30 秒答案**：在 Runtime 上定义 typed `DecisionPolicy`，分别实现 bounded ReAct、Plan 或 Search；Runtime 继续统一 Provider、Tool/权限、Context、队列、预算和终态。
- **2 分钟标准答案**：接口输入是 task state + model-visible context + capability，输出是 action/finish/clarify/replan 等有限 decision。Factory 注册策略，Runtime 校验并执行，不把权限交给策略。统一 event/terminal 协议，checkpoint 保存 policy/version 和 state。contract suite 覆盖动作、观察、取消、压缩、HITL、迟到事件和 Provider 切换；新策略 feature flag 灰度，与基线比较成功率、成本和循环。
- **深挖**：策略能直接调用 Provider 吗？可请求推理，但不能绕过 Runtime；Search 如何分支？sandbox state；如何兼容旧 Agent？默认策略保持现状。
- **项目映射**：复用 `P1-HUB-RUNTIME-001/P1-HUB-PROVIDER-001/P1-HUB-PLANNER-001`，用将来时。
- **评分/危险**：现有 Planner/事件循环不代表已经实现全部论文策略。

## Q83. 项目没用 ReAct，面试时怎样回答才不吃亏？

- **来源映射**：`INT-NC-A01` 说明 ReAct 会被直接问；本题服务项目诚信。
- **30 秒答案**：先完整讲 ReAct 机制和生产化，再解释项目为何选显式 Workflow，并给一个受限引入场景；明确当前未实现。
- **2 分钟标准答案**：先画 ReAct 的 decide/validate/authorize/act/observe/terminate，说明预算、Tool 安全和循环。然后回到业务：ESGQA 步骤和风险较可枚举，StateGraph 更便于状态、fallback 和审计；并不意味着不了解 ReAct。若证据缺口需要动态多源检索，可局部引入只读、有步数限制的 ReAct，再以评测证明是否值得。最后明确这属于演进方案，不是冻结代码现状。
- **深挖**：为何不一开始用 ReAct？复杂度与验证成本；项目算 Agent 吗？可称领域状态化 Agent/工作流并讲边界；如何验证升级？baseline 对比。
- **项目映射**：`P1-ESG-STATEGRAPH-001` 与 `P1-ESG-GAP-TOOL-REGISTRY-001`。
- **评分/危险**：“没用所以不懂”或“相似循环就是 ReAct”都失分。

## Q84. 面对新任务，如何系统选择 Workflow、ReAct、Planning、Reflection、Search 或 Multi-Agent？

- **来源映射**：多条 A/B 面经的架构选型综合题。
- **30 秒答案**：依次判断步骤可枚举性、环境动态性、依赖并行、是否有可信 verifier、能否回滚、主体是否真需分离，以及风险与预算上限。
- **2 分钟标准答案**：可枚举先 Workflow；必须根据中间 Observation 调整才局部 ReAct；长任务且依赖清晰用 Plan/DAG；有新反馈和可验收时才 Reflection；状态可复制/回滚且 evaluator 可靠才 Search；能力、权限或上下文必须隔离才 Multi-Agent。任何升级都从最简单 baseline 开始，以成功率、成本、延迟、安全和维护性证明。生产常见组合是确定性外壳 + 局部模型策略。
- **深挖**：无 verifier 怎么办？先定义人工/规则验收，避免盲目 Reflection/Search；高风险又开放？模型做建议，真实动作固定流程；多种范式能否叠加？可以，但每层需独立价值。
- **项目映射**：ESGQA 代表 Workflow/StateGraph，agentHub 代表 Runtime 基础；都不能替代任务分析。
- **评分/危险**：脱离任务直接推荐热门框架属于架构跟风。


