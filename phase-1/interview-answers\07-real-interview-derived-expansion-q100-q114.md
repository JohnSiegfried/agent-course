---
tags: [phase-1, interview, standardized-answers, real-interview-derived, multi-agent, evaluation]
created: 2026-07-23
updated: 2026-07-23
status: complete
question_range: Q100-Q114
continues: 06-real-interview-derived-expansion
---

# Q100-Q114：公开面经衍生题标准答案（续）

## Q100. RAG 有哪些检索策略，常见向量相似度如何选择？

- **公开面经映射**：`INT-NC-B03` 报告百度面试询问检索策略和向量匹配指标。
- **30 秒答案**：策略包括 metadata/规则过滤、BM25、稀疏向量、dense ANN、hybrid、query rewrite/分解和多阶段 rerank；相似度必须与 Embedding 训练方式、归一化和索引实现一致。
- **2 分钟标准答案**：先做 ACL、时间、类型等硬过滤；词项精确场景用 BM25/稀疏，语义同义用 dense，生产常用 hybrid 扩大候选再 rerank。复杂问题可 query rewrite、多查询、HyDE 或子问题分解，但要评估漂移。距离上 cosine 看方向；inner product 同时受方向和模长影响；L2 看欧氏距离。FAISS 官方说明归一化向量后 cosine 可用 inner product，且与 L2 排名有对应关系，见 `SRC-FAISS-METRIC-001`。不能在不归一化时把内积分数直接叫 cosine。
- **严格追问**：为什么不默认 cosine？模型训练目标可能是 dot product/L2；hybrid 怎么融合？RRF 避免原始分数不可比，线性融合需归一化和标注调权；query rewrite 何时变差？丢失专名、否定和过滤条件。
- **项目映射**：ESGQA 采用 BM25 + FAISS + Ensemble + BGE Reranker，证据 `P1-ESG-RETRIEVAL-001`；0.4/0.6 权重是代码事实，不是普适最佳值。
- **评分/危险答案**：“向量检索就是 cosine topK”缺少训练契合、过滤和多阶段检索。

## Q101. 为什么加 Reranker 后效果可能下降，如何定位？

- **公开面经映射**：`INT-NC-B02` 汇总的腾讯 AI Agent 二面明确报告该追问。
- **30 秒答案**：候选召回不足、reranker 域不匹配/截断、query-document 格式错误、分数/排序方向、topN 太小或业务规则冲突都可能使效果下降；先确认正确证据是否进入候选池，再看重排是否把它降了。
- **2 分钟标准答案**：固定同一 query set，保存 first-stage candidates 和 labels。若相关项没进候选，问题在 ingestion/chunk/retrieval；若进入却被降，检查模型语言/领域、query-document 顺序/prefix、最大长度截断、batch/padding、score direction、版本和 topN。再检查业务 boost、去重和过滤是否在 rerank 前后改变集合。分开报告 candidate Recall@N、rerank nDCG/MRR、P95 和最终 answerability；对失败 query 人工看 hard negatives。官方 Elastic 将 rerank 定位为对第一阶段候选的更昂贵排序，见 `SRC-ELASTIC-RERANK-001`。
- **严格追问**：reranker 分数能设统一阈值吗？需在目标分布校准；topN 越大越好吗？质量上限增加但成本/噪声上升；Cross-Encoder 必然优于 embedding？不保证 OOD 和截断。
- **项目映射**：ESGQA 确有 BGE Reranker，但没有冻结评测证明一定提升；正确表达是“实现了该阶段，并应做消融”。
- **评分/危险答案**：“重排模型更强，所以效果只会更好”错误。

## Q102. 短期记忆何时转长期记忆，如何合并、淘汰和防污染？

- **公开面经映射**：`INT-NC-B02` 报告短期/长期记忆转换、触发和淘汰的连续追问。
- **30 秒答案**：只有跨会话有复用价值、来源可信、用户允许且通过敏感检查的信息才晋升；按事实/偏好/决策结构化合并，用 TTL、价值和冲突治理替代简单 FIFO。
- **2 分钟标准答案**：短期保存当前对话和 working state；候选长期记忆由明确用户陈述、反复确认或已验证业务事件触发。写入前分类、脱敏、去重，记录 source、time、scope、confidence、TTL 和 consent。合并时同一事实版本化，冲突不静默覆盖；偏好随时间衰减，关键决策保留 provenance。容量管理按效用、时效、访问和法务要求归档/删除，不只先进先出。读取先做 tenant/actor 权限和污染过滤；用户应能查看、更正和遗忘。
- **严格追问**：由 LLM 自动总结是否够？不够，需 Schema、规则/verifier 和写入门；长期记忆一定淘汰吗？可归档并按需检索，但合规删除必须真实删除；恶意文档能写入吗？外部数据默认不晋升为用户事实。
- **项目映射**：ESGQA 有近期 4 条、长期画像、FAISS 兜底，但无上述完整晋升/TTL/用户治理，见 `P1-ESG-MEMORY-001`。
- **评分/危险答案**：“超过五轮就摘要进向量库”是时间规则，不是记忆质量策略。

