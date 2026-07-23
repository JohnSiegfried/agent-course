---
tags: [phase-1, project-evidence, esgqa, agenthub, interview]
created: 2026-07-22
updated: 2026-07-22
status: complete
evidence_commits: [ef02aad, 815f645]
---

# Phase 1 双项目证据地图

> [!important] 使用规则
> 本文件是课程中所有项目陈述的事实底座。Lesson 可以解释和比较这些证据，但不能扩大证据范围。`源码已验证` 不等于功能已经在当前机器运行；没有保存运行命令与输出时，不得改写为 `运行已验证`。

## 1. 冻结版本

| 项目 | 冻结版本 | 证据入口 | 核验结果 |
|---|---|---|---|
| ESGQA | `ef02aad64c30976274ec63b8c9b195b350336729` | `D:\hm\ESGQA`，通过 `git -c safe.directory=D:/hm/ESGQA show ef02aad:<path>` 读取 | Git 对象类型为 `commit`；课程使用 `agents.py`、`retriever_core.py`、`server.py` |
| agentHub | `815f645` | `learn/_meta/evidence/agentHub-reference-815f645.zip` | 归档 96 个条目；关键 Agent/Provider/Context/Sandbox 文件齐全 |

## 2. 五类事实标签

| 标签 | 含义 | 面试使用方式 |
|---|---|---|
| `源码已验证` | 在冻结提交中定位到类型、函数、分支或配置 | “我在提交 X 的文件 Y、符号 Z 中确认……” |
| `运行已验证` | 保存了环境、命令、输入和真实输出 | 当前 Phase 1 没有给两个项目新增此标签 |
| `文档声明` | README、ADR、注释或威胁模型如此描述，但代码未完全证明 | 必须说“文档列出/声明”，不能说“系统已经保证” |
| `未实现` | 冻结证据中没有对应实现，只有依赖、接口缺口或显式待办 | 用于限制项目包装，不能转换成成果 |
| `优化建议` | 基于缺口提出的未来设计 | 使用“可以增加/下一步会”这样的将来时 |

## 3. Lesson 索引

| Lesson | ESGQA | agentHub |
|---|---|---|
| 1.1 Agent 定义 | `P1-ESG-STATEGRAPH-001` | `P1-HUB-RUNTIME-001` |
| 1.2 Agent Loop | `P1-ESG-STATEGRAPH-001`、`P1-ESG-ROUTING-001` | `P1-HUB-RUNTIME-001`、`P1-HUB-PLANNER-001` |
| 1.3 Tool-use | `P1-ESG-GAP-TOOL-REGISTRY-001`、`P1-ESG-STRUCTURED-OUTPUT-001` | `P1-HUB-PERMISSION-001`、`P1-HUB-RUNTIME-001` |
| 1.4 路由 | `P1-ESG-ROUTING-001` | `P1-HUB-ROUTING-001`、`P1-HUB-PLANNER-001` |
| 1.5 Provider | `P1-ESG-GAP-PROVIDER-001` | `P1-HUB-PROVIDER-001`、`P1-HUB-GAP-CODEX-PROVIDER-001` |
| 1.6 上下文 | `P1-ESG-MEMORY-001`、`P1-ESG-RETRIEVAL-001` | `P1-HUB-CONTEXT-001`、`P1-HUB-COMPRESSION-001` |
| 1.7 安全 | `P1-ESG-API-SECURITY-001`、`P1-ESG-GAP-SANDBOX-001` | `P1-HUB-SANDBOX-001`、`P1-HUB-PERMISSION-001`、`P1-HUB-DOC-SECURITY-GAPS-001` |

## 4. ESGQA 证据卡

### P1-ESG-STATEGRAPH-001：领域状态图

