---
tags: [phase-1, agent-architectures, react, planning, reflection, search, multi-agent, interview]
created: 2026-07-22
updated: 2026-07-22
status: complete
role: required-interview-supplement
prerequisites: ["[[lessons/1.1-agent-definition-and-first-principles]]", "[[lessons/1.2-agent-loop-patterns]]"]
evidence: [P1-ESG-STATEGRAPH-001, P1-ESG-ROUTING-001, P1-HUB-RUNTIME-001, P1-HUB-PLANNER-001, P1-HUB-ROUTING-001]
sources: [SRC-REACT-001, SRC-PLAN-SOLVE-001, SRC-REWOO-001, SRC-LLMCOMPILER-001, SRC-REFLEXION-001, SRC-SELF-REFINE-001, SRC-CRITIC-001, SRC-SELF-CORRECTION-LIMIT-001, SRC-TOT-001, SRC-GOT-001, SRC-LATS-001, SRC-CAMEL-001, SRC-AUTOGEN-001]
---

# 通用 Agent 架构模式：面试完整专题

> [!important] 两条证据线
> 本专题是面试必修的通用架构知识，不以 ESGQA 或 agentHub 是否实现为准。论文说明一种方法提出了什么机制；项目证据只说明冻结代码实现了什么。二者可以对照，不能互相冒充。

## 1. 为什么项目没用，面试仍必须会

面试官问 ReAct、Planning、Reflection 或 Tree Search，通常不是检查你是否复制过某个 Prompt，而是在检查五种能力：

1. 能否把自然语言方法还原成状态机、接口和控制权；
2. 能否比较适应性、调用成本、并行度、可审计和失败恢复；
3. 能否把论文原型改造成带权限、预算、终止和可观测性的 Runtime；
4. 能否根据任务选择简单 Workflow 或复杂搜索，而不是追热点；
5. 能否诚实区分通用知识、框架能力、项目实现和未来建议。

因此回答应分两段：先完整讲通用模式，再说项目映射。例如：

> ReAct 是推理/决策与 Action/Observation 交错的策略模式；生产实现要由 Runtime 约束工具、步数、预算和权限。我的 ESGQA 使用显式 StateGraph，不是通用 ReAct；agentHub 有事件 Runtime 和 Planner/Manager，也不能直接等同 ReAct。若引入，我会把它限制在开放探索子图中。

## 2. 先统一术语：架构、策略、Prompt 与 Runtime

很多面试回答混乱，是因为把不同层叫成同一个架构。

| 层 | 负责什么 | 例子 |
|---|---|---|
| 推理/决策策略 | 下一步候选如何产生 | ReAct、Plan-and-Execute、ToT |
| 控制流 | 状态如何迁移、何时分支/循环 | while-loop、StateGraph、DAG、事件状态机 |
| Runtime | 执行、队列、取消、重试、终止 | Runner、AgentRuntime、workflow engine |
| 能力层 | 模型能请求哪些环境动作 | Tool Registry、MCP、API、沙箱命令 |
| 记忆/上下文 | 历史和反馈如何保存与选择 | scratchpad、episodic memory、ContextBus |
| 治理层 | 哪些动作允许、如何审计 | Policy、HITL、budget、trace、sandbox |

ReAct 原论文强调 reasoning 与 acting 交错；它不自动规定你的容器、Provider、权限系统或事件存储。StateGraph 是控制流表达；图中可以实现固定 Workflow，也可以包含 ReAct 节点。Multi-Agent 描述多个决策主体的协作；每个主体内部又可以使用 ReAct 或 Planning。

### 2.1 通用状态模型

所有模式都可以落到：

\[
s_{t+1} = T(s_t, a_t, o_{t+1}), \quad a_t = \pi(s_t, g, b, p)
\]

- \(g\)：目标与验收；
- \(s_t\)：当前状态、历史、计划、记忆和错误；
- \(a_t\)：候选动作；
- \(o_{t+1}\)：环境观察；
- \(b\)：步数、Token、时间、费用预算；
- \(p\)：权限与安全策略；
- \(T\)：由受信任 Runtime 实现的状态转移；
- \(\pi\)：可能由 LLM、规则、搜索器或组合实现的策略。

比较任何架构时都问：谁实现 \(\pi\)，谁控制 \(T\)，谁验证 \(a_t\)，谁判断终止。