## Q103. 使用状态机时，模型如何感知当前状态而不控制真实状态？

- **公开面经映射**：`INT-NC-B02` 报告状态机能力和模型如何感知状态。
- **30 秒答案**：Runtime 保存 authoritative state，只把经过权限过滤的 typed state view、允许动作和最近 Observation 给模型；模型返回候选 transition，Runtime 校验后原子提交。
- **2 分钟标准答案**：状态机定义合法状态、事件、guard 和 transition。每次调用构建 `StateView`：当前阶段、目标、已验证 artifact、未完成条件、预算和可选 action，不暴露连接/密钥或可伪造的真实提交字段。模型输出 action + parameters + reason code；Runtime 检查当前版本、合法边、权限和业务不变量，用 compare-and-swap/事务更新。Tool 回调带 execution id，迟到事件不能把终态改回进行中。
- **严格追问**：把整个 state JSON 给模型可以吗？会泄露、污染和超预算；模型说已支付怎么办？真实状态只读账务系统；并发 transition？版本号/锁/事件序列化。
- **项目映射**：ESGQA 的 `CBAMState` 与条件边可作领域例子；agentHub Runtime/事件是另一层。都应强调模型不是 state owner。
- **评分/危险答案**：让模型自由返回任意 `next_state` 并直接写库会破坏不变量。

## Q104. Multi-Agent 之间如何传输数据和共享上下文？

- **公开面经映射**：`INT-NC-B01` 汇总蚂蚁 Agent 开发一面中的 Agent 间通信问题。
- **30 秒答案**：用带 Schema、版本、owner、权限、provenance 和 artifact 引用的消息/任务契约，不复制整段对话；共享状态按 scope 和单写者/版本规则管理。
- **2 分钟标准答案**：Supervisor 下发 TaskEnvelope：task id、goal、input refs、capability、deadline、权限和 acceptance；Worker 返回 ResultEnvelope：status、typed output、evidence refs、error 和 usage。大文件放 artifact store，消息只传引用和摘要。共享 Context 区分 global/team/private，按最小权限检索；关键 artifact 指定 owner，更新用 version/CAS 或 append-only event。通信可同步 RPC、队列或事件总线，至少处理去重、重放、超时、取消、背压和 trace propagation。
- **严格追问**：为何不共享同一聊天历史？Token 复制、泄露和责任混乱；两个 Agent 修改同一计划？单 writer 或合并协议；消息如何防注入？来源标记、Schema 和接收方策略。
- **项目映射**：agentHub 多 Agent 能力可作为 Runtime 背景，但冻结证据不能证明上述完整协议；项目回答要引用实际文件后再说。
- **评分/危险答案**：“Agent 互相聊天”只是表象，不是通信设计。

## Q105. 多个 Agent 并发操作数据库或文件时如何保证一致性？

- **公开面经映射**：`INT-NC-B01` 汇总蚂蚁一面连续询问 Multi-Agent 并发与共享 DB/文件。
- **30 秒答案**：先减少共享写并分配 owner；数据库用事务、唯一约束、乐观锁/幂等和 outbox，文件用 workspace 隔离、版本 hash、原子写和 merge/冲突检测。
- **2 分钟标准答案**：调度前标注资源和读写集；无冲突只读可并行，同一资源写默认串行或由单 writer service 处理。数据库更新带业务幂等键和 expected version，事务内检查不变量；跨服务用 saga/outbox 而非模型补偿。代码文件每 Agent 独立分支/worktree，patch 基于基线 hash，合并前检测重叠 diff、运行测试并由确定性工具合并。锁要有 lease/fencing token，不能只设过期时间；失败/取消保存 operation state。
- **严格追问**：分布式锁服务挂了？fencing token + 存储层版本；两个 Agent 修改不同文件就一定安全？可能有语义/接口冲突；冲突让 LLM 合并可以吗？可提议，仍需版本、测试和审查。
- **项目映射**：现有项目没有证明通用多 Agent 共享资源一致性；只能讲设计。agentHub 角色写路径有 `P1-HUB-PERMISSION-001`，权限不等于一致性。
- **评分/危险答案**：`Promise.all` + 最后写入者获胜会丢更新。

