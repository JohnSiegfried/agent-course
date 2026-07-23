---
tags: [phase-1, review, flashcards, self-check, interview]
created: 2026-07-22
updated: 2026-07-23
status: complete
mode: self-managed
---

# Phase 1 复习清单与速记

> [!note] 使用原则
> 本清单不判定你是否学会，也不阻塞后续课程。按自己的节奏勾选即可；需要面试冲刺时，优先练口述、白板和源码证据，不必搭实验环境。
>
> [!important] 公开面经扩展
> 当前入口为 [[interview-question-bank-expanded|114 题扩展主索引]]。先练 P0 题，再用 [[real-interview-source-register]] 区分直接复盘、可追溯汇总和训练题单。

## 1. 可选复习节奏

| 时间 | 建议动作 | 目标 |
|---|---|---|
| 当天 | 通读一课，合上笔记口述 3 分钟 | 建立结构 |
| D+1 | 回答该课 8 道面试题 | 暴露遗漏 |
| D+3 | 画架构或状态机，讲一个项目证据 | 建立调用链 |
| D+7 | 做跨课系统设计题 | 串联知识 |
| D+14 | 只看标题复述，核对危险答案 | 修正边界 |
| 面试前 | 30 秒、2 分钟、5 分钟三档模拟 | 控制表达 |

## 2. 七课一页速记

### 1.1 Agent

- 定义：目标驱动、状态化、在 Runtime 中执行 action-observation 闭环。
- 八要素：goal、state、policy、action、environment、observation、termination、governance。
- 边界：模型是策略组件，Runtime 管状态、执行、权限和终止。
- 项目：ESGQA = 领域状态图；agentHub = 事件驱动 Runtime。

### 1.2 Loop

- 主链：observe -> decide/plan -> validate -> authorize -> act -> observe -> update -> terminate。
- 预算：step、token、wall-clock deadline；检测循环与无进展。
- 模式概览：ReAct、Plan-and-Execute、ReWOO、DAG 并行、显式状态图。
- 完整专题：[[general-agent-architecture-patterns]]，还需掌握 Reflexion、Self-Refine、CRITIC、ToT、GoT、LATS、Multi-Agent 与 Hybrid。
- 失败：错误分类、有界重试、幂等、补偿、取消、部分结果。

### 1.3 Tool

- Tool 是能力契约，不是函数别名。
- 契约：Schema、权限、风险、副作用、幂等、timeout、result/error、audit。
- 执行权：模型请求 -> Runtime 校验/授权/审批 -> executor 执行。
- 区分：Function Calling != Registry != MCP。

### 1.4 Router

- 四层：规则、模型、策略、故障/编排。
- 决策：route、reason、confidence、matched rule、policy、fallback、version、trace。
- confidence 需校准；阈值绑定业务成本和风险。
- 评测：per-route 指标、校准、澄清/拒绝、后悔成本、延迟。

### 1.5 Provider

- 稳定核心 + capability matrix + native extension。
- 生命周期：start、send/stream、cancel、resume、stop。
- 规范化：message、tool、event、usage、error；唯一终态与迟到事件。
- 项目：agentHub 有 claude-code/opencode/test；无 Codex Provider。ESGQA 无通用层。

### 1.6 Context

- 四层：local runtime、model-visible、external memory、execution state。
- 预算：window - fixed - input - tools - output reserve - safety = 可选上下文。
- 减量：去噪、裁 Tool、检索/去重、摘要、压缩。
- 压缩：状态机、pending input、迟滞、来源和保真探针。

### 1.7 Security

- 模型与外部内容不可信；Prompt 不是授权边界。
- 分层：authn、business authz、tool authz、HITL、sandbox、audit。
- 文件：canonical root、link/reparse、TOCTOU、mount。
- 容器：user、rootfs、mount、cap、seccomp/LSM、network、resource、socket。

## 通用 Agent 架构模式速记

| 模式 | 必须说清的核心 | 首要风险 |
|---|---|---|
| ReAct | 决策与 Action/Observation 交错，Runtime 保留执行权 | 循环、注入、成本波动 |
| Plan-and-Execute | 先计划、校验、调度，再按观察重规划 | 计划漂移 |
| ReWOO | Planner 用变量引用未来观察，Worker 执行，Solver 汇总 | 中途适应弱、变量依赖错误 |
| Reflexion | 失败反馈形成语言反思，进入后续 trial 的 episodic memory | 错误反思持久化 |
| Self-Refine | draft -> self-feedback -> revision | 无外部证据时改坏正确答案 |
| CRITIC | 外部工具验证后修订 | 工具结果污染与权限 |
| ToT | 多分支生成、评估、剪枝和回溯 | evaluator 偏差、成本爆炸 |
| GoT | thought 可拆分、聚合、变换和反馈为图 | 合并、循环和 provenance |
| LATS | MCTS、LM value、reflection 与环境反馈 | 环境不可回滚、搜索昂贵 |
| LLMCompiler/DAG | 编译依赖并行执行 ready tasks | 副作用冲突、partial failure |
| Multi-Agent | 按能力、权限或上下文拆分主体 | 协调、责任和 Token 膨胀 |
| Hybrid | 确定性外壳包裹局部 Agent 策略 | 边界与状态转换复杂 |

面试回答顺序：目标 -> 组件/状态流 -> 与相邻模式差异 -> 失败/成本/安全 -> 适用边界 -> 项目真实映射。

## 3. 必画的白板图