- **Lesson/概念**：1.1、1.2；状态、节点、边、确定性工作流。
- **位置**：`agents.py:106-141` 定义 `CBAMState`；`agents.py:906-966` 定义 `build_cbam_graph()`。
- **调用链**：`START -> memory -> intent -> 条件分支 -> evaluator -> export -> END`；错误分支进入 `fallback -> END`。
- **正常行为**：共享状态携带查询、记忆、业务参数、检索上下文、错误与指标；节点按显式边更新状态。
- **失败行为**：多个 `check_*()` 读取 `error_msg` 后转入 fallback。
- **标签**：`源码已验证`。
- **限制**：这是 CBAM 领域 Pipeline；图中没有通用的“模型自由选择任意工具”循环。
- **可说**：“我把 ESGQA 设计成显式状态图，业务阶段、状态字段和失败出口都能从源码定位。”
- **不可说**：“ESGQA 已实现通用 ReAct Agent 或任意任务自治。”
- **Demo**：`DEMO-ESG-01`。

### P1-ESG-ROUTING-001：意图与错误路由

- **Lesson/概念**：1.2、1.4；规则路由、模型分类结果、默认分支、fallback。
- **位置**：`agents.py:251-285` 的 `node_intent_parser()`；`agents.py:923-959` 的 `route_intent()` 与 `check_*()`。
- **调用链**：Pydantic `IntentOutput` -> 写入 `task_type` -> `route_intent()` 选择 `policy_qa/web_search/general_chat/retriever`；错误优先进入 fallback。
- **正常行为**：分类值映射到固定节点；未命中特殊类型时默认进入 calculation 的 retriever 路径。
- **失败行为**：`error_msg` 高于业务分支；各阶段错误均转入 fallback。
- **标签**：`源码已验证`。
- **限制**：`task_type` 没有在路由函数内再次做枚举校验；默认 calculation 可能掩盖未知分类。
- **可说**：“路由采用模型结构化分类加程序规则的混合方式，错误分支优先于业务分支。”
- **不可说**：“所有路由都有概率校准、人工确认和策略引擎。”
- **Demo**：`DEMO-ESG-02`。

### P1-ESG-STRUCTURED-OUTPUT-001：结构化模型输出

- **Lesson/概念**：1.1、1.3、1.4；类型边界与 Schema。
- **位置**：`agents.py:144-170` 的 `IntentOutput`、`FactorOutput`、`RatesOutput`、`RagTriadEval`；调用见 `agents.py:254`、`:438`、`:467`、`:665`。
- **调用链**：`ChatOllama(...).with_structured_output(PydanticModel)` -> `invoke()` -> 类型字段写入 `CBAMState`。
- **正常行为**：意图、排放因子、汇率/碳价、RAG 三元评估以明确字段承载。
- **失败行为**：模型解析异常由节点异常处理逻辑转为 `error_msg` 或降级结果，具体节点策略并不完全一致。
- **标签**：`源码已验证`。
- **限制**：Pydantic 输出模型不是通用 Tool Schema，也不能证明所有数值语义正确。
- **可说**：“关键 LLM 输出先经过 Pydantic 结构化，再进入业务状态，减少自由文本解析。”
- **不可说**：“有 Schema 就不会幻觉，或系统已经具备通用 Function Calling。”
- **Demo**：`DEMO-ESG-03`。

### P1-ESG-MEMORY-001：长短期记忆组装

- **Lesson/概念**：1.6；上下文来源、优先级和污染风险。
- **位置**：`agents.py:202-248` 的 `node_memory()`。
- **调用链**：SQLite 最近 4 条 -> SQLite 长期画像 -> FAISS 相似搜索冷启动 -> 拼接为 `memory_context` -> 意图节点使用。
- **正常行为**：有结构化记忆时仍附加向量兜底；无数据库结果时使用 FAISS 文本。
- **失败行为**：SQLite 异常被降级为空字符串；FAISS 检索本身没有在该函数内捕获。
- **标签**：`源码已验证`。
- **限制**：没有 Token 预算、摘要质量评估、TTL、用户可编辑记忆或敏感数据治理证据；`user_id` 固定为 `user_001`。
- **可说**：“ESGQA 区分近期对话、长期画像和向量兜底，并在意图识别前组装。”
- **不可说**：“已实现完整企业级长期记忆生命周期。”
- **Demo**：`DEMO-ESG-04`。

