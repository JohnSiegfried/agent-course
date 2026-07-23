---
tags: [phase-1, interview, question-bank, real-interview-derived, standardized-answers]
created: 2026-07-23
updated: 2026-07-23
status: complete
question_count: 114
existing_generated_questions: 84
real_interview_derived_additions: 30
---

# Phase 1 Agent 面试题库：114 题扩展主索引

> [!important] 真实性口径
> Q1-Q84 是依据课程、技术资料和冻结项目源码生成的系统训练题，全部保留；Q85-Q114 是对 A/B 级公开面经问题去重、规范化后的新增训练题。二者都不是企业官方题库。具体来源、等级和限制见 [[real-interview-source-register]]，原题到题库的合并依据见 [[interview-source-to-question-map]]。

## 1. 使用入口

| 内容 | 作用 |
|---|---|
| [[interview-question-bank\|Q1-Q84 原精简题卡]] | 保留旧编号、30 秒提纲、评分点和危险答案 |
| [[interview-answers/01-agent-runtime-and-architecture\|Q1-Q16 详细答案]] | Agent 定义、Runtime、Loop、规划与双项目架构 |
| [[interview-answers/02-tools-routing-and-provider\|Q17-Q40 详细答案]] | Tool、Router、Provider |
| [[interview-answers/03-context-memory-and-rag\|Q41-Q48 详细答案]] | Context、Memory、压缩 |
| [[interview-answers/04-security-evaluation-and-system-design\|Q49-Q64 详细答案]] | 安全、评测、排障和系统设计 |
| [[interview-answers/05-advanced-agent-patterns\|Q65-Q84 详细答案]] | ReAct、Planning、Reflection、Search、Multi-Agent |
| [[interview-answers/06-real-interview-derived-expansion\|Q85-Q99 详细答案]] | 模型/部署、检索选型、Tool 排障、Prompt、LangGraph、指标 |
| [[interview-answers/07-real-interview-derived-expansion-q100-q114\|Q100-Q114 详细答案]] | 检索、记忆、多 Agent、事故恢复、项目真实性与联合评测 |
| [[technical-source-register-interview-expansion\|新增答案技术来源]] | RAG、Elastic、FAISS、BEIR、LangChain/LangGraph 一手依据 |

## 2. Q1-Q84 保留结构

| 范围 | 核心主题 | 面经重复度 | 详细答案 |
|---|---|---:|---|
| Q1-Q8 | Agent 定义、Workflow、Runtime、状态与双项目总览 | 高 | [[interview-answers/01-agent-runtime-and-architecture]] |
| Q9-Q16 | Loop、ReAct/Planning、预算、失败、并行、Planner | 高 | [[interview-answers/01-agent-runtime-and-architecture]] |
| Q17-Q24 | Tool 契约、参数、ToolResult、MCP、幂等、权限 | 高 | [[interview-answers/02-tools-routing-and-provider]] |
| Q25-Q32 | Router、规则/模型、校准、评测、fallback | 中高 | [[interview-answers/02-tools-routing-and-provider]] |
| Q33-Q40 | Provider、能力矩阵、流式、错误、fallback、接入验证 | 中 | [[interview-answers/02-tools-routing-and-provider]] |
| Q41-Q48 | Context 预算、选择、压缩、Memory 生命周期 | 高 | [[interview-answers/03-context-memory-and-rag]] |
| Q49-Q56 | Threat model、Prompt Injection、权限、沙箱、HITL | 中高 | [[interview-answers/04-security-evaluation-and-system-design]] |
| Q57-Q64 | 知识问答、代码 Agent、观测、排障、成本、评测、项目边界 | 高 | [[interview-answers/04-security-evaluation-and-system-design]] |
| Q65-Q84 | ReAct、ReWOO、Reflection、ToT/GoT/LATS、DAG、Multi-Agent | 中高 | [[interview-answers/05-advanced-agent-patterns]] |

