---
tags: [phase-1, sources, papers, specifications, official-docs]
created: 2026-07-22
updated: 2026-07-23
status: complete
verified_on: 2026-07-23
---

# Phase 1 技术来源登记

> [!note] 来源规则
> 论文用于支持其明确研究问题与实验结论；官方文档用于支持当前协议/API 语义；项目事实仍以 [[project-evidence-map]] 为准。课程中的架构归纳、面试框架和改造设计属于 [AI-Generated] 综合推导，不伪装成来源原话。Q85-Q114 新增的 RAG、检索和 LangChain/LangGraph 依据见 [[technical-source-register-interview-expansion]]。

## 1. 来源总表

| ID | 来源 | 类型 | 主要支持 | Lesson |
|---|---|---|---|---|
| `SRC-REACT-001` | [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) | ICLR 论文 | 推理轨迹与动作/观察交错的 ReAct 范式 | 1.1、1.2 |
| `SRC-REWOO-001` | [ReWOO: Decoupling Reasoning from Observations](https://arxiv.org/abs/2305.18323) | 论文 | Planner、Worker、Solver 解耦及减少重复推理调用 | 1.2 |
| `SRC-PLAN-SOLVE-001` | [Plan-and-Solve Prompting](https://arxiv.org/abs/2305.04091) | ACL 论文 | 先分解计划、再执行子任务的思想及 missing-step 问题 | 1.2 |
| `SRC-LLMCOMPILER-001` | [An LLM Compiler for Parallel Function Calling](https://arxiv.org/abs/2312.04511) | 论文 | 规划器、任务调度和并行执行的函数调用架构 | 1.2、1.3 |
| `SRC-TOOLFORMER-001` | [Toolformer](https://arxiv.org/abs/2302.04761) | NeurIPS 论文 | 模型学习何时调用 API、传什么参数、如何消费结果 | 1.3 |
| `SRC-SWE-AGENT-001` | [SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering](https://arxiv.org/abs/2405.15793) | 论文 | Agent-Computer Interface 对工具可用性与行为的影响 | 1.3、1.7 |
| `SRC-LANGGRAPH-001` | [LangGraph Graph API overview](https://docs.langchain.com/oss/python/langgraph/graph-api) | 官方文档 | State、Nodes、Edges 与条件图工作流 | 1.1、1.2、1.4 |
| `SRC-ANTHROPIC-TOOLS-001` | [Claude Tool Use Overview](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview) | 官方文档 | `input_schema`、`tool_use`、客户端执行、`tool_result` 回传与 strict tool use | 1.3、1.5 |
| `SRC-OPENAI-FC-001` | [OpenAI Function Calling](https://developers.openai.com/api/docs/guides/function-calling) | 官方文档 | 工具定义、Schema、调用结果回传和结构化输出 | 1.3、1.5 |
| `SRC-OPENAI-AGENT-001` | [OpenAI Agents SDK: Agents](https://openai.github.io/openai-agents-python/agents/) | 官方文档/源码文档 | Agent、Runner、工具、guardrail、handoff、结构化输出和 Loop 所属边界 | 1.1、1.2、1.5 |
| `SRC-OPENAI-TOOLS-001` | [OpenAI Agents SDK: Tools](https://openai.github.io/openai-agents-python/tools/) | 官方文档/源码文档 | 托管工具、本地工具、函数工具、超时、错误与审批门 | 1.3、1.7 |
| `SRC-OPENAI-CONTEXT-001` | [OpenAI Agents SDK: Context management](https://openai.github.io/openai-agents-python/context/) | 官方文档 | 本地运行上下文与模型可见上下文的区别 | 1.6 |
| `SRC-OPENAI-USAGE-001` | [OpenAI Agents SDK: Usage](https://openai.github.io/openai-agents-python/usage/) | 官方文档 | 请求级/运行级 Token Usage 追踪 | 1.5、1.6 |
| `SRC-MCP-SPEC-001` | [Model Context Protocol Specification 2025-11-25](https://modelcontextprotocol.io/specification/2025-11-25) | 协议规范 | Host/Client/Server、tools/resources/prompts 与协商能力 | 1.3、1.5 |
| `SRC-MCP-SEC-001` | [MCP Security Best Practices](https://modelcontextprotocol.io/docs/tutorials/security/security_best_practices) | 官方安全指南 | 最小 Scope、SSRF、令牌代理、显式授权、沙箱和 TOCTOU 风险 | 1.3、1.7 |
| `SRC-PYDANTIC-001` | [Pydantic Models](https://pydantic.dev/docs/validation/latest/concepts/models/) | 官方文档 | BaseModel 验证、Schema、严格模式和 validation bypass 风险 | 1.1、1.3、1.4 |
| `SRC-LOST-MIDDLE-001` | [Lost in the Middle](https://aclanthology.org/2024.tacl-1.9/) | TACL 论文 | 长上下文中相关信息位置影响模型利用效果 | 1.6 |
| `SRC-OWASP-LLM-001` | [OWASP Top 10 for LLM Applications 2025](https://owasp.org/www-project-top-10-for-large-language-model-applications/assets/PDF/OWASP-Top-10-for-LLMs-v2025.pdf) | 安全社区规范 | Prompt Injection、Excessive Agency、敏感信息披露等风险分类 | 1.3、1.7 |
| `SRC-NIST-GENAI-001` | [NIST AI 600-1: Generative AI Profile](https://doi.org/10.6028/NIST.AI.600-1) | 政府标准指南 | Govern/Map/Measure/Manage 的生成式 AI 风险治理 | 1.7 |
| `SRC-DOCKER-SEC-001` | [Docker Engine Security](https://docs.docker.com/engine/security/) | 官方文档 | namespace、cgroup、daemon attack surface、capabilities | 1.7 |
| `SRC-DOCKER-SECCOMP-001` | [Docker Seccomp Profiles](https://docs.docker.com/engine/security/seccomp/) | 官方文档 | 默认 seccomp allowlist 与系统调用限制 | 1.7 |
| `SRC-DOCKER-RUN-001` | [docker container run](https://docs.docker.com/reference/cli/docker/container/run) | 官方参考 | `--read-only`、`no-new-privileges`、security-opt 等运行加固项 | 1.7 |
| `SRC-DOCKER-ROOTLESS-001` | [Docker Rootless Mode](https://docs.docker.com/engine/security/rootless/) | 官方文档 | daemon 与容器非 root 运行的风险降低机制 | 1.7 |

| `SRC-REFLEXION-001` | [Reflexion: Language Agents with Verbal Reinforcement Learning](https://arxiv.org/abs/2303.11366) | 论文 | 语言反馈写入 episodic memory，在后续 trial 中影响决策；不是参数更新 | 架构专题 |
| `SRC-SELF-REFINE-001` | [Self-Refine: Iterative Refinement with Self-Feedback](https://arxiv.org/abs/2303.17651) | 论文 | 同一模型生成、反馈、修订的迭代输出改进框架 | 架构专题 |
| `SRC-TOT-001` | [Tree of Thoughts](https://arxiv.org/abs/2305.10601) | 论文 | 生成、评估、搜索多条 thought 分支并支持回溯 | 架构专题 |
| `SRC-GOT-001` | [Graph of Thoughts](https://arxiv.org/abs/2308.09687) | 论文 | 将 thought 建模为可聚合、变换和反馈的任意依赖图 | 架构专题 |
| `SRC-LATS-001` | [Language Agent Tree Search](https://proceedings.mlr.press/v235/zhou24r.html) | ICML 论文 | 结合 MCTS、LM value、self-reflection 与环境反馈的 Agent 搜索 | 架构专题 |
| `SRC-CRITIC-001` | [CRITIC: Tool-Interactive Critiquing](https://arxiv.org/abs/2305.11738) | ICLR 论文 | 使用外部工具反馈检查并修订模型输出 | 架构专题 |
| `SRC-SELF-CORRECTION-LIMIT-001` | [Large Language Models Cannot Self-Correct Reasoning Yet](https://arxiv.org/abs/2310.01798) | 论文 | 无外部反馈的内在自纠错可能无效甚至退化，约束自反思的扩大结论 | 架构专题 |
| `SRC-CAMEL-001` | [CAMEL: Communicative Agents](https://arxiv.org/abs/2303.17760) | 论文 | 角色扮演式 communicative agents 与协作研究框架 | 架构专题 |
| `SRC-AUTOGEN-001` | [AutoGen: Multi-Agent Conversation](https://arxiv.org/abs/2308.08155) | 论文/框架 | 可配置 Agent 通过对话、工具和人工输入形成交互模式 | 架构专题 |

## 2. 每课最低来源集

### Lesson 1.1

- `SRC-REACT-001`：Agent 不只是语言生成，而是推理、行动、观察互相反馈的一种范式。
- `SRC-LANGGRAPH-001`：State/Nodes/Edges 是图工作流的工程构件。
- `SRC-OPENAI-AGENT-001`：现代 SDK 中 Agent 通常是模型配置加工具与运行时能力，而不是“模型本身”。
- **限制**：没有任何来源给出全行业唯一 Agent 定义；课程采用可操作的工程定义。

### Lesson 1.2

- `SRC-REACT-001`：交错式推理-行动。
- `SRC-REWOO-001`：计划与观察解耦。
- `SRC-PLAN-SOLVE-001`：先计划再解决。
- `SRC-LLMCOMPILER-001`：面向函数调用 DAG 的并行执行。
- `SRC-LANGGRAPH-001`：图状态机表达显式控制流。
- **限制**：论文基准结果不能直接外推为用户项目的延迟、成本或准确率。

### Lesson 1.3

- `SRC-ANTHROPIC-TOOLS-001`、`SRC-OPENAI-FC-001`：当前主要 API 的工具 Schema 与结果回传循环。
- `SRC-MCP-SPEC-001`：MCP 是连接 Host、Client、Server 的协议，不能与单次 Function Calling 混为一谈。
- `SRC-TOOLFORMER-001`：工具使用涉及“何时、调用哪个、参数是什么、如何消费结果”。
- `SRC-SWE-AGENT-001`：工具接口设计会影响 Agent 的可用性和行为。
- **限制**：API 文档会变化，课程避免绑定具体模型价格和过时版本字符串。

### Lesson 1.4

- `SRC-LANGGRAPH-001`：边与条件边承载控制流。
- `SRC-PYDANTIC-001`：结构化输出只保证可验证形状，不自动保证业务正确。
- `SRC-LLMCOMPILER-001`：计划和调度可被拆成独立组件。
- **限制**：课程中的“规则优先、模型补充、策略兜底”是 `[AI-Generated]` 工程归纳。

### Lesson 1.5

- `SRC-OPENAI-AGENT-001`、`SRC-OPENAI-TOOLS-001`：模型/Agent/工具的当前 SDK 边界。
- `SRC-ANTHROPIC-TOOLS-001`、`SRC-OPENAI-FC-001`：不同 Provider 的消息与工具语义并不完全相同。
- `SRC-MCP-SPEC-001`：Provider 抽象和工具协议属于不同维度。
- `SRC-OPENAI-USAGE-001`：Usage 是成本、预算和可观测性的标准输入之一。
- **限制**：本课不声称存在跨厂商完整统一标准；强调最小公分母与能力发现。

### Lesson 1.6

- `SRC-LOST-MIDDLE-001`：长上下文容量不等于均匀利用能力。
- `SRC-OPENAI-CONTEXT-001`：应用本地上下文与模型可见上下文是不同层。
- `SRC-OPENAI-USAGE-001`：Usage 可作为实际 Token 计数来源。
- `SRC-MCP-SPEC-001`：工具定义、资源和结果也会竞争上下文预算。
- **限制**：字符估算、阈值和压缩策略是实现选择，必须用自己的数据校准。

### Lesson 1.7

- `SRC-OWASP-LLM-001`：LLM 应用风险分类。
- `SRC-NIST-GENAI-001`：风险治理贯穿生命周期，不止是输入过滤。
- `SRC-MCP-SEC-001`：工具连接中的授权、Scope、SSRF、令牌和沙箱风险。
- `SRC-DOCKER-SEC-001`、`SRC-DOCKER-SECCOMP-001`、`SRC-DOCKER-RUN-001`、`SRC-DOCKER-ROOTLESS-001`：容器隔离与加固的不同层次。
- `SRC-SWE-AGENT-001`：Agent-Computer Interface 是安全和可控性的设计面。
- **限制**：容器不是虚拟机级自动安全保证；安全结论必须结合威胁模型、宿主机和挂载配置。

### 通用 Agent 架构模式专题

- `SRC-REACT-001`、`SRC-REWOO-001`、`SRC-PLAN-SOLVE-001`、`SRC-LLMCOMPILER-001`：行动反馈、规划解耦与 DAG 调度基础。
- `SRC-REFLEXION-001`、`SRC-SELF-REFINE-001`：跨 trial 语言记忆与单次输出迭代修订并非同一机制。
- `SRC-CRITIC-001`、`SRC-SELF-CORRECTION-LIMIT-001`：外部可验证反馈通常比无依据自评更可靠；自纠错不能假设必然提升。
- `SRC-TOT-001`、`SRC-GOT-001`、`SRC-LATS-001`：树/图搜索、候选评估、回溯、MCTS 和环境反馈的不同组合。
- `SRC-CAMEL-001`、`SRC-AUTOGEN-001`：多 Agent 对话与角色协作的研究/框架入口。
- **限制**：论文中的任务、模型、Prompt、搜索预算和指标各不相同；课程只归纳机制与工程权衡，不横向拼接数字，也不把论文算法归属于两个项目。

## 3. 不能从来源推出的结论

1. ReAct 在论文数据集有效，不代表它适合所有企业流程；可预测的业务流程通常优先确定性 Workflow。
2. Function Calling 或 strict Schema 能降低格式错误，但不能验证参数真实性、权限或副作用安全。
3. 长上下文模型宣称支持更大窗口，不代表中间证据可稳定利用，也不代表应该把所有历史直接塞入。
4. Docker 提供隔离原语，但错误挂载、Docker socket、root、网络和 capability 配置仍可能扩大攻击面。
5. OWASP/NIST 给出风险类别和治理建议，不替代具体系统的代码审计、红队、权限测试和事故响应。
6. 官方 SDK 的现有能力不等于 ESGQA 或 agentHub 已经采用；项目事实只看 [[project-evidence-map]]。
7. Reflexion、Self-Refine 或 CRITIC 在论文任务上的改进，不代表无外部验证的自我批评会普遍提高正确率。
8. ToT、GoT、LATS 和 Multi-Agent 增加搜索或协作能力，也会增加模型调用、评估误差、状态合并和安全面；不能据架构名称断言优于简单 Workflow。

## 4. 更新策略

- 再次生成课程或准备面试前，复核所有 API/协议文档的版本与弃用信息。
- 论文结论保留论文任务、数据集和适用边界，不只记“提升了多少”。
- MCP 当前课程基线使用 2025-11-25 正式规范；候选版本、路线图和未来日期内容不作为已生效规范。
- OpenAI/Anthropic 示例只用于比较协议语义，不把模型名、价格或上下文长度写成稳定事实。
- 新增来源时必须给出 ID、核验日期、支持主张、限制和消费 Lesson。