## 3. 对照基线：确定性 Workflow、StateGraph 与事件 Runtime

### 3.1 确定性 Workflow

控制流由代码预定义，模型只在局部节点完成分类、抽取或生成。

```text
authenticate -> classify -> retrieve -> rerank -> generate -> verify -> respond
```

优点：可测试、可审计、延迟上界清楚、权限容易收紧。缺点：开放任务和未知观察需要不断加分支。适合金融审批、固定 RAG、报告流水线和高风险业务。

### 3.2 StateGraph

StateGraph 把状态、节点、边和条件分支显式化。它不是某种 Agent 策略；同一张图可承载固定 Workflow、局部 ReAct 循环、人工审批与恢复节点。

### 3.3 事件驱动 Runtime

长生命周期任务由 Provider、Tool、用户、队列和定时事件驱动。Runtime 维护 `idle/running/waiting_approval/compressing/failed/completed` 等状态。它适合流式输出、外部进程和并发会话，但需要处理事件顺序、迟到事件、背压和唯一终态。

### 3.4 为什么先讲基线

复杂 Agent 模式不是默认答案。若任务步骤可枚举且错误代价高，确定性外壳通常更优。面试中的高级判断不是总选 ReAct，而是能解释何时**不选**。

## 4. ReAct：Reasoning 与 Acting 交错

