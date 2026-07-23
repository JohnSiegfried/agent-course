---
tags: [phase-1, interview, standardized-answers, real-interview-derived, rag, project-defense]
created: 2026-07-23
updated: 2026-07-23
status: complete
question_range: Q85-Q99
---

# Q85-Q99：公开面经衍生题标准答案（上）

> [!important] 来源与答案是两条证据链
> “公开面经映射”指向 [[real-interview-source-register]]，只证明有人报告过类似问题；“技术依据”指向 [[technical-source-register]]、论文或官方文档，负责约束答案。题目均为规范化改写，不宣称是企业官方逐字原题。

## Q85. 项目使用什么大模型，为什么这样选？

- **公开面经映射**：`INT-NC-A01`；`INT-NC-B01/B02`。字节、腾讯等公开面经都报告过项目模型与选型追问。
- **30 秒答案**：不应只报模型名。我会先给任务约束和候选集，再按质量、Tool/结构化能力、上下文、延迟、成本、部署与合规做离线集和小流量对比，最后说明项目当前真实选择及未验证项。
- **2 分钟标准答案**：先定义任务：是分类、RAG 生成、代码、Tool Calling 还是长上下文；再定义硬门槛，如中文、结构化输出、函数调用、私有化、数据地域和 SLA。对通过硬门槛的模型，用版本化 Golden Set 比 task success、事实忠实、Tool 选择/参数、Schema 通过率、P95、每个成功任务成本和安全拒绝。还要看 Provider 的限流、可取消、Usage、版本稳定和 fallback。最终不是“参数越大越好”，而是满足质量底线下综合成本最优，并保留模型/Prompt/参数版本以便复现。
- **严格追问**：为什么不用更大模型？用指标和约束回答；模型升级如何验证？固定集 + 失败 Trace + canary；没有对比实验怎么办？明确没有，不编造，再给实验设计。
- **项目映射**：ESGQA 冻结代码只证明多处使用 `ChatOllama`，见 `P1-ESG-GAP-PROVIDER-001`；若没有冻结配置中的精确模型值，不现场猜。agentHub 证明 `claude-code/opencode/test` Provider 注册，不等于证明底层固定模型或效果。
- **评分/危险答案**：高分要有硬门槛、指标和版本；“因为这个模型最强/最流行”无法应付追问。

## Q86. 推理框架和部署架构如何选，部署遇到什么问题？

- **公开面经映射**：`INT-NC-B01` 记录字节项目部署与推理框架追问。
- **30 秒答案**：按模型形态、吞吐/延迟、并发、硬件、量化、流式、Tool 集成和运维能力选；部署问题要用“症状、根因、改动、验证”回答，不罗列框架名。
- **2 分钟标准答案**：托管 API 关注能力、合规、限流和可用性；自部署关注模型大小、GPU 显存、KV Cache、批处理、并行、量化和调度。架构上入口鉴权/限流，模型网关做路由、熔断、Usage 和版本，推理服务负责 batching/streaming，Tool 与业务服务独立，Trace 串联。常见问题包括冷启动、OOM、长上下文吞吐下降、流中断、连接池、并发排队和版本漂移。回答真实问题时给监控证据和前后对比；没有运行记录就说“源码/设计已验证，部署未在本阶段复现”。
- **严格追问**：吞吐和延迟如何取舍？动态 batch 提吞吐但增加排队；为什么不用本地模型？看隐私、硬件与维护成本；灾难恢复？健康探测、兼容 fallback、幂等和降级。
- **项目映射**：ESGQA `P1-ESG-API-SECURITY-001` 证明 FastAPI/Celery 入口与异步任务代码，不证明当前机器或生产部署。agentHub `P1-HUB-SANDBOX-001` 是 Agent 会话容器，不等同模型推理集群。
- **评分/危险答案**：没有保存部署命令和输出时，不说“线上稳定运行”。

## Q87. Embedding 模型和向量数据库如何选？