### P1-ESG-RETRIEVAL-001：混合检索与缓存降级

- **Lesson/概念**：1.6，并为后续 RAG Phase 铺垫。
- **位置**：`retriever_core.py:25-97` 的初始化；`:99-224` 的 `search()`。
- **调用链**：Redis 缓存读取 -> BM25 与 FAISS 各召回 -> Ensemble 0.4/0.6 -> 元数据和启发式过滤 -> BGE Reranker -> 业务规则提权 -> Redis 写回。
- **正常行为**：混合模式扩大到 `k=30` 后精排；声明日期参与有效期过滤；缓存 Key 包含查询、模式、优先文件和日期。
- **失败行为**：Redis 缺失或连接失败时无缓存运行；空候选返回空列表；向量库不存在时初始化失败。
- **标签**：`源码已验证`。
- **限制**：`FAISS.load_local(...allow_dangerous_deserialization=True)` 需要可信索引文件；没有当前运行指标证明召回质量。
- **可说**：“代码体现稀疏+稠密+重排+业务提权的多阶段检索，并对 Redis 做可用性降级。”
- **不可说**：“混合检索已经在线达到某个准确率或缓存一定提升多少性能。”
- **Demo**：`DEMO-ESG-05`。

### P1-ESG-API-SECURITY-001：API 鉴权、限流与异步隔离

- **Lesson/概念**：1.7；应用层安全与稳定性。
- **位置**：`server.py:39-80` 初始化 FastAPI、SlowAPI、Prometheus、Celery；`:96-124` JWT；`:129-145` Agent 异步任务；`:200-240` 端点与限流。
- **调用链**：登录获取 JWT -> 请求依赖校验 -> 10/minute 限流 -> Celery 提交任务 -> 状态查询；Prometheus 暴露指标。
- **正常行为**：API 调用与后台 Agent 执行解耦；身份和请求频率在入口控制。
- **失败行为**：JWT 解码失败返回 401；后台任务异常进入 Celery 重试/失败状态。
- **标签**：`源码已验证`。
- **限制**：这不是 Agent 工具沙箱；源码中 `SECRET_KEY` 管理、租户隔离和业务授权仍需单独审计。
- **可说**：“ESGQA 在服务入口有 JWT、限流、异步任务与指标，不把长任务直接压在同步请求上。”
- **不可说**：“JWT 与 Celery 等于 Agent 执行环境已经隔离。”
- **Demo**：`DEMO-ESG-06`。

### P1-ESG-GAP-TOOL-REGISTRY-001：没有通用 Tool Registry

- **Lesson/概念**：1.3。
- **核验范围**：`agents.py` 的 retriever/fetcher/calculator/web 节点及冻结项目结构。
- **结果**：外部能力由特定节点直接实现和调用；未定位到统一 `ToolSpec/ToolRegistry`、每工具输入 Schema、权限元数据和标准 `ToolResult`。
- **标签**：`未实现`。
- **限制**：这不否定节点能调用外部能力，只限制“通用工具系统”的表述。
- **优化建议**：引入工具契约、Registry、参数校验、权限和副作用等级；代价是额外抽象、迁移和错误协议治理。
- **可说**：“当前是节点级能力集成；若扩展为开放 Agent，我会先抽统一 Tool 协议。”
- **不可说**：“ESGQA 已接入 MCP 或拥有生产级 Tool Registry。”
- **Demo**：`DEMO-GAP-01`。

### P1-ESG-GAP-PROVIDER-001：没有多 Provider 抽象