## Q106. Agent 失败、中断或恢复时，怎样做到重试安全？

- **公开面经映射**：`INT-NC-B02` 报告腾讯 AI 应用后端面试的失败/中断与重试安全。
- **30 秒答案**：把任务拆成可检查的 step，每步有 execution id、checkpoint、幂等/补偿和真实副作用状态；恢复先核验已完成事实，只重放安全的未完成步骤。
- **2 分钟标准答案**：持久 TaskState 保存 plan/version、step status、input/output artifact、attempt、deadline 和 terminal reason。读操作可有界重试；写操作用 idempotency key、唯一约束或状态机，超时标 unknown 并查询；不可逆动作在 commit 前 HITL/事务门。恢复时读取 checkpoint，与外部系统对账，已成功步骤不重跑；在途步骤按 execution id 去重，迟到结果按版本处理。Retry policy 按错误 taxonomy，带退避、jitter、熔断和总预算；补偿本身也是显式幂等业务步骤。
- **严格追问**：checkpoint 写成功但业务写失败？需要事务/outbox 或可对账；恰好一次能保证吗？通常通过 at-least-once + 幂等逼近业务 exactly-once；Provider 会话丢失？从结构化状态重建，不依赖聊天缓存。
- **项目映射**：agentHub 有队列、Provider 生命周期和 Manager abort/replan，但没有证据证明所有 Tool 副作用重试安全。
- **评分/危险答案**：“失败从头再跑”会重复成功副作用。

## Q107. 跨境汇款等高风险业务中，Agent 超时/失败如何确保资金安全？

- **公开面经映射**：`INT-NC-B02` 汇总腾讯面试的跨境汇款场景题。
- **30 秒答案**：Agent 只做材料收集、解释和候选指令；真实账务由确定性交易状态机、额度/合规规则、双重确认、幂等和账本服务提交。超时进入“处理中/待核验”，绝不盲重试。
- **2 分钟标准答案**：流程分 quote、validate/KYC-AML、user confirmation、submit、settle/reconcile。模型不能改金额、收款人、币种或跳过规则；批准绑定规范化参数和汇率有效期。提交使用 payment id/idempotency key，账本服务事务性写入并返回权威状态；客户端超时先查询/对账。状态只能按合法边迁移，例如 `CREATED -> VALIDATED -> CONFIRMED -> SUBMITTED -> SETTLED/FAILED/REVIEW`，unknown 不等于 failed。异常触发冻结、人工审核和审计，补偿按金融业务规则而非自动“反向转账”。
- **严格追问**：模型识别到“紧急”能绕额度吗？不能；Provider 挂了怎么办？交易提交不依赖生成模型，降级到固定流程/人工；如何防参数替换？签名摘要、一次性审批、执行前重验。
- **项目映射**：ESGQA 与 agentHub 都没有跨境汇款实现证据。本题只能作为系统设计，严禁映射成项目成果。
- **评分/危险答案**：“Agent 重试三次，失败就退款”不理解未知结果、账本和合规。

## Q108. 如何介绍 Agent 项目的目标、架构、团队和个人贡献？

- **公开面经映射**：`INT-NC-A04` 直接报告美团询问项目做什么；`INT-NC-B01` 多场项目架构追问。
- **30 秒答案**：用“业务问题与验收 -> 一条调用链 -> 我的边界与关键决策 -> 证据/结果 -> 缺口与演进”回答，团队规模和个人贡献必须可核验。
- **2 分钟标准答案**：先说用户、痛点、为什么不用规则/搜索/单次 LLM，以及成功标准。白板画入口、编排、模型、RAG/Memory、Tool、数据和观测，沿一个请求走状态与失败。再讲自己实际负责的文件、接口、决策和 review，而不是把团队全部成果说成个人。结果分源码已验证、运行已验证和文档声明；给真实数据版本/实验记录。最后主动讲一个未实现能力和下一步验证，显示边界感。
- **严格追问**：项目多少人？如实说角色和接口，不猜；为什么用 Agent？对比确定性 baseline；你的代码在哪？给 commit/file/symbol/call chain。
- **项目映射**：ESGQA 可从 `P1-ESG-STATEGRAPH-001` 走图；agentHub 可从 `P1-HUB-RUNTIME-001` 走事件。个人贡献若当前材料未记录，不能由课程替你虚构。
- **评分/危险答案**：框架堆栈介绍超过业务和调用链，容易暴露并非亲手实现。