- **公开面经映射**：`INT-NC-A01` 直接报告 Embedding 与常见向量数据库问题。
- **30 秒答案**：Embedding 按目标语言/领域、检索粒度、维度、最大输入、吞吐和许可评测；向量库按规模、ANN、过滤、更新、一致性、租户、备份和运维选，不能只看榜单或品牌。
- **2 分钟标准答案**：先建立领域 query-document 标注，比较 Recall@K、nDCG/MRR、跨语言和 OOD，同时记录编码 P95、显存、向量维度和存储成本。确认模型要求的 query/document prefix、归一化和距离度量。向量库小规模可用 FAISS/pgvector 简化；需要分布式、过滤、在线更新、复制和多租户时再考虑专用库或 Elasticsearch。选 ANN 索引时权衡 recall、latency、memory 和 build/update cost。数据库必须支持 metadata/ACL 过滤、版本化索引、备份恢复和观测。
- **严格追问**：cosine 与 inner product？向量 L2 归一化后 cosine 可转内积，依据 `SRC-FAISS-METRIC-001`；HNSW 参数怎么调？在目标数据上画 recall-latency-memory 曲线；榜单第一为何未必最好？领域、语言和 Chunk 分布不同。
- **项目映射**：ESGQA 使用 FAISS 和 BGE Reranker 的源码见 `P1-ESG-RETRIEVAL-001`，但当前材料没有运行指标证明选型最优。
- **评分/危险答案**：“Milvus 数据大、FAISS 数据小”只是入口，必须补过滤、一致性和评测。

## Q88. 从 Elasticsearch 切到向量检索，哪些能力会下降、提升，怎样迁移？

- **公开面经映射**：`INT-NC-A01` 直接报告该场景。
- **30 秒答案**：语义召回和同义表达可能提升，但精确词项、过滤、聚合、解释性和成熟运维可能下降；通常先做 hybrid，而不是把 ES 一刀切掉。
- **2 分钟标准答案**：ES/BM25 擅长专有名词、编号、精确匹配、布尔过滤、聚合和可解释打分；dense vector 擅长词面不同但语义相近。纯向量会受 Embedding 域偏移、近似索引和过滤影响，还增加编码成本。迁移步骤是冻结标注集 -> dual write/离线回填 -> lexical/dense/hybrid 对比 -> shadow query -> 小流量 canary -> 监控 Recall/nDCG、零结果、P95、成本和 ACL -> 保留回滚。融合优先用 RRF 或经归一化的线性权重；官方 Elastic 文档支持全文+向量与 RRF，见 `SRC-ELASTIC-HYBRID-001`。
- **严格追问**：为什么不直接换？没有 baseline 和回滚；ES 本身能做向量吗？可以，迁移可能是索引/查询模式变化而非换产品；如何处理老数据？版本化回填和校验。
- **项目映射**：ESGQA 当前是 BM25 + FAISS + Reranker，不是“从 ES 迁移成功”的项目证据；可用 `P1-ESG-RETRIEVAL-001` 说明 hybrid 思路。
- **评分/危险答案**：“向量检索理解语义，所以全面优于 ES”会被精确匹配和过滤反例击穿。

## Q89. Agent 选错工具、参数或调用顺序时如何系统排查？

- **公开面经映射**：`INT-NC-A02` 直接报告字节大模型应用面试问“Agent 调用工具不正确怎么办”。
- **30 秒答案**：先把错误分成该不该调、调哪个、参数、顺序、执行和结果消费六层，再用 Trace 判断问题来自 Tool 描述、可见集合、Prompt/Router、Schema、上下文还是模型能力。
- **2 分钟标准答案**：保留完整 replay：用户请求、模型/Prompt/Tool 版本、可见工具集合、candidate action、validation、ToolResult 和终态。若不该调用，改 abstain/direct-answer 数据与路由门；选错工具，减少重叠描述、加禁用场景和最小工具集合；参数错，收紧 Schema、枚举、示例和确定性业务校验；顺序错，用计划依赖或状态机；执行错按 Q12 错误分类；结果消费错用 typed Observation 和 verifier。先做 component test，再端到端回归，不要把所有问题归因 Prompt。
- **严格追问**：工具描述越长越好吗？会增加 Token 和冲突；为什么不给模型全工具？选择空间与攻击面扩大；如何量化？tool selection accuracy、argument validity、execution success、task success 分层。
- **项目映射**：ESGQA 没有通用 Tool Registry，不能说已做此治理；agentHub 的权限和 Provider 事件可作为 Runtime 改造基础。
- **评分/危险答案**：“再让模型反思/换大模型”没有定位根因。

