---
tags: [phase-1, interview, deduplication, coverage-map, provenance]
created: 2026-07-22
updated: 2026-07-23
status: complete
existing_questions: 84
new_questions: 30
final_questions: 114
---

# 公开面经题与 Phase 1 题库去重映射

## 1. 判定规则

| 判定 | 含义 | 处理 |
|---|---|---|
| `exact` | 现有题已经考查同一核心能力，换公司或场景不改变答案骨架 | 不新增编号；给现有题补来源和详细答案 |
| `partial` | 现有题覆盖一部分，但面经增加了独立、可追问的工程维度 | 新增题，且标明与旧题的关系 |
| `gap` | Q1-Q84 没有可独立训练的答案入口 | 新增 Q85+ |
| `later` | 真实相关，但属于模型训练、RAG 深水区或传统后端后续阶段 | 来源登记保留，本阶段不展开 |

> [!important] 去重不等于删掉场景价值
> 例如“工具失败怎么办”已有 Q12，但“跨境汇款超时后如何确保资金安全”包含不可逆副作用、状态核验和人工处置，保留为独立场景题。相反，“Agent 和 Workflow 的区别”与 Q2 完全同构，只补来源，不再造一个重复编号。

## 2. 已被 Q1-Q84 完整覆盖的公开面经题

| 规范化题型 | 来源 | 现有题 | 判定与说明 |
|---|---|---|---|
| 什么是 Agent，与普通模型调用有何不同？ | `INT-NC-A03`、`INT-NC-B02` | Q1 | `exact`：目标、状态、动作、观察与终止闭环 |
| Agent 和固定 Workflow 有什么区别？ | `INT-NC-A05` | Q2 | `exact`：确定性控制流与动态决策，补混合架构 |
| Agent 的核心运行流程是什么？ | `INT-NC-B01/B03` | Q9 | `exact`：observe/decide/validate/authorize/act/update/terminate |
| ReAct 的架构和工作原理是什么？ | `INT-NC-A01` | Q65-Q68 | `exact`：策略范式、生产化与不适用场景 |
| 什么是 Plan 模式，何时重规划？ | `INT-NC-B03` | Q10、Q15、Q69 | `exact`：计划、校验、执行、漂移与重规划 |
| Agent 如何判断信息足够并最终输出？ | `INT-NC-B01` | Q5 | `exact`：业务验收、终态原因、信息充分性与安全停止 |
| Agent 陷入死循环怎么办？ | `INT-NC-B01` | Q5、Q11 | `exact`：步数/Token/deadline、重复状态和 progress detector |
| Tool 调用的基本流程是什么？ | `INT-NC-B03` | Q17-Q20 | `exact`：契约、参数校验、执行、ToolResult 回注 |
| Function Calling、Tool Registry、MCP 有什么区别？ | `INT-CS-C01`、`INT-DY-C01` | Q20 | `exact`：模型动作协议、运行时目录、跨进程协议分层 |
| Tool 写操作超时能否直接重试？ | `INT-NC-B02` | Q21 | `exact`：未知结果、幂等键、状态核验和补偿 |
| Router 为什么不只是意图分类？ | `INT-NC-B02` | Q25-Q30 | `exact`：能力、策略、成本、风险和 fallback 决策 |
| Provider 如何抽象和做跨模型 fallback？ | `INT-CS-C02` | Q33-Q40 | `exact`：能力矩阵、统一事件、错误分类和不变量 |
| 上下文窗口受什么限制，过长怎么办？ | `INT-NC-B02/B04` | Q41-Q46 | `exact`：预算、选择、压缩、检索与生命周期 |
| Agent Memory 如何设计？ | `INT-DY-C01`、`INT-NC-B02` | Q45-Q48 | `exact`：写入、检索、更新、过期、遗忘和污染治理 |
| Prompt Injection 为什么不能只靠 Prompt？ | `INT-CS-C01` | Q49-Q56 | `exact`：不可信数据与控制面隔离、权限与沙箱 |
| Agent 全链路如何观测和评测？ | `INT-NC-B01`、`INT-JG-C01` | Q59、Q62 | `exact`：Trace、组件指标、端到端指标与人工复核 |
| 单 Agent 与 Multi-Agent 如何选择？ | `INT-NC-B01` | Q79-Q80 | `exact`：能力/权限/上下文是否真需分离及增益验证 |
| 如何设计多 Agent 架构？ | `INT-NC-B02/B03` | Q79-Q80 | `exact`：Supervisor/Worker、typed artifact、成本与瓶颈 |
| 项目没有用 ReAct 怎么回答？ | 面经共同追问模式 | Q81-Q84 | `exact`：先讲标准机制，再解释项目选择与演进边界 |