- **Lesson/概念**：1.5。
- **核验范围**：`agents.py` 中多处直接构造 `ChatOllama(model=LLM_MODEL, ...)`。
- **结果**：未定位到 Provider Protocol、Factory、能力发现、统一事件或跨厂商消息转换。
- **标签**：`未实现`。
- **优化建议**：先抽最小 `complete(messages, tools)` 接口，再按能力而非厂商堆条件；需要承担抽象泄漏和测试矩阵成本。
- **可说**：“ESGQA 当前是 Ollama 单 Provider；多模型是改造方案，不是现状。”
- **不可说**：“支持 Anthropic/OpenAI/Ollama 动态切换。”
- **Demo**：`DEMO-GAP-02`。

### P1-ESG-GAP-SANDBOX-001：没有 Agent 文件/命令沙箱

- **Lesson/概念**：1.7。
- **核验范围**：Agent 节点、服务入口和项目结构。
- **结果**：未定位到 workspace 根目录策略、命令白名单、`shell=False` 执行器、符号链接/TOCTOU 加固或容器级每会话沙箱。
- **标签**：`未实现`。
- **优化建议**：若未来开放文件或命令工具，应采用默认拒绝、最小挂载、非 root、网络出站控制和审计。
- **可说**：“现有 API 安全不能替代 Agent 能力安全；项目目前没有开放通用命令执行。”
- **不可说**：“ESGQA 已完成三层 Agent 沙箱。”
- **Demo**：`DEMO-GAP-03`。

## 5. agentHub 证据卡

### P1-HUB-RUNTIME-001：事件驱动 Agent Runtime

- **Lesson/概念**：1.1、1.2、1.3；生命周期、队列、事件、权限与终止。
- **位置**：`apps/api/src/agent/AgentRuntime.ts:138` 的 `AgentRuntime`；`:168-255` Provider 切换、忙时排队和压缩入口；`:296-345` Provider 创建/启动；`:452-503` 权限事件；`:826-904` 队列与空闲关闭。
- **调用链**：收到 prompt -> 检查运行实例和 Provider -> busy 时排队、idle 时发送 -> Provider 事件归一化 -> 权限/工具/完成处理 -> 下一队列或 idle timer -> 停止/分离容器。
- **正常行为**：同一 Agent 生命周期由 Runtime 管理；Provider 配置变化只在空闲切换；事件驱动后续处理。
- **失败行为**：Provider/容器异常进入事件和清理分支；权限请求有超时；队列发送失败有 fallback。
- **标签**：`源码已验证`。
- **限制**：不是教材里的单函数 while-loop；底层推理/Tool loop 的一部分由 Provider SDK 承担。
- **可说**：“agentHub 的 AgentRuntime 是生命周期与事件编排层，负责 Provider、队列、权限和空闲回收。”
- **不可说**：“所有模型推理步骤都由 AgentRuntime 自己实现。”
- **Demo**：`DEMO-HUB-01`。

### P1-HUB-PROVIDER-001：Provider 接口与 Factory

- **Lesson/概念**：1.5。
- **位置**：`apps/api/src/agent/providers/base.ts:1-73`；`factory.ts:1-35`。
- **调用链**：`ProviderFactory.init()` 注册 `claude-code/opencode/test` -> `create(name)` -> `AbstractProvider.start()` -> `sendPrompt()` -> `onEvent()` 输出统一事件 -> `stop()`。
- **正常行为**：Runtime 依赖接口而非具体 Provider；Factory 拒绝未知名称；事件结构覆盖 thinking/tool/subagent/text/usage 等类别。
- **失败行为**：未注册 Provider 创建时报错；具体 Provider 启停错误由 Runtime 处理。
- **标签**：`源码已验证`。
- **限制**：统一接口仍可能泄漏 SDK 差异；并非每个 Provider 支持完全相同的工具、会话和事件语义。
- **可说**：“项目已有 Provider 接口、Factory 和两个生产实现，Runtime 可以按配置构造。”
- **不可说**：“任何模型或 SDK 都能零成本接入且行为完全一致。”
- **Demo**：`DEMO-HUB-02`。

### P1-HUB-ROUTING-001：优先级事件路由规则