## Q90. Tool Calling 效果差时，何时用 Prompt/Schema/Router，何时考虑 SFT 或 RL？

- **公开面经映射**：`INT-NC-A02` 在工具误调用后继续追问 SFT/RL。
- **30 秒答案**：先修数据、接口和 Runtime：工具集合、描述、Schema、校验、路由和反馈；只有错误在大量稳定样本上仍系统存在、工程规则难覆盖且收益覆盖训练成本时才考虑 SFT/RL。
- **2 分钟标准答案**：先建立错误 taxonomy 和 baseline。格式/参数错优先 Schema 与 constrained output；工具重叠优先 Tool 设计和 Router；上下文缺失先修检索；权限/副作用永远由 Runtime 控制。SFT 适合学习稳定的调用格式、工具选择和轨迹，需要高质量成功/纠错样本；RL 可优化长程策略或过程奖励，但奖励设计、仿真环境、信用分配和 reward hacking 成本高。即使用训练提升模型，也不能移除执行前校验和安全门。
- **严格追问**：什么指标触发训练？按错误分桶的剩余率、样本量、业务收益和工程成本；能用线上事故直接训练吗？需脱敏、去泄漏和人工校验；RL 奖励怎么做？属于后续训练阶段，本阶段只讲升级门槛。
- **项目映射**：两个冻结项目没有 SFT/RL Tool Calling 训练证据，不能包装。
- **评分/危险答案**：“微调后模型就不会调错工具”忽略分布漂移和安全。

## Q91. 模型怎样决定调用工具还是直接回答？

- **公开面经映射**：`INT-NC-B02` 报告腾讯面试问 thinking 阶段如何决定 Tool 还是回复。
- **30 秒答案**：先由策略过滤：必须工具、禁止工具和可直接回答场景；剩余长尾由模型在 `direct/clarify/tool/refuse` 的有限动作中选择，再由 Runtime 校验。
- **2 分钟标准答案**：是否调用取决于知识新鲜度、确定性计算、是否需要外部副作用、证据要求和风险。实时数据、私有知识、精确计算和真实动作通常必须 Tool；闲聊、已在上下文中的解释可直接回答；信息不足先澄清；越权则拒绝。实现上给当前 actor 最小工具集和明确“何时不用”，输出 typed decision + reason code + required evidence；Runtime 验证 required-tool 规则、权限和预算。评测四类混淆，尤其关注“本应调用却直接编造”和“无必要调用导致成本/风险”。
- **严格追问**：模型自报 confidence 够吗？不够；RAG 是 Tool 还是固定步骤？都可，按业务可预测性选择；工具失败后能直接回答？只有明确标注降级且不伪造事实。
- **项目映射**：ESGQA 主要由状态图固定路由，不是模型自由 Tool 选择；agentHub Runtime 也不自动证明该决策策略。
- **评分/危险答案**：“模型根据 Prompt 自己判断”缺少规则、校验和指标。

## Q92. 如何限制 Tool 的可见性、次数、参数、权限和成本？