## Q109. 如何回答“项目最大挑战是什么”并证明不是套话？

- **公开面经映射**：`INT-NC-A04` 直接报告美团追问项目最大挑战。
- **30 秒答案**：选一个真实且高影响的问题，按症状/约束、根因证据、候选方案、取舍、实现、验证和残余风险讲；没有量化记录就不编数字。
- **2 分钟标准答案**：挑战不应是“模型幻觉很严重”这种泛词。应具体到可定位链路，例如“混合检索候选里有证据但 rerank 降权”“Provider busy 时新 prompt 与压缩竞态”“未知意图默认进入 calculation”。给 Trace/源码位置说明如何发现，列至少两个方案和为何选当前方案，描述失败路径和回归测试。结果只报告保存的指标；若未运行验证，就说完成了源码分析/设计，并给计划中的测量方法。
- **严格追问**：为什么这是最大挑战？用影响范围、频率和阻塞路径；你个人做了什么？精确到提交/函数；方案副作用？主动说明成本、延迟或维护代价。
- **项目映射**：ESGQA 可选路由默认、Memory/检索边界；agentHub 可选压缩状态机、Provider 事件或权限默认，但必须与真实个人经历对齐。
- **评分/危险答案**：“时间紧、技术难、沟通复杂”没有工程证据。

## Q110. 项目没有正式用户或没有上线，应该怎样回答？

- **公开面经映射**：`INT-NC-B02` 报告腾讯面试直接追问是否有正式用户、为何没上线。
- **30 秒答案**：直接说明验证层级和没上线原因，不把 Demo 当生产；再讲已完成证据、上线门槛、风险清单和最小灰度计划。
- **2 分钟标准答案**：我会说“当前是源码/离线 Demo/内部试用中的哪一级，没有正式用户数据”。解释原因要具体：需求阶段、数据/合规、质量门槛、成本、运维或安全缺口，而不是模糊“时间原因”。已验证内容给 commit、测试或离线集；未验证内容明确列出。上线计划是 shadow -> 内部白名单 -> 只读低风险 -> 小流量 canary，并配置 SLO、评测、回滚、审计和人工兜底。
- **严格追问**：没用户怎么证明价值？访谈/任务集/baseline/可用性测试，但不等同线上；为什么写进简历？说明完成的工程难点；上线最大 blocker？给一项可验证门槛。
- **项目映射**：Phase 1 明确没有新增运行验证；回答必须使用“源码已验证”，不能改成“线上验证”。
- **评分/危险答案**：“已经上线但用户量不方便透露”若无证据是严重诚信风险。

## Q111. 面试官质疑准确率、召回率、性能提升数字时如何自证？

- **公开面经映射**：`INT-NC-A01` 的 Recall@5 追问；`INT-NC-B02` 的项目效果和量化追问。
- **30 秒答案**：给指标定义、基线、数据版本/样本量、切分/标注、实验环境、重复次数/置信区间、原始记录和失败案例；缺一项就降低结论强度。
- **2 分钟标准答案**：先复述公式和单位，例如 Recall@5 是 chunk 还是 document；说明 baseline 与唯一变量，避免同时换模型、Chunk 和 Prompt。给数据来源、时间、去重、train/dev/test 隔离、标注规则和切片；性能给硬件、并发、warmup、P50/P95/P99、吞吐和错误率，不只平均值。保存脚本、配置、commit、seed 和结果 artifact，可复跑。对外报告置信区间和适用范围，展示失效场景。没有这些记录时诚实撤回数字，改为定性源码事实和评测计划。
- **严格追问**：提升 50% 是相对还是绝对？必须说清；只跑一次？不能估稳定性；为什么选这个 baseline？与真实替代方案一致。
- **项目映射**：[[project-evidence-map]] 明确当前没有新增运行已验证标签，因此课程不为 ESGQA/agentHub 生成任何性能数字。
- **评分/危险答案**：借用论文、博客或别人的面经数字到自己项目，属于不可接受的事实错误。

## Q112. 当前 Agent 落地最关键的挑战有哪些？