- **Lesson/概念**：1.4。
- **位置**：`EventRoutingRules.ts:7-44` 的 `RouteRule/RouteEvent/RouteMatch`；`:214-294` 的排序、匹配与输出。
- **调用链**：规则按 priority 降序 -> 逐项匹配 eventType/toolName/sender/exitCode/path/result -> 为每个 notify target 构造 InboxEntry。
- **正常行为**：支持默认规则、自定义追加/替换、glob 路径和多目标通知。
- **失败行为**：字段缺失或不匹配时跳过；没有命中则返回空数组。
- **标签**：`源码已验证`。
- **限制**：返回空数组是隐式“无动作”；ID 使用时间与随机数，不适合直接做确定性重放。
- **可说**：“事件路由是可审计的规则层，优先级和匹配字段都在代码中显式化。”
- **不可说**：“规则冲突已被形式化证明，或路由一定只有一个结果。”
- **Demo**：`DEMO-HUB-03`。

### P1-HUB-CONTEXT-001：结构化 ContextBus 与预算选择

- **Lesson/概念**：1.6。
- **位置**：`ContextBus.ts:24-46` 存储与权重；`:48-91` 版本化 set；`:107-134` query；`:137-173` `getProjectDigest(maxTokens)`；`:242-280` 归档与 GC。
- **调用链**：结构化 ContextEntry 写入 -> 按类型/标签/状态查询 -> 类型基础分、7 天衰减和引用奖励排序 -> 在 Token 预算内构建 digest -> 归档/GC。
- **正常行为**：convention、decision、known-issue 等类型有不同基础优先级；超预算时截断当前值并停止。
- **失败行为**：预算非正或无活动条目返回空；超过 maxEntries 淘汰最旧条目。
- **标签**：`源码已验证`。
- **限制**：权重和字符截断是启发式；没有证据证明摘要信息保真或跨重启持久化由该类单独完成。
- **可说**：“ContextBus 把上下文变成带类型、状态、版本、作者和权重的结构化条目，再按预算选取。”
- **不可说**：“上下文选择已经通过离线评测证明最优。”
- **Demo**：`DEMO-HUB-04`。

### P1-HUB-COMPRESSION-001：70% 阈值压缩流程

- **Lesson/概念**：1.6。
- **位置**：`AgentRuntime.ts:76` 阈值常量；`:240-253` 压缩入口；`:508-539` 摘要完成后的 Provider 重启；`:787-790` Usage 触发。
- **调用链**：usage 计算 contextPct > 70 -> `needsCompression=true` -> 下一个 prompt 暂存 -> 发送压缩 prompt -> 收到 summary -> 重启 Provider -> summary + pending prompt 恢复会话。
- **正常行为**：压缩与用户新请求串行，避免两个 prompt 同时进入旧上下文。
- **失败行为**：摘要为空时使用 fallback 文本；重启失败走 Runtime 异常处理。
- **标签**：`源码已验证`。
- **限制**：70% 是硬阈值；没有迟滞区间、摘要保真评分或压缩前后任务质量证据。
- **可说**：“实现了基于 Usage 比例的压缩状态机，并显式保存压缩期间到达的 prompt。”
- **不可说**：“70% 是普适最佳阈值，压缩不会丢信息。”
- **Demo**：`DEMO-HUB-05`。

### P1-HUB-SANDBOX-001：每会话 Docker 沙箱与挂载

- **Lesson/概念**：1.7。
- **位置**：`SandboxManager.ts:29-88`。
- **调用链**：解析 session sandbox -> 选择用户 workspace -> 清理旧容器 -> 构造 `/sandbox`、`/workspace`、`/home/agents` bind mounts -> 设置内存和工作目录 -> 启动容器。
- **正常行为**：每个 session 使用命名容器；运行目录、用户文件和 Agent 持久身份分区挂载；有内存上限。
- **失败行为**：自定义 workspace 不存在时抛错；同名旧容器先清理。
- **标签**：`源码已验证`。
- **限制**：该段没有只读根文件系统、capability drop、`no-new-privileges`、自定义 seccomp/AppArmor、网络隔离；bind mount 权限取决于宿主机配置。
- **可说**：“项目具备会话级容器和三类目录挂载，解决生命周期与文件域划分。”
- **不可说**：“只要用了 Docker 就已经达到强多租户隔离。”
- **Demo**：`DEMO-HUB-06`。