- **公开面经映射**：`INT-NC-A01` 直接报告“限制工具调用怎么做”。
- **30 秒答案**：五道门：任务/角色过滤可见工具；step/tool budget；Schema 与业务校验；actor-bound authz/HITL；timeout、rate limit、cost ceiling 和 circuit breaker。
- **2 分钟标准答案**：Registry 先按 tenant、role、task 和环境给最小 Tool 集；每个 Tool 有 per-run/per-tool 次数、Token、金额和 deadline；参数 canonicalize 后做类型、业务、权限与副作用校验；高风险动作审批绑定参数摘要；执行层再做并发、限流、熔断、幂等和网络/文件沙箱。防止模型换名字绕过，权限绑定稳定 Tool ID/版本；Trace 记录拒绝原因和剩余预算。限制触发时返回 typed terminal/clarify，不让模型无限换工具。
- **严格追问**：只隐藏 Tool 是否安全？不，执行器必须强制；预算按谁计？session/actor/tenant 多层；Tool 链间接调用怎么办？传播 capability 和剩余预算。
- **项目映射**：agentHub `P1-HUB-PERMISSION-001` 有角色工具/写路径检查，但没有证据证明完整五道门；ESGQA 无统一 Registry。
- **评分/危险答案**：Prompt 写“最多调用三次”不是不可绕过限制。

## Q93. Agent Prompt 怎样设计、版本化、测试和防止职责越界？

- **公开面经映射**：`INT-NC-B02/B03` 报告“写 Agent Prompt”和幻觉约束。
- **30 秒答案**：Prompt 明确角色/目标、输入边界、可用动作、停止/澄清/拒绝、输出 Schema 和不可信数据规则；版本化并与模型、Tool Schema、策略和评测集一起测试。
- **2 分钟标准答案**：System 部分只放稳定职责和安全原则；动态业务规则来自版本化 Policy；用户/检索/ToolResult 明确标为不可信 data。不要要求 Prompt 自己执行权限，而是告诉模型输出候选动作；完成条件、无证据时的 abstain、错误返回和禁止猜测要明确。Prompt 以 ID/hash 管理，变更记录动机和兼容模型；用正常、边界、OOD、Prompt Injection 和 Tool 错误回放做离线 A/B，再 shadow/canary。指标包括 task success、tool error、拒答、成本、长度和安全回归。
- **严格追问**：Few-shot 越多越好吗？会占预算并造成过拟合；系统 Prompt 能否包含密钥？不能；如何防 Tool 描述注入？Tool 元数据受信来源、签名/审核和最小集合。
- **项目映射**：ESGQA 有多处节点 Prompt，但现有证据不足以证明统一 Prompt Registry/版本评测；回答应区分代码现状与建议。
- **评分/危险答案**：把所有业务、权限和知识塞进一个超长 Prompt 是设计异味。

## Q94. LangChain 与 LangGraph 有什么区别，如何选？

- **公开面经映射**：`INT-NC-B02` 报告腾讯内容服务部面试直接询问二者区别。
- **30 秒答案**：LangChain 提供模型、Tool、Agent Loop 和大量集成等较高层抽象；LangGraph 是更低层的有状态编排 Runtime，强调图、持久执行、流式和 HITL。简单标准 Agent 先 LangChain，需要显式状态/分支/恢复时用 LangGraph。
- **2 分钟标准答案**：官方文档说明 `create_agent` 是预构建 Agent，并建立在 LangGraph 上；LangGraph 可独立使用，重点是 state/nodes/edges、durable execution、persistence、streaming 和 HITL，见 `SRC-LANGCHAIN-AGENT-001`、`SRC-LANGGRAPH-OVERVIEW-001`。二者不是竞争替代：可用 LangChain 的模型/Tool 集成作为节点组件，用 LangGraph 编排复杂流程。选型看控制权、状态恢复、长任务和测试，而不是“哪个更新”。使用 LangGraph 仍需自己设计状态、终止、权限和业务校验。
- **严格追问**：用了 LangGraph 就持久化了吗？需配置 checkpointer/存储；StateGraph 等于 Agent 吗？不是；什么时候不用二者？简单 API/Workflow 可直接代码实现。
- **项目映射**：ESGQA 使用 StateGraph，可说显式业务图；不能宣称框架所有能力已配置。agentHub 是自定义 Runtime，不等于 LangGraph。
- **评分/危险答案**：“LangChain 是链，LangGraph 是图”只答了名字。