> [!note] 原题修正
> 原 Q16 的精简卡曾出现答案与题目错位；正确答案已在 [[interview-answers/01-agent-runtime-and-architecture#Q16. ESGQA 图与 agentHub Runtime Loop 的根本差异？|Q16 详细卡]]中给出：ESGQA 是领域状态图编排，agentHub 是会话/Provider/事件/权限生命周期 Runtime，两者处于不同抽象层。

## 3. Q85-Q114 新增题索引

| 题号 | 规范化题目 | 公开面经来源 | 复习优先级 |
|---|---|---|---:|
| Q85 | 项目使用什么大模型，为什么这样选？ | `INT-NC-A01`、`INT-NC-B01/B02` | P0 |
| Q86 | 推理框架和部署架构如何选，部署遇到什么问题？ | `INT-NC-B01` | P1 |
| Q87 | Embedding 模型和向量数据库如何选？ | `INT-NC-A01` | P0 |
| Q88 | 从 Elasticsearch 切到向量检索，能力如何变化并怎样迁移？ | `INT-NC-A01` | P1 |
| Q89 | Agent 选错工具、参数或顺序时如何系统排查？ | `INT-NC-A02` | P0 |
| Q90 | Tool Calling 效果差时，何时做工程优化，何时考虑 SFT/RL？ | `INT-NC-A02` | P1 |
| Q91 | 模型怎样决定调用工具还是直接回答？ | `INT-NC-B02` | P0 |
| Q92 | 如何限制 Tool 的可见性、次数、参数、权限和成本？ | `INT-NC-A01` | P0 |
| Q93 | Agent Prompt 怎样设计、版本化、测试和防止职责越界？ | `INT-NC-B02/B03` | P0 |
| Q94 | LangChain 与 LangGraph 有什么区别，如何选？ | `INT-NC-B02` | P0 |
| Q95 | 上下文压缩方案怎样设计，阈值和保留轮数如何确定？ | `INT-NC-A01/B04` | P0 |
| Q96 | Recall@K 如何计算、构造标注并证明数字可信？ | `INT-NC-A01` | P0 |
| Q97 | 评测集为什么要更新，如何防泄漏和过拟合？ | `INT-NC-A01` | P0 |
| Q98 | RAG 知识库如何不停服更新并保证版本一致性？ | `INT-NC-B02` | P0 |
| Q99 | RAG 与直接让 LLM 回答有何区别，何时不该用？ | `INT-NC-A05` | P0 |
| Q100 | RAG 检索策略和向量相似度如何选择？ | `INT-NC-B03` | P0 |
| Q101 | 为什么加 Reranker 后效果可能下降，如何定位？ | `INT-NC-B02` | P0 |
| Q102 | 短期记忆何时转长期记忆，如何合并、淘汰和防污染？ | `INT-NC-B02` | P0 |
| Q103 | 模型如何感知状态机当前状态而不控制真实状态？ | `INT-NC-B02` | P1 |
| Q104 | Multi-Agent 之间如何传输数据和共享上下文？ | `INT-NC-B01` | P0 |
| Q105 | 多个 Agent 并发操作数据库或文件时如何保证一致性？ | `INT-NC-B01` | P0 |
| Q106 | Agent 失败、中断或恢复时怎样做到重试安全？ | `INT-NC-B02` | P0 |
| Q107 | 跨境汇款等高风险业务中，Agent 超时/失败如何保证资金安全？ | `INT-NC-B02` | P0 |
| Q108 | 如何介绍 Agent 项目的目标、架构、团队和个人贡献？ | `INT-NC-A04/B01` | P0 |
| Q109 | 如何回答“项目最大挑战是什么”并证明不是套话？ | `INT-NC-A04` | P0 |
| Q110 | 项目没有正式用户或没有上线，应该怎样回答？ | `INT-NC-B02` | P0 |
| Q111 | 面试官质疑准确率、召回率、性能提升数字时如何自证？ | `INT-NC-A01/B02` | P0 |
| Q112 | 当前 Agent 落地最关键的挑战有哪些？ | `INT-NC-A03` | P0 |
| Q113 | Agent Skill、Tool、Function Calling 与 MCP 如何区分和组合？ | `INT-NC-B03` | P1 |
| Q114 | RAG 评测与 Agent Trace 评测有何不同，怎样联合评估？ | `INT-NC-B01/B04` | P0 |

## 4. 严格复习顺序

1. **第一轮只练 P0 的 30 秒答案**：先做到每题结论准确、没有项目夸大。
2. **第二轮练 2 分钟答案**：每题必须包含机制、失败路径、指标和边界。
3. **第三轮做项目映射**：ESGQA 与 agentHub 各选一个源码证据；未实现能力必须说“优化建议”。
4. **第四轮接受反例追问**：模型调错 Tool、压缩丢约束、Reranker 变差、写操作超时、没有上线和数字被质疑。
5. **第五轮按整场面试串联**：项目介绍 Q108 -> 架构 Q2/Q9 -> Tool Q89/Q92 -> Context Q95/Q102 -> 评测 Q96/Q111/Q114 -> 安全 Q107。

## 5. 统一回答合格线

- **事实层**：知道来源等级；不把公开复盘说成官方真题。
- **技术层**：先定义，再讲控制流、失败、指标和权衡。
- **项目层**：引用 evidence ID；区分源码、运行、文档、未实现和建议。
- **数据层**：任何数字都能给公式、数据、版本、环境和原始记录；否则撤回数字。
- **面试层**：主动给一个反例和一个残余风险，不等面试官替你指出。