- [ ] Agent 八要素与 Runtime 边界图。
- [ ] 带 validate/authorize/terminate 的 Agent Loop。
- [ ] ReAct、Plan-and-Execute、ReWOO 三种轨迹的并列对照。
- [ ] ToT/GoT/LATS 的树、图和 MCTS 控制差异。
- [ ] Reflexion、Self-Refine、CRITIC 的反馈来源与时间尺度。
- [ ] Model -> Tool Request -> Policy -> Executor -> ToolResult。
- [ ] Hard Rule -> Model Candidate -> Policy -> RouteDecision。
- [ ] Provider Factory -> Adapter -> Event Stream -> Runtime State。
- [ ] Context budget 分区与 compression state machine。
- [ ] User/Content -> API -> Runtime/Policy -> Tool Broker -> Sandbox 信任边界。

每张图检查：入口、状态、输出、失败、终止、指标、权限是否齐全。

## 4. 项目证据口述清单

### ESGQA

- [ ] `CBAMState`、节点、条件边与 `build_cbam_graph()`。
- [ ] `route_intent` 和 `check_*` 的正常/错误分支。
- [ ] Pydantic 结构化输出的价值与业务验证边界。
- [ ] `node_memory()` 的最近 4 条、长期画像、FAISS 兜底。
- [ ] BM25 + FAISS + Ensemble + Reranker + 业务提权。
- [ ] JWT、SlowAPI、Celery、Prometheus 的服务入口能力。
- [ ] 三个缺口：通用 Tool Registry、多 Provider、Agent 文件/命令沙箱。

### agentHub

- [ ] AbstractProvider、Factory、Runtime 事件主链。
- [ ] busy queue、permission event、idle/stop 生命周期。
- [ ] EventRoutingRules 的 priority 和字段匹配。
- [ ] Planner 有界重试、Manager continue/replan/abort。
- [ ] ContextBus 类型/衰减/引用权重、预算、归档/GC。
- [ ] Usage 超 70% 的压缩与 pending prompt。
- [ ] per-session Docker、mount、PermissionProfiles。
- [ ] 空 `allowedTools` 非 deny-all、`**/*` 和 glob 路径风险。
- [ ] Factory 无 Codex Provider，依赖不等于实现。
- [ ] threat model 声明但未证明修复的安全缺口。

证据总表：[[project-evidence-map]]。

## 5. 高频对比卡

| A | B | 一句话区别 |
|---|---|---|
| Agent | LLM call | Agent 有 Runtime 状态、环境动作、反馈与终止 |
| Workflow | Agent | 控制流主要由代码预定义 vs 受控动态决策 |
| Node | Tool | 图控制单元 vs 可请求、授权和执行的能力契约 |
| Schema valid | Action valid | 形状正确 vs 业务、权限和事实均允许 |
| Function Calling | MCP | 模型 API 调用格式 vs Host/Client/Server 协议 |
| Router | Classifier | 完整决策控制面 vs 标签预测组件 |
| retry | fallback | 重做同类动作 vs 切受约束替代路径 |
| timeout | cancel | 不再等待 vs 请求远端停止工作 |
| Provider name | capability | 名称不能替代可发现的真实能力 |
| local context | model context | 运行时私有对象 vs 实际进入模型的内容 |
| memory | vector DB | 生命周期化知识/事件 vs 一种检索存储 |
| authn | authz | 身份是谁 vs 是否允许具体动作 |
| allowlist glob | path safety | 字符串策略 vs 真实路径、链接、竞态和 mount 安全 |
| Docker | complete isolation | 隔离原语 vs 依赖完整配置的安全边界 |

## 6. 危险答案清单

- [ ] 不说用了框架所以自动可靠。
- [ ] 不说 temperature=0 所以完全确定。
- [ ] 不说 Pydantic/JSON Schema 通过所以动作安全。
- [ ] 不说异常统一重试或统一 fallback。
- [ ] 不说大窗口可以替代上下文选择和检索。
- [ ] 不说向量库等于完整长期记忆。
- [ ] 不说 JWT/Celery 等于 Agent 沙箱。
- [ ] 不说 Docker/glob 等于文件和主机安全。
- [ ] 不把论文 benchmark 外推成项目性能。
- [ ] 不把框架、依赖、README 声明冒充冻结源码实现。
- [ ] 不把 StateGraph 叫 GoT，不把事件循环叫 ReAct，不把摘要叫 Reflection。
- [ ] 不因为项目没实现某论文架构，就省略通用面试知识。
- [ ] 不虚构线上规模、准确率、成本收益或真实公司题目。

## 7. 三档表达模板

### 30 秒

结论 -> 两到三个核心构件 -> 一个项目证据 -> 一个边界。

### 2 分钟

问题与目标 -> 接口/状态机 -> 正常路径 -> 失败/安全 -> 指标 -> 项目证据与缺口。

### 5 分钟

再补方案比较、选择理由、极端场景、验证计划和演进顺序。讲项目时固定使用：

```text
冻结版本与位置 -> 调用链 -> 已实现 -> 可演示
-> 限制/未实现 -> 改进建议 -> 如何验证
```

## 8. 面试前最终核对

- [ ] 能在 5 分钟讲清 Phase 1 六层系统图。
- [ ] 每课至少能回答 4 个追问，不只背主问题。
- [ ] 每个项目至少准备 3 条完整调用链。
- [ ] 每条项目主张能指向 [[project-evidence-map]]。
- [ ] 能主动说出一个实现缺口并给出验证严谨的改造路径。
- [ ] 不确定的内容使用需核验，而不是现场猜测。
- [ ] 代码示例只说教学设计，不说已在项目运行。

完整题库：[[interview-question-bank-expanded]]；面经来源：[[real-interview-source-register]]；技术依据：[[technical-source-register]]、[[technical-source-register-interview-expansion]]。