## Q95. 上下文压缩方案怎样设计，阈值和保留轮数如何确定？

- **公开面经映射**：`INT-NC-A01` 报告三级压缩和“为什么五轮”；`INT-NC-B04` 报告上下文压缩。
- **30 秒答案**：按去重/外置、选择、摘要、重建分级；阈值和 N 轮不是经验拍脑袋，要以 Token 分布、任务质量、摘要保真、成本和延迟实验确定，并设迟滞。
- **2 分钟标准答案**：一级去掉重复 Tool 输出和无关噪声，大 artifact 外置；二级按目标、未完成状态、可信度和时效检索；三级把已完成历史压成 typed summary，保留约束、决策、证据、open items 和引用。高水位触发、低水位恢复，并为输出/ToolResult 留余量。候选阈值和最近轮数在真实 Trace 回放上比较 task success、constraint retention、faithfulness、压缩次数、P95 和 Token。不同任务可用不同策略，不把“五轮”当普适结论。
- **严格追问**：摘要由同一模型是否可靠？需 verifier、原始引用和回放；为何不每轮摘要？成本和累积损失；如何测忘记？约束问答、未完成任务和长程依赖集。
- **项目映射**：agentHub 当前硬 70% 阈值和 pending prompt 由 `P1-HUB-COMPRESSION-001` 证明，但无迟滞/最优性证据。ESGQA 最近 4 条是代码事实，不是实验结论。
- **评分/危险答案**：“业内都保留五轮/70% 最优”没有依据。

## Q96. Recall@K 如何计算、构造标注并证明数字可信？

- **公开面经映射**：`INT-NC-A01` 直接报告 Recall@5 数字追问。
- **30 秒答案**：对每个 query 定义相关集合 `R_q`，`Recall@K = |TopK_q ∩ R_q| / |R_q|`，再报告 macro/micro、样本数、置信区间、数据版本和切片；单个 91.2% 没有这些信息不可审计。
- **2 分钟标准答案**：先明确评测单位是 document、chunk 还是 evidence，以及一问是否多证据。由领域标注者制定 relevance rubric，双人标注/仲裁；测试集与开发调参隔离，去重并防同文档泄漏。冻结 corpus、Chunk、Embedding、index、query set 和 K，保存每问 topK。除了 Recall@K，还看 HitRate、MRR/nDCG、precision、latency 和 end-to-end answerability，因为 Recall 高不代表排序和生成好。按主题、长尾、时间和权限切片，bootstrap 给置信区间，并展示失败案例。
- **严格追问**：只有一个相关 chunk 时？Recall@K 退化为 hit/no-hit；K 为什么是 5？结合 Context budget 和候选成本，在曲线上选；人工标注不全怎么办？pooling、主动补标并声明 incompleteness。
- **技术依据**：`SRC-BEIR-001` 支持异构检索基准和方法权衡；公式是标准 IR 定义，不把公开面经中的数字带入项目。
- **项目映射**：ESGQA 没有当前 Recall@K 运行证据；不能使用其他人的 91.2%。
- **评分/危险答案**：“我随机测了 100 条，命中 91 条”还缺标注、版本、K 和泄漏说明。

## Q97. 评测集为什么要更新，如何防止数据泄漏和对评测集过拟合？

