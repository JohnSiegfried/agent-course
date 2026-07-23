---
tags: [phase-1, index, navigation, agent-foundations]
created: 2026-07-22
updated: 2026-07-23
status: complete
phase_status: content-complete
---

# Phase 1：Agent 工程基础

本阶段为纯内容课程：不要求搭建实验环境，不安装依赖，不提供可运行实验包。课程中的代码均为教学示例，并在课内标明未在项目运行。学习进度由你自行掌握，本目录只记录内容完成状态。

> [!important] 通用架构面试必修
> 两个项目是否使用 ReAct 不影响课程覆盖。完成 1.2 后阅读 [[general-agent-architecture-patterns|通用 Agent 架构模式专题]]，系统掌握 ReAct、Planning、ReWOO、Reflection、Search、DAG、Multi-Agent 和 Hybrid；项目归属仍以证据地图为准。

## 建议阅读顺序

1. [[lessons/1.1-agent-definition-and-first-principles|1.1 Agent 定义与第一性原理]]
2. [[lessons/1.2-agent-loop-patterns|1.2 Agent Loop 与编排模式]]
3. [[general-agent-architecture-patterns|通用 Agent 架构模式（面试必修专题）]]
4. [[lessons/1.3-tool-use-design|1.3 Tool-use 设计模式]]
5. [[lessons/1.4-routing-and-decision|1.4 Router 与决策边界]]
6. [[lessons/1.5-provider-abstraction|1.5 Provider 抽象与多模型运行时]]
7. [[lessons/1.6-context-engineering|1.6 Context Engineering、记忆与压缩]]
8. [[lessons/1.7-agent-security-boundaries|1.7 Agent 安全边界与纵深防御]]

## 面试与复习

- [[interview-question-bank-expanded|114 题扩展面试题库]]：保留 Q1-Q84，新增 30 道公开面经衍生题，并提供 7 个详细答案模块。
- [[general-agent-architecture-patterns|通用 Agent 架构模式]]：面试必修的独立架构知识与项目边界。
- [[review-checklist|复习清单]]：一页速记、白板图、项目证据、危险答案与三档表达。
- [[project-evidence-map|项目证据地图]]：ESGQA 与 agentHub 的源码事实、限制、Demo 和不可说边界。
- [[technical-source-register|技术来源登记]]：原有论文、官方文档、规范、适用主张与限制。
- [[real-interview-source-register|公开面经来源登记]]、[[interview-source-to-question-map|去重映射]]、[[technical-source-register-interview-expansion|新增答案技术来源]]：分离题目来源与答案依据。

## 项目冻结基线

| 项目 | 冻结版本 | Phase 1 主要用途 |
|---|---|---|
| ESGQA | `ef02aad64c30976274ec63b8c9b195b350336729` | 领域状态图、条件路由、结构化输出、记忆、混合检索、API 入口安全 |
| agentHub | `815f645` | Agent Runtime、Provider、事件路由、Planner/Manager、ContextBus、压缩、沙箱与权限 |

所有项目论据以 [[project-evidence-map]] 为准。框架能力、依赖项或文档规划不能替代源码实现证据；后续项目更新时应新增冻结版本和证据卡，不覆盖旧结论。

## 内容状态

| 内容 | 状态 |
|---|---|
| 7 课详细课程与笔记 | complete |
| 项目证据地图 | complete |
| 技术来源登记 | complete |
| 通用 Agent 架构模式专题 | complete |
| 114 题面试题库（84 道原训练题 + 30 道公开面经衍生题） | complete |
| 复习清单 | complete |
| 可运行实验环境 | 不在本次范围 |

## 一句话主线

Agent 是受策略约束的状态化 Runtime：Loop 管控制，Tool 管能力，Router 管决策，Provider 管模型执行，Context 管有限信息，Security 管副作用边界。