### P1-HUB-PERMISSION-001：角色工具与写路径权限

- **Lesson/概念**：1.3、1.7。
- **位置**：`PermissionProfiles.ts:1-64` 权限模型与内置角色；`:74-124` 检查逻辑。
- **调用链**：按 agentName 取 profile -> forbiddenTools 优先拒绝 -> 非空 allowedTools 白名单 -> Write/Edit/NotebookEdit 检查 writePatterns -> 允许或返回理由/委派目标。
- **正常行为**：planner/reviewer 默认不可写；code-agent 按扩展名写；test-agent 仅测试路径；拒绝可委派 code-agent。
- **失败行为**：未知 Agent 使用宽松的 `CUSTOM_AGENT_DEFAULT`，其 write/read 为 `**/*`；空 allowedTools 在代码语义上不是“禁止全部”，而是“不启用白名单约束”。
- **标签**：`源码已验证`。
- **限制**：glob 是字符串正则转换，不是路径 canonicalization；不能防符号链接和 TOCTOU；工具名权限与真实操作权限仍需执行层配合。
- **可说**：“角色权限有明确检查顺序，但默认配置和路径语义仍需继续收紧。”
- **不可说**：“PermissionProfiles 单独阻止了所有越权文件访问。”
- **Demo**：`DEMO-HUB-07`。

### P1-HUB-PLANNER-001：计划校验、有限重试与失败审查

- **Lesson/概念**：1.2、1.4。
- **位置**：`PlannerAgent.ts:7-90`；`ManagerLoop.ts:64-149`。
- **调用链**：Planner 生成 JSON -> `extractAndValidate` -> 校验失败有限重试 -> 任务失败时 Manager 汇总失败、上游结果和文件树 -> 输出 continue/replan/abort -> 非法动作回落 abort。
- **正常行为**：规划和执行失败决策分层；解析失败有上限；Manager 输出受 Schema 和动作集合约束。
- **失败行为**：超过 Planner 重试上限抛错；Manager 解析/调用异常默认 abort。
- **标签**：`源码已验证`。
- **限制**：模型决策正确性仍取决于上下文；日期型 plan ID 不保证跨进程全局唯一。
- **可说**：“规划结果先校验，运行失败再由 Manager 在有限动作集合中决策，避免无上限自循环。”
- **不可说**：“Planner 一定生成正确 DAG，Manager 能恢复所有失败。”
- **Demo**：`DEMO-HUB-08`。

### P1-HUB-GAP-CODEX-PROVIDER-001：Codex 依赖不等于 Provider 实现

- **Lesson/概念**：1.5。
- **核验范围**：Provider 目录与 `ProviderFactory.init()`。
- **结果**：Factory 只注册 `claude-code`、`opencode`、`test`；归档中无 `providers/codex.ts`。即使依赖清单包含 Codex SDK，也不能证明运行时已接通。
- **标签**：`未实现`。
- **优化建议**：实现 Codex Provider 时需要对齐事件、会话、工具、权限、取消和 Usage；不能只完成 API 调用。
- **可说**：“依赖已经存在，但当前冻结提交没有 Codex Provider；这是明确缺口。”
- **不可说**：“agentHub 已支持 Codex Provider。”
- **Demo**：`DEMO-GAP-04`。

### P1-HUB-DOC-SECURITY-GAPS-001：威胁模型记录的生产缺口