- **公开面经映射**：`INT-NC-A01` 直接报告“评测集有没有更新、怎么防过拟合”。
- **30 秒答案**：业务、知识、用户表达和失败模式会漂移；评测集需版本化滚动，但稳定 core test 必须保留。训练/dev/test、时间和来源隔离，禁止用最终 test 反复调参。
- **2 分钟标准答案**：维护三层：长期稳定 core set 看可比趋势；近期 production slice 看分布漂移；事故/adversarial set 防回归。新样本先去重、脱敏、标注和难度/来源分层，再发布版本。调 Prompt/检索参数只看 train/dev，最终 test 低频解封；多轮比较做统计校正并保留 blind holdout。若失败 Trace 被加入训练，应另建未来时间窗测试。报告每次评测的数据版本、覆盖、变更和置信区间。
- **严格追问**：一直增加难题会导致分数下降怎么办？报告分层和同版本趋势；线上日志能直接做标签吗？不能，需标注和偏差处理；如何发现泄漏？近重复、文档/用户/时间分组和哈希审计。
- **项目映射**：两个项目没有完整数据集治理证据，回答是规范方案。
- **评分/危险答案**：“评测集固定才能公平”忽略业务漂移；“每次把错题加进去”也会让 test 变 dev。

## Q98. RAG 知识库如何不停服更新并保证版本一致性？

- **公开面经映射**：`INT-NC-B02` 报告腾讯 AI 应用后端面试询问 RAG 不停服更新。
- **30 秒答案**：构建新版本索引，离线校验后用逻辑 alias/路由原子切换；请求固定读取一个 index version，旧版本保留回滚，增量写用 CDC/dual write 追平。
- **2 分钟标准答案**：文档进入 staging，解析/Chunk/Embedding 生成 `kb_vN+1`；做数量、checksum、抽样检索、ACL 和评测校验。全量构建期间，用变更日志/CDC 记录增量并追平。通过后原子把 read alias 从 `vN` 切到 `vN+1`；Elastic alias 官方支持原子多动作和无停机 reindex，见 `SRC-ELASTIC-ALIAS-001`。每个请求/Trace 记录 index version，缓存 key 包含版本。切换后观察零结果、质量、P95 和错误，旧索引在回滚窗口后再回收。写索引与读索引职责明确，失败不影响旧版服务。
- **严格追问**：向量库没有 alias 怎么办？应用层 version routing/蓝绿 collection；更新中删除文档？tombstone + CDC；多个索引混用？请求级快照/版本，避免 retrieval 和引用跨版本。
- **项目映射**：ESGQA 有 FAISS 本地索引和缓存，但没有不停服双索引切换证据；只能提出改造。
- **评分/危险答案**：原地重建并覆盖文件可能造成部分索引、缓存污染和请求中断。

## Q99. RAG 与直接让 LLM 回答有什么区别，何时反而不该用 RAG？

- **公开面经映射**：`INT-NC-A05` 直接报告京东 Agent 二面询问该题。
- **30 秒答案**：RAG 在生成前检索外部非参数知识，使知识可更新、可引用和可做私域控制；代价是解析、召回、排序、权限、延迟和错误证据链。非知识密集、上下文已充分或确定性查询可直接完成时不一定用。
- **2 分钟标准答案**：直接 LLM 依赖参数记忆，链路短但知识可能过期、无私域数据且难追溯。RAG 由 ingestion/index、retrieval/rerank、context building 和 grounded generation 组成，适合动态政策、企业文档和需要引用的知识密集任务，基础定义见 `SRC-RAG-001`。它不保证消除幻觉：没召回、召回错误、权限错或模型不忠实都会失败。闲聊、改写、分类、简单计算，或答案已在受信当前上下文时，RAG 可能只增加噪声；结构化精确数据优先 SQL/API。
- **严格追问**：RAG 和微调怎么选？知识更新/引用优先 RAG，行为/风格/稳定格式可能训练；RAG 一定比长上下文省吗？取决于语料和调用；如何拒答？证据不足阈值 + verifier + 明确 provenance。
- **项目映射**：ESGQA `P1-ESG-RETRIEVAL-001` 证明混合检索代码，不证明 RAG 必然降低幻觉。
- **评分/危险答案**：“RAG 能解决幻觉”应改成“在证据正确且生成忠实时降低部分事实错误”。