## 3. 部分覆盖或缺失：新增 Q85-Q114

| 新题 | 规范化题目 | 主要来源 | 旧题关系 | 判定 |
|---|---|---|---|---|
| Q85 | 项目使用什么大模型，为什么这样选？ | `INT-NC-A01`、`INT-NC-B01/B02` | Q34/Q37 讲 Provider 能力，但没有完整模型选型答法 | `partial` |
| Q86 | 推理框架和部署架构如何选，部署遇到什么问题？ | `INT-NC-B01` | Q38 只验证 Provider 接通 | `gap` |
| Q87 | Embedding 模型和向量数据库如何选？ | `INT-NC-A01` | Q44 只说明长窗口仍需检索 | `gap` |
| Q88 | 从 Elasticsearch 切到向量检索，能力如何变化并怎样迁移？ | `INT-NC-A01` | Q57 有知识问答设计，没有迁移权衡 | `gap` |
| Q89 | Agent 选错工具、参数或顺序时如何系统排查？ | `INT-NC-A02` | Q12 偏工具执行失败，Q18 偏参数验证 | `partial` |
| Q90 | Tool Calling 效果差时，何时做 Prompt/Schema/Router，何时考虑 SFT/RL？ | `INT-NC-A02` | Q15 有校验修复，但没有训练与工程手段的升级门槛 | `partial` |
| Q91 | 模型怎样决定调用工具还是直接回答？ | `INT-NC-B02` | Q25 讲 Router，未单独训练 abstain/direct/tool 三分决策 | `partial` |
| Q92 | 如何限制 Tool 的可见性、次数、参数、权限和成本？ | `INT-NC-A01` | Q11/Q22 分散覆盖预算与权限 | `partial` |
| Q93 | Agent Prompt 怎样设计、版本化、测试和防止职责越界？ | `INT-NC-B02/B03` | 旧题只有零散 Prompt 安全，没有完整生命周期 | `gap` |
| Q94 | LangChain 与 LangGraph 有什么区别，如何选？ | `INT-NC-B02` | Q14/Q76 讲状态图价值，不直接比较框架职责 | `gap` |
| Q95 | 上下文压缩方案怎样设计，阈值和保留轮数如何确定？ | `INT-NC-A01/B04` | Q43/Q46 给原则，但缺指标化选阈方法 | `partial` |
| Q96 | Recall@K 如何计算、构造标注并证明数字可信？ | `INT-NC-A01` | Q62 是通用评测，没有检索指标答辩模板 | `gap` |
| Q97 | 评测集为什么要更新，如何防止数据泄漏和对评测集过拟合？ | `INT-NC-A01` | Q62 未展开集合治理 | `gap` |
| Q98 | RAG 知识库如何不停服更新并保证版本一致性？ | `INT-NC-B02` | Q57 未覆盖双索引、别名切换与回滚 | `gap` |
| Q99 | RAG 与直接让 LLM 回答有什么区别，何时反而不该用 RAG？ | `INT-NC-A05` | Q57 设计系统，缺少明确对比与反例 | `gap` |
| Q100 | RAG 有哪些检索策略，常见向量相似度如何选择？ | `INT-NC-B03` | Q44 只讲为何检索 | `gap` |
| Q101 | 为什么加 Reranker 后效果可能下降，如何定位？ | `INT-NC-B02` | ESGQA 有重排证据，但旧题无故障题 | `gap` |
| Q102 | 短期记忆何时转长期记忆，如何合并、淘汰和防污染？ | `INT-NC-B02` | Q45 是生命周期总纲，缺面试场景化算法 | `partial` |
| Q103 | 使用状态机时，模型如何感知当前状态而不控制真实状态？ | `INT-NC-B02` | Q4/Q14 分别讲状态与图，缺模型可见投影 | `partial` |
| Q104 | Multi-Agent 之间如何传输数据和共享上下文？ | `INT-NC-B01` | Q79 只简述 typed artifact | `partial` |
| Q105 | 多个 Agent 并发操作数据库或文件时如何保证一致性？ | `INT-NC-B01` | Q13/Q78 讲并行条件，缺共享资源并发控制 | `partial` |
| Q106 | Agent 失败、中断或恢复时，怎样做到重试安全？ | `INT-NC-B02` | Q12/Q21/Q29 分散覆盖，需要端到端恢复答法 | `partial` |
| Q107 | 跨境汇款等高风险业务中，Agent 超时/失败如何确保资金安全？ | `INT-NC-B02` | Q21/Q54 有原则，但场景包含账务状态机与人工处置 | `partial` |
| Q108 | 如何介绍 Agent 项目的目标、架构、团队和个人贡献？ | `INT-NC-A04/B01` | Q7/Q8 是指定项目证据，不是通用项目答辩结构 | `gap` |
| Q109 | 如何回答“项目最大挑战是什么”并证明不是套话？ | `INT-NC-A04` | Q63 讲未实现能力，未训练挑战陈述 | `gap` |
| Q110 | 项目没有正式用户或没有上线，应该怎样回答？ | `INT-NC-B02` | Q63 可兜边界，但缺验证层级和诚实表达 | `gap` |
| Q111 | 面试官质疑准确率、召回率、性能提升数字时如何自证？ | `INT-NC-A01/B02` | Q38/Q62 提验证，缺数字证据链模板 | `gap` |
| Q112 | 当前 Agent 落地最关键的挑战有哪些？ | `INT-NC-A03` | Q60/Q61 是排障与成本，缺系统化行业挑战总结 | `gap` |
| Q113 | Agent Skill、Tool、Function Calling 与 MCP 如何区分和组合？ | `INT-NC-B03` | Q20 未包含 Skill 的封装层 | `partial` |
| Q114 | RAG 评测与 Agent Trace 评测有什么不同，怎样联合评估？ | `INT-NC-B01/B04` | Q62 只给通用总纲 | `partial` |