- **Lesson/概念**：1.7。
- **位置**：`docs/security/threat-model.md:41-42` 容器缺口；`:63` iframe/CSP；`:83-85` Trust 模式；`:96` workspace 写范围；`:122-124` 修复优先级。
- **内容**：文档明确列出未启用 seccomp/AppArmor、只读根文件系统、API 容器挂载 Docker socket、iframe sandbox/CSP、Trust ON 审计、workspace 写边界等风险。
- **标签**：`文档声明`。
- **限制**：这是项目自己的威胁模型，不等同于外部安全认证；部分描述可能随代码演进变化。
- **可说**：“项目威胁模型主动记录了容器、预览、权限和审计缺口，我会把它们作为改进清单。”
- **不可说**：“这些风险已经全部修复，或项目通过了渗透测试。”
- **Demo**：`DEMO-HUB-09`。

## 6. 文字 Demo 索引

| Demo ID | 目标 | 类型 | 使用位置 |
|---|---|---|---|
| `DEMO-ESG-01` | 从 CBAMState 走到状态图与 fallback | 静态源码走查 | 1.1、1.2 |
| `DEMO-ESG-02` | 对比合法意图、未知意图和 error_msg 优先级 | 分支推演 | 1.4 |
| `DEMO-ESG-03` | Pydantic 字段合法但业务语义错误 | 失败推演 | 1.3、1.4 |
| `DEMO-ESG-04` | 新问题被旧记忆污染的风险与防火墙规则 | 状态推演 | 1.6 |
| `DEMO-ESG-05` | BM25/FAISS/Reranker/缓存的多阶段检索 | 调用链走查 | 1.6 |
| `DEMO-ESG-06` | JWT、限流与 Celery 不等于工具沙箱 | 边界辨析 | 1.7 |
| `DEMO-HUB-01` | busy 排队、Provider 事件、idle 关闭 | 生命周期推演 | 1.1、1.2 |
| `DEMO-HUB-02` | Factory 创建 Provider 与未知名称失败 | 接口推演 | 1.5 |
| `DEMO-HUB-03` | 两条规则同时命中及优先级含义 | 路由推演 | 1.4 |
| `DEMO-HUB-04` | ContextEntry 权重与预算裁剪 | 手算示例 | 1.6 |
| `DEMO-HUB-05` | 70% 压缩阈值、pending prompt 与摘要损失 | 状态机推演 | 1.6 |
| `DEMO-HUB-06` | 三分区挂载和容器未加固的残余风险 | 威胁建模 | 1.7 |
| `DEMO-HUB-07` | 空 allowedTools、宽松 custom default 与路径 glob | 权限反例 | 1.3、1.7 |
| `DEMO-HUB-08` | Planner 校验失败重试与 Manager abort | 故障推演 | 1.2、1.4 |
| `DEMO-HUB-09` | 威胁模型中的未修复项转成改造顺序 | 设计评审 | 1.7 |
| `DEMO-GAP-01` | 把 ESGQA 节点能力迁移为统一 Tool 协议 | 优化建议 | 1.3 |
| `DEMO-GAP-02` | 把单 Ollama 调用迁移为 Provider 层 | 优化建议 | 1.5 |
| `DEMO-GAP-03` | 为 ESGQA 增加工作区沙箱的最小步骤 | 优化建议 | 1.7 |
| `DEMO-GAP-04` | Codex Provider 不能只做 HTTP 适配 | 优化建议 | 1.5 |

## 7. 面试表达总边界

1. 先说项目目标和你负责/研究的代码，再说具体提交、文件和状态流。
2. 使用“源码表明”与“运行观察到”时必须区分；当前材料大多属于前者。
3. 看到外部调用节点，不自动称为 Tool Registry；看到 Docker，不自动称为强沙箱；看到 SDK 依赖，不自动称为 Provider 已实现。
4. 优化建议按“现状 -> 缺口 -> 方案 -> 代价 -> 验证”回答，绝不偷换成已有成果。
5. 面试官追问数字时，没有真实压测或评估结果就明确说“当前未做运行验证”，再给出你会如何测。