来源：`SRC-REACT-001`，[ReAct](https://arxiv.org/abs/2210.03629)。

### 4.1 解决什么问题

纯 Chain-of-Thought 只能在模型已有上下文中推演，无法主动获取新事实；纯 Action policy 又可能缺少任务分解与错误修正。ReAct 让模型在推理/决策轨迹与环境动作之间交错：最新 Observation 会改变下一步判断。

经典概念轨迹：

```text
Question/Goal
Thought: 当前缺哪条信息
Action: search(query)
Observation: 搜索结果
Thought: 结果冲突，需要核验另一个来源
Action: fetch(document_id)
Observation: 文档内容
Thought: 证据足够，可以回答
Final: 带来源的结论
```

生产系统不必保存或展示模型的完整私有推理文本。可观测轨迹应记录结构化 `decision_summary/reason_code/action/observation_ref/policy_result`，既支持审计，也避免把不可控思维文本当稳定 API。

### 4.2 最小状态机

```text
READY
  -> DECIDING
  -> FINAL_REQUESTED ----------------------> COMPLETED
  -> ACTION_REQUESTED
       -> VALIDATING -> DENIED ------------> STOPPED
       -> AUTHORIZING -> APPROVAL_REQUIRED -> WAITING
       -> EXECUTING -> OBSERVING ----------> DECIDING
       -> FAILED -> RETRYABLE? ------------> DECIDING / STOPPED
```

ReAct 的关键不是 `while True`，而是观察能进入下一次决策，且 Runtime 保留验证、授权、执行和终止控制。

### 4.3 未运行教学伪代码

> [!warning] 教学代码边界
> 下例未在两个项目中运行，只用于说明生产 ReAct 的控制权。

```python
def run_react(goal, provider, tools, policy, budget):
    state = RunState(goal=goal)
    while True:
        budget.check_before_decision()
        decision = provider.decide(state.model_view())
        decision = validate_decision_schema(decision)

        if decision.kind == 'final':
            return verify_and_finish(decision.answer, state)

        tool = tools.require(decision.tool_name)
        args = tool.validate(decision.arguments)
        effect = policy.authorize(state.actor, tool, args)
        if effect != 'allow':
            return stop_with_policy_reason(effect, state)

        result = execute_with_deadline(tool, args, state.deadline)
        state.append_observation(sanitize_for_model(result))
        state.check_progress_and_step_limit()
```

省略但生产必需：流式事件、幂等、HITL 恢复、Tool timeout、Provider retry、Context 压缩、敏感数据处理和持久 checkpoint。

### 4.4 ReAct 的优势

- 对未知 Observation 有较强适应性；
- 能通过查询、执行和反馈修正初始假设；
- 每步动作短，容易在局部插入规则与审批；
- 适合网页探索、开放检索、交互式排障和不完全信息环境。

### 4.5 典型失败

1. **无限循环**：反复调用同一工具或同义查询。
2. **上下文膨胀**：每步 Thought/Action/Observation 复制进历史。
3. **观察污染**：网页或 ToolResult 中的恶意指令影响下一步。
4. **工具误选**：名称相似、描述不足或候选过多。
5. **错误传播**：早期错误 Observation 被后续当事实。
6. **副作用重复**：超时后重新决策导致重复写。
7. **成本和延迟不可预测**：模型调用次数由轨迹决定。
8. **假终止**：模型输出 Final，但验收条件仍未满足。

### 4.6 生产化清单

- typed decision 联合类型：`Final | ToolRequest | Clarification`；
- Tool allowlist、Schema、Policy、HITL 与副作用分类；
- `max_steps/max_tokens/deadline/max_cost`；
- progress fingerprint 和重复动作检测；
- 结构化 Observation、来源、可信度与不可信内容边界；
- output reserve 与轨迹压缩；
- Tool call ID、幂等键和未知提交状态；
- terminal reason 与可恢复 checkpoint；
- 每步 trace、延迟、Usage、错误类和业务进展。

### 4.7 面试表达

**30 秒**：ReAct 把推理/决策与动作/观察交错，使 Agent 能根据环境反馈逐步修正。生产实现必须由 Runtime 校验动作、授权工具、限制预算并确定性终止，不能只是让模型无限输出 Thought/Action。

**2 分钟**：再画状态机，讲 typed action、ToolResult、Prompt Injection、重复副作用、Context 增长和 progress detector；最后说明项目是否真实采用。

**危险答案**：ReAct 就是 `while True`；有 Thought 就一定更准确；模型决定调用什么就应直接执行。

## 5. Planning 家族：Plan-and-Solve 与 Plan-and-Execute

来源：`SRC-PLAN-SOLVE-001`，[Plan-and-Solve](https://arxiv.org/abs/2305.04091)。

### 5.1 核心思想

先把目标分解为子任务与依赖，再执行。工程上的 Plan-and-Execute 通常包含：

```text
Goal -> Planner -> Plan Validator -> Scheduler/Executor
     -> Observation/Progress -> Continue | Replan | Abort -> Synthesizer
```

计划不是自然语言列表就结束。最小 Plan Schema：

```text
Plan {
  goal, assumptions, version,
  steps[{id, objective, dependencies, required_capability,
         expected_output, acceptance, risk}],
  final_acceptance
}
```

### 5.2 适用

- 长任务、依赖明确、可并行子任务；
- 需要进度、人工审查、任务分派和失败恢复；
- 软件开发、研究报告、多数据源分析。

### 5.3 计划漂移与重规划

早期 Planner 只基于不完整信息，常见错误是缺步骤、依赖错、工具不存在、输出无法满足后继节点。每步后比较 `expected vs observed`：

- 局部参数错误：修当前 step；
- 前提失效但目标不变：replan 剩余步骤，保留已验证产物；
- 目标或权限变化：重新确认或终止；
- 连续 replan 无进展：停止，不能无限规划。

### 5.4 与 ReAct 的差异和组合

ReAct 在每个 Observation 后动态决定下一步，适应性高但调用多；Plan-and-Execute 先形成全局结构，可审计并可并行，但可能计划过早。常用 Hybrid：外层计划与进度，单个开放 step 内运行有界 ReAct。

### 5.5 面试追问

- 如何验证 Plan？答：Schema、工具存在、依赖无环、输入可得、权限、预算和验收可执行。
- replan 会不会丢工作？答：产物版本化，只替换未完成子图，已验证结果通过引用保留。
- Planner 和 Executor 用同一模型吗？答：可以，但角色分离不等于模型必须不同；高风险计划可用独立规则/critic 校验。

## 6. ReWOO：规划与 Observation 解耦

来源：`SRC-REWOO-001`，[ReWOO](https://arxiv.org/abs/2305.18323)。

### 6.1 三个角色

- **Planner**：一次生成带变量依赖的执行蓝图；
- **Worker**：按计划调用工具并绑定结果变量；
- **Solver**：读取计划与变量结果，综合最终答案。

概念轨迹：

```text
Plan: 需要先找公司报告，再提取排放值，最后比较
E1 = Search[公司 2023 ESG 报告]
E2 = Extract[E1, 碳排放]
E3 = Search[公司 2022 ESG 报告]
E4 = Extract[E3, 碳排放]
Solve[E2, E4]
```

变量名只是概念表示；生产实现应使用 typed artifact ID、依赖图和 Schema，不能靠字符串替换。

### 6.2 为什么减少重复推理

Planner 在看到工具结果前生成蓝图，Worker 执行时不必每个 Observation 都再次请求 Planner。独立分支可并行，Solver 最后汇总。

### 6.3 代价

- 计划无法充分利用中间意外发现；
- 某变量失败可能阻塞大量后继节点；
- Planner 可能引用未定义变量、产生环或选择不存在的工具；
- Solver 可能忽略失败/冲突并强行合成。

### 6.4 生产化

把计划编译为 DAG；加载时检查变量定义唯一、依赖无环、工具能力存在；Worker 返回 typed artifact；失败按节点策略重试或触发局部 replan；Solver 只能消费成功且有权限的产物。

### 6.5 与 Plan-and-Execute

二者都先计划。ReWOO 的突出特征是以变量占位把推理计划与未来工具 Observation 解耦，并由 Worker/Solver 分工；不要把所有有 Planner 的系统都叫 ReWOO。

## 7. Reflection 家族：不要把四种反馈循环混为一谈

### 7.1 通用 Reflection

泛指对当前轨迹或输出进行评价，再决定修订、重试或改变策略。必须定义：反馈来自谁、依据什么、作用在哪个时间尺度、保存多久。

### 7.2 Reflexion

来源：`SRC-REFLEXION-001`，[Reflexion](https://arxiv.org/abs/2303.11366)。

核心是从任务反馈产生语言反思，把反思放入 episodic memory，影响**后续 trial**；不通过梯度更新模型权重。

```text
Trial 1: actor trajectory -> evaluator feedback -> failure
Reflector: 归纳失败原因与下一次策略
Memory: 保存反思
Trial 2: actor(goal + selected reflections) -> new trajectory
```

风险：反思本身可能错误；失败归因可能把环境噪声写成长期规则；memory 污染和过期；trial 成本高。需要外部结果信号、反思 Schema、来源、TTL、冲突处理和验证后晋升。

### 7.3 Self-Refine

来源：`SRC-SELF-REFINE-001`，[Self-Refine](https://arxiv.org/abs/2303.17651)。

同一模型可承担 generator、feedback provider、refiner，对**同一输出**进行迭代：

```text
draft -> feedback(criteria, draft) -> revised draft
      -> stop criteria met or max rounds
```

它不等于跨 trial episodic memory，也不必与环境行动交错。适合可读性、格式、覆盖度等有明确 rubric 的生成任务。若模型缺少事实，反复自评可能只是更自信地重写错误。

### 7.4 CRITIC

来源：`SRC-CRITIC-001`，[CRITIC](https://arxiv.org/abs/2305.11738)。

关键差异是让模型通过搜索、代码执行等外部工具取得可验证反馈，再修订输出。它比纯内在自评多了环境证据，但 Tool 结果仍需权限、来源与注入防护。

### 7.5 自纠错局限

来源：`SRC-SELF-CORRECTION-LIMIT-001`，[Large Language Models Cannot Self-Correct Reasoning Yet](https://arxiv.org/abs/2310.01798)。

课程结论不是自反思无用，而是**无外部反馈、无独立标准时不能假设一定提升**。面试中应主动提出：

- 外部 verifier、测试、检索证据或人类反馈；
- critic 与 generator 的错误相关性；
- max rounds 和 improvement threshold；
- 防止正确答案被错误 critic 改坏；
- 保存 draft、feedback、revision 和最终选择依据。

### 7.6 对比表

| 模式 | 反馈来源 | 时间尺度 | 记忆 | 主要用途 |
|---|---|---|---|---|
| Reflection | 任意 | step/trial | 可选 | 泛化概念 |
| Reflexion | 任务结果 + 语言反思 | 跨 trial | episodic | 从失败经验调整下一次行为 |
| Self-Refine | 通常同一 LLM 自反馈 | 同一输出多轮 | draft history | 修订生成结果 |
| CRITIC | 外部工具验证 | 同一任务多轮 | tool evidence | 事实、代码等可验证修订 |

## 8. Search 家族：ToT、GoT 与 LATS

### 8.1 为什么搜索

单轨 ReAct 或 CoT 一旦早期选择错误，后续通常沿错误路径继续。搜索式方法维护多个候选状态，使用 evaluator/value 选择、剪枝、回溯或合并。代价近似随分支数和深度快速增长。

### 8.2 Tree of Thoughts

来源：`SRC-TOT-001`，[Tree of Thoughts](https://arxiv.org/abs/2305.10601)。

组件：

1. thought decomposition：定义一个可评估的中间状态；
2. generator：从当前状态生成多个候选；
3. evaluator：打分或投票；
4. search：BFS/DFS/beam 等；
5. terminal verifier：判断最终解。

```text
root
 |- thought A -> A1 -> dead end
 |             -> A2 -> candidate
 |- thought B -> B1 -> best candidate
 |- thought C -> pruned
```

核心风险不是不会生成分支，而是 evaluator 不可靠。若生成器和评价器同源且共享偏差，搜索会扩大错误确信。生产中要限制 branching/depth/token，缓存等价状态，用外部 verifier 评估可验证任务。

### 8.3 Graph of Thoughts

来源：`SRC-GOT-001`，[Graph of Thoughts](https://arxiv.org/abs/2308.09687)。

GoT 允许 thought 形成任意图，可拆分、合并、聚合、精炼和反馈，而非树中每个节点只有一个父节点。适合多个候选部分结果需要合并的任务。

工程要求：typed node/edge、DAG 或显式循环策略、版本、provenance、合并冲突、去重和终止。不要把任何 LangGraph 应用都叫 Graph of Thoughts：一个是推理候选图范式，一个是通用控制流框架。

### 8.4 LATS

来源：`SRC-LATS-001`，[Language Agent Tree Search](https://proceedings.mlr.press/v235/zhou24r.html)。

LATS 将推理、行动、环境反馈、LM value 和 self-reflection 放入类似 MCTS 的搜索循环：选择候选节点、扩展动作、与环境交互、评估结果、把价值反馈到祖先，再选择后续路径。

生产挑战：环境动作可能不可逆，不能像棋盘模拟一样随意展开；需要可克隆 sandbox 或只读/模拟动作；value model 可能偏；搜索成本高；树中敏感 Observation 和凭证需要隔离。

### 8.5 搜索模式选择

- 有明确 verifier、状态可复制、错误分支可回退：适合 ToT/LATS。
- 候选结果需要合并：考虑 GoT/DAG。
- 副作用不可逆、环境昂贵、评价模糊：优先 Plan/Workflow，谨慎搜索。
- 简单任务：多分支搜索常得不偿失。

## 9. 并行与编译式执行：DAG、LLMCompiler

来源：`SRC-LLMCOMPILER-001`，[LLMCompiler](https://arxiv.org/abs/2312.04511)。

### 9.1 核心

Planner 把任务编译成带依赖的函数调用 DAG；scheduler 运行 ready nodes；joiner 根据结果决定完成或继续。相比串行 ReAct，独立工具可并行。

\[
T_{parallel} \ge \max_{path \in DAG} \sum_{v \in path} latency(v)
\]

关键路径决定理论下界，但 Provider/Tool 限流、资源竞争、重试和结果合并会降低收益。

### 9.2 安全并行条件

- 无数据依赖；
- 不争用同一可变资源，或有明确事务/锁；
- 副作用可交换、幂等或隔离；
- 各节点有独立 deadline 和取消传播；
- joiner 能表达 partial failure，而不是只 `Promise.all`。

### 9.3 失败处理

`fail-fast` 适合核心依赖；`best-effort` 适合多个可选数据源；`quorum` 适合多证据/多 Provider；补偿适合已完成的副作用节点。失败策略应写在 DAG 节点/边契约中。

## 10. Multi-Agent 基础编排

来源：`SRC-CAMEL-001`、`SRC-AUTOGEN-001`。

Multi-Agent 的价值不是让多个相同模型互相聊天，而是拆分能力、权限、上下文或并行责任。

### 10.1 Supervisor / Worker

Supervisor 分解、分派、检查并决定下一轮；Worker 拥有专业工具或独立上下文。风险是 Supervisor 成为瓶颈和单点错误。

### 10.2 Router / Specialist

Router 根据任务选择一个或多个 Specialist。适合领域边界清楚的客服、代码、安全、数据分析。风险是误路由和跨 Specialist 合并。

### 10.3 Generator / Critic / Verifier

一个 Agent 生成，另一个按 rubric 批评，确定性 verifier 或人类做最终门。不同角色不等于独立错误：若使用同模型与同上下文，偏差高度相关。

### 10.4 Debate

多个候选提出不同方案，再由 judge 选择。适合答案空间开放且可比较的任务；成本高，judge 可能被表达风格而非事实影响。

### 10.5 Blackboard

Agent 通过共享结构化工作区交换 artifact、decision、issue，而非复制整段聊天。需要条目 owner、version、source、status、权限、冲突和 GC。

### 10.6 Multi-Agent 常见失败

- 对话循环和角色漂移；
- 重复工作、上下文复制和 Token 爆炸；
- 责任不清，错误在 Agent 间转述后失去来源；
- 高权限 Agent 信任低权限 Agent 的恶意内容；
- 最终合并器忽略冲突；
- Agent 数量增加但能力、数据和工具没有差异。

面试判断：先证明单 Agent + Tools 不够，再引入 Multi-Agent。能用函数或并行任务表达的，不必人格化为 Agent。

## 11. Hybrid：生产系统最常见的答案

论文模式适合隔离机制，生产系统通常组合：

### 11.1 Workflow 外壳 + 局部 ReAct

```text
auth -> classify -> fixed retrieval
     -> [bounded ReAct for open evidence gap]
     -> deterministic verify -> answer
```

开放探索有适应性，入口、输出和高风险动作仍可控。

### 11.2 Plan + ReAct Workers

Planner 生成高层任务；每个开放 step 使用有界 ReAct；Manager 根据验收决定 continue/replan/abort。

### 11.3 Plan + DAG + Reflection

稳定子任务并行执行；失败节点使用外部反馈生成反思，仅修订剩余计划。防止每个成功节点都无意义反思。

### 11.4 Router + Specialist + Deterministic Policy

模型 Router 选择专业 Agent，Policy Engine 独立决定工具和数据权限，Runtime 统一预算与 trace。

### 11.5 Search in Sandbox

ToT/LATS 只在无副作用的推理空间或可克隆 sandbox 内展开；候选通过 verifier 后，真实副作用只提交一次。

## 12. 统一选型矩阵

| 模式 | 动态观察 | 全局计划 | 并行 | 调用成本 | 可审计 | 主要风险 | 典型任务 |
|---|---:|---:|---:|---:|---:|---|---|
| 固定 Workflow | 低 | 代码定义 | 中高 | 低 | 高 | 长尾僵化 | 固定 RAG、审批流程 |
| ReAct | 高 | 弱/滚动 | 低 | 中高且波动 | 中 | 循环、注入、成本 | 网页探索、交互排障 |
| Plan-and-Execute | 中 | 强 | 中高 | 中 | 高 | 计划漂移 | 长任务、研究、开发 |
| ReWOO | 中低 | 强且预先 | 高 | 较低 | 高 | 中途适应弱 | 多工具信息汇总 |
| Reflexion | 跨 trial | 可选 | 低 | 高 | 中 | 错误反思入记忆 | 可重复尝试任务 |
| Self-Refine | 输出内 | 无 | 低 | 中 | 中 | 自评偏差 | 文本/结构修订 |
| CRITIC | 中 | 无/弱 | 低 | 中高 | 高 | 工具反馈污染 | 代码、事实核验 |
| ToT | 候选内 | 搜索隐式 | 中 | 高 | 高 | evaluator 偏差 | 可回溯推理 |
| GoT | 图内 | 图结构 | 高 | 高 | 高 | 合并/循环复杂 | 可拆分合并问题 |
| LATS | 高 | 搜索规划 | 低到中 | 很高 | 高 | value/环境成本 | 可模拟交互任务 |
| Multi-Agent | 取决于内部 | 可选 | 高 | 高 | 中 | 协调和权限扩大 | 能力/上下文确需分离 |

### 12.1 决策问题

1. 步骤是否可枚举？是则先 Workflow。
2. 中间观察会否改变下一步？是则考虑 ReAct/replan。
3. 子任务依赖是否明确且可并行？是则 Plan/DAG/ReWOO。
4. 是否有可信 evaluator/verifier？没有则谨慎 Reflection/Search。
5. 环境能否回滚或模拟？不能则限制 ToT/LATS 的真实动作。
6. 多主体是否真的有不同能力、权限或上下文？没有则不必 Multi-Agent。
7. 错误代价、预算和延迟上限是什么？高风险场景加强确定性外壳。

## 13. 项目源码映射：相似不等于实现

### 13.1 ESGQA

证据：`P1-ESG-STATEGRAPH-001`、`P1-ESG-ROUTING-001`。

| 模式 | 映射结论 | 依据 |
|---|---|---|
| Workflow/StateGraph | **已实现** | `CBAMState`、节点、显式边和条件路由 |
| 局部模型决策 | **已实现** | 意图解析和结构化领域输出 |
| ReAct | **未实现** | 没有通用 Thought/Action/Observation 自由工具闭环 |
| Plan/ReWOO | **未实现** | 没有通用 Plan/DAG/变量 Worker-Solver 协议 |
| Reflection/Search | **未实现** | 没有 Reflexion、Self-Refine、ToT、LATS 控制链 |
| Multi-Agent | **未实现/不应声称** | 当前证据是领域图，不是角色协作 Runtime |

合理建议：对开放检索缺口设置局部有界 ReAct；对报告生成使用显式 Plan；外层保留 ESGQA 图、RAG 证据门和 API 安全。建议不是当前成果。

### 13.2 agentHub

证据：`P1-HUB-RUNTIME-001`、`P1-HUB-PLANNER-001`、`P1-HUB-ROUTING-001`。

| 模式 | 映射结论 | 依据 |
|---|---|---|
| 事件 Runtime | **已实现** | Provider 事件、队列、权限、压缩和 idle 生命周期 |
| Planning 管理 | **局部实现** | Planner 校验/有界重试，Manager continue/replan/abort |
| 路由/多角色基础 | **局部实现** | 事件规则和目标 Agent 路由 |
| ReAct | **不能直接认定** | 事件循环与工具事件不等于论文 ReAct 策略 |
| ReWOO | **未见完整实现** | 未见变量计划、Worker 执行、Solver 汇总闭环 |
| Reflexion/Self-Refine | **未见实现** | 压缩摘要不等于反思学习或输出迭代 |
| ToT/GoT/LATS | **未见实现** | 未见候选搜索、value、回溯或 MCTS |

合理建议：将策略抽为 `DecisionPolicy`，为 ReAct/Plan/Search 定义统一 typed event 和 contract suite；现有 Runtime 继续负责权限、Provider、Context 和终止。

## 14. 常见概念混淆

1. **CoT != ReAct**：CoT 只生成推理文本；ReAct 还与环境动作和观察交错。
2. **while-loop != ReAct**：循环只是控制结构，策略可能是规则、Planner 或 ReAct。
3. **StateGraph != GoT**：前者是工作流图框架；后者是 thought 生成/变换/聚合的推理范式。
4. **有 Planner != ReWOO**：还需变量化计划、Worker 与 Solver 解耦。
5. **摘要 != Reflection**：压缩上下文不等于从成败反馈提炼策略。
6. **重试 != Reflexion**：重做同一调用没有语言反馈记忆。
7. **两个角色 Prompt != 成熟 Multi-Agent**：还需通信、路由、状态、权限和终止协议。
8. **搜索更多 != 更正确**：evaluator 偏差可能放大，成本也随分支增长。
9. **论文方法 != 生产 Runtime**：权限、幂等、预算、取消、审计通常需要额外设计。
10. **框架支持 != 项目实现**：必须找到注册、状态、调用链和测试。

## 15. 大厂面试回答模板

### 15.1 被问某一种架构

```text
一句话目标
-> 核心角色与状态流
-> 与相邻模式的差异
-> 正常轨迹
-> 失败、成本与安全
-> 适用/不适用
-> 项目真实映射与边界
```

### 15.2 被问如何选型

先问任务可预测性、Observation 动态性、依赖 DAG、verifier、环境可回滚性、风险和预算。给出最简单可行基线，再说明何时升级为 ReAct、Plan、Reflection、Search 或 Multi-Agent。

### 15.3 被问项目为什么没用 ReAct

合格回答：

> ESGQA 的业务路径和合规边界较明确，因此用显式 StateGraph 获得可测试与可审计控制；这不是不知道 ReAct。对未来开放证据探索，我会只在受限子图引入有界 ReAct，并保留 Tool allowlist、步数/预算、证据校验和确定性出口。

### 15.4 被问最推荐哪种

不存在脱离任务的最佳架构。生产中常见答案是确定性 Workflow/Runtime 外壳，内部按局部不确定性选择 ReAct、Plan、DAG 或 verifier-backed Reflection。

## 16. 白板练习与文字 Demo

### Demo A：同一研究任务的四种实现

任务：比较两家公司最近两年的 ESG 指标。

- Workflow：固定检索四份报告 -> 抽取 -> 校验 -> 比较。
- ReAct：每次根据缺失证据决定搜索/提取下一步。
- Plan/ReWOO：先规划四个独立抽取，变量绑定后 Solver 汇总。
- ToT：生成多种指标解释路径，verifier 选择与原报告一致的路径。

比较模型调用、并行度、失败恢复和证据一致性。

### Demo B：Reflection 是否真的有用

第一次代码任务测试失败。纯 Self-Refine 只让模型阅读自己的代码；CRITIC 把真实编译/测试错误作为反馈；Reflexion 还把失败策略写入下一次 trial 的 episodic memory。让面试官看到三者不是同义词。

### Demo C：搜索不可用于真实删除

ToT/LATS 可以在候选计划或 sandbox 快照中搜索，但不能对真实文件系统展开多个删除分支。真实副作用只能在最终候选通过 Policy/HITL 后提交一次。

## 17. 复习卡与掌握检查

### 一句话卡

- ReAct：观察后再决定下一步。
- Plan-and-Execute：先全局拆解，再执行并重规划。
- ReWOO：变量计划把推理与未来 Observation 解耦。
- Reflexion：失败反馈形成语言记忆，影响后续 trial。
- Self-Refine：同一输出经历 feedback -> refine。
- CRITIC：用外部工具验证后修订。
- ToT：树上生成、评估、剪枝、回溯。
- GoT：thought 可形成、合并和变换为图。
- LATS：MCTS + LM value + reflection + 环境反馈。
- LLMCompiler/DAG：编译依赖并行工具任务。
- Multi-Agent：只有能力、权限或上下文确需分离时才值得。

### 闭卷任务

1. 画 ReAct 生产状态机，并标出模型不能控制的三处。
2. 用一个任务分别写出 ReAct、ReWOO 和 ToT 轨迹。
3. 解释 Reflexion、Self-Refine、CRITIC 的反馈来源与时间尺度。
4. 说出 ToT evaluator 错误为何会让搜索更差。
5. 分别解释两个项目与通用模式的已实现和未实现边界。

## 18. 关键来源与阅读顺序

基础行动/规划：

- `SRC-REACT-001`：[ReAct](https://arxiv.org/abs/2210.03629)
- `SRC-PLAN-SOLVE-001`：[Plan-and-Solve](https://arxiv.org/abs/2305.04091)
- `SRC-REWOO-001`：[ReWOO](https://arxiv.org/abs/2305.18323)
- `SRC-LLMCOMPILER-001`：[LLMCompiler](https://arxiv.org/abs/2312.04511)

反馈与反思：

- `SRC-REFLEXION-001`：[Reflexion](https://arxiv.org/abs/2303.11366)
- `SRC-SELF-REFINE-001`：[Self-Refine](https://arxiv.org/abs/2303.17651)
- `SRC-CRITIC-001`：[CRITIC](https://arxiv.org/abs/2305.11738)
- `SRC-SELF-CORRECTION-LIMIT-001`：[Self-Correction Limits](https://arxiv.org/abs/2310.01798)

搜索与协作：

- `SRC-TOT-001`：[Tree of Thoughts](https://arxiv.org/abs/2305.10601)
- `SRC-GOT-001`：[Graph of Thoughts](https://arxiv.org/abs/2308.09687)
- `SRC-LATS-001`：[Language Agent Tree Search](https://proceedings.mlr.press/v235/zhou24r.html)
- `SRC-CAMEL-001`：[CAMEL](https://arxiv.org/abs/2303.17760)
- `SRC-AUTOGEN-001`：[AutoGen](https://arxiv.org/abs/2308.08155)

来源边界见 [[technical-source-register]]；项目事实见 [[project-evidence-map]]；主课程见 [[lessons/1.2-agent-loop-patterns]]；专项问题见 [[interview-question-bank#10. 通用 Agent 架构专项题]]。