## 4. 本阶段不展开但保留来源的题

| 题型 | 来源 | 原因 | 计划归属 |
|---|---|---|---|
| PPO、DPO、GRPO 原理、Loss 与 Reward 设计 | `INT-NC-A02/A03` | 需要强化学习与对齐训练前置 | 后续模型后训练阶段 |
| 用 SFT/GRPO 训练 Function Calling 的数据与奖励细节 | `INT-NC-A02` | Q90 只讲何时升级，不展开训练实现 | 后续 Agentic Training 阶段 |
| Transformer、Cross-Attention、Top-K/Top-P | `INT-NC-A03` | 属于大模型基础与生成机制 | 后续 LLM 基础阶段 |
| SDD、Spec-kit、OpenSpec、Harness Engineering 全套 | `INT-NC-A01` | 与 Agent 工程有关，但超出本阶段七课主线 | AI Coding/Agent Harness 专题 |
| DDD、MySQL、Redis、线程池、网络、算法手撕 | 多来源 | 真实岗位常与后端基础混合考，但不属于 Phase 1 Agent 专项 | 后端基础并行题库 |

## 5. 覆盖结论

- 原 Q1-Q84 已覆盖公开面经中最常见的 Agent 定义、Workflow、Loop、ReAct、Tool、Context、Memory、安全、评测和 Multi-Agent 主干。
- 真实缺口主要集中在三类：**项目真实性答辩**、**RAG/检索工程细节**、**生产事故与高风险场景**。
- 新增 Q85-Q114 后，总题数为 **114**；新增题不是“30 道官方原题”，而是根据 A/B 级面经问题规范化、去重后形成的训练入口。
- C 级材料只参与覆盖校验，不作为新增题的公司归因依据。
- `INT-NC-C01`、`INT-NC-C02` 仅用于校验项目深挖的追问组织和主题覆盖，不单独贡献新增题，也不作为美团或百度的公司真题归因。