- **公开面经映射**：`INT-NC-A03` 直接报告美团 Agent 算法面试要求介绍 Agent 与当前挑战。
- **30 秒答案**：我归纳为可靠决策与评测、上下文/记忆、Tool/副作用安全、长任务恢复、成本/延迟、可观测与治理；核心不是模型“会不会想”，而是系统能否稳定完成且失败可控。
- **2 分钟标准答案**：第一，模型非确定且错误相关，需 typed action、verifier 和分层评测；第二，上下文有限，检索/压缩/记忆会丢信息或被污染；第三，Tool 把文本错误放大成真实副作用，需权限、幂等、sandbox 和 HITL；第四，多步任务有循环、计划漂移、并发和恢复；第五，调用链长导致 P95、成本和供应商依赖；第六，线上 Trace、数据治理、Prompt/模型版本和合规。应按业务风险选择 Workflow + 局部 Agent，而不是追求最大自治。
- **严格追问**：哪个最难？按目标岗位和真实项目给一项并展开证据；模型变强后哪些仍存在？权限、业务状态、合规、观测不会消失；如何排优先级？风险×频率×可检测性和业务价值。
- **项目映射**：ESGQA 显示 Tool/Provider/Sandbox 缺口；agentHub 显示权限、容器和压缩残余风险，可作为具体例子。
- **评分/危险答案**：“幻觉、上下文、成本”只列三个词不够，需机制和控制。

## Q113. Agent Skill、Tool、Function Calling 与 MCP 如何区分和组合？

- **公开面经映射**：`INT-NC-B03` 报告百度面试询问 Agent Skill 开发；Function Calling/MCP 在多来源出现。
- **30 秒答案**：Skill 是面向任务的可复用能力包/流程说明，可能组合 Prompt、Tool 和脚本；Tool 是可执行能力；Function Calling 是模型表达调用意图；MCP 是跨进程发现/调用工具、资源和 Prompt 的协议。
- **2 分钟标准答案**：Tool 有明确输入输出和副作用；Function Calling 只生成结构化 action，Runtime 执行并授权；MCP 规定 Host/Client/Server 和能力协商，让外部 Server 暴露 tools/resources/prompts，依据 `SRC-MCP-SPEC-001`。Skill 的具体定义依平台而异，通常比单 Tool 更高层，封装“何时用、步骤、资源、脚本和验收”，但不能绕过 Runtime 权限。典型组合：Skill 被任务选择 -> Registry/MCP 发现所需 Tool -> 模型用 Function Calling 提议 -> Runtime 校验执行 -> Skill 验收。
- **严格追问**：Skill 是行业统一协议吗？不是，必须先说明具体平台语义；MCP 自带安全么？不，需认证、最小 scope、SSRF/TOCTOU 防护，见 `SRC-MCP-SEC-001`；Skill 能直接写文件吗？通过受控 Tool。
- **项目映射**：ESGQA 无通用 Skill/Tool Registry/MCP；agentHub 即使有 Agent 配置，也不能未经源码证据称完整 Skill 平台。
- **评分/危险答案**：四者都说成“插件”会丢失抽象层和安全责任。

## Q114. RAG 评测与 Agent Trace 评测有什么不同，怎样联合评估？

- **公开面经映射**：`INT-NC-B01/B04` 报告 RAG 评测；`INT-JG-C01` 用作 Trace 评测趋势校验。
- **30 秒答案**：RAG 评测关注 ingestion、候选召回/排序、上下文和答案忠实；Agent Trace 还关注决策、Tool、状态、重试、终止、副作用和成本。联合时既看每步，也看端到端任务成功。
- **2 分钟标准答案**：RAG 层：解析/Chunk 完整性、ACL、Recall@K、MRR/nDCG、context precision/coverage、citation/faithfulness、answer correctness、延迟和新鲜度。Agent 层：route accuracy、plan validity、tool selection/arguments/success、loop/progress、memory use、permission violations、unknown outcomes、terminal reason 和 task success。联合数据记录 query、corpus/index/model/Prompt/Tool/policy 版本和完整 Trace。用 oracle 替换法定位：给正确证据看生成、给正确 Tool 看策略；最终以业务验收和安全不变量为最高层。线上用 shadow/canary 与失败 Trace 回流。
- **严格追问**：RAG 指标高为何任务仍失败？模型未忠实使用、Tool/决策错误或验收错；最终成功为何组件错也可以？下游补救掩盖风险；LLM-as-Judge？需 rubric、校准、人工抽检和防同源偏差。
- **项目映射**：ESGQA 适合映射检索链，agentHub 适合映射事件 Trace；当前没有联合评测运行证据，只能给实施方案。
- **评分/危险答案**：只用“最终答案看起来不错”不能发现危险工具尝试和错误证据。


