---
tags: [phase-1, sources, rag, retrieval, langchain, langgraph, interview-expansion]
created: 2026-07-23
updated: 2026-07-23
status: complete
source_count: 9
---

# Phase 1 面试扩展技术来源补充

> [!note] 用途
> 本文件补充 Q85-Q114 新增答案所需的一手技术依据。它与 [[real-interview-source-register]] 分工不同：后者说明“公开面经报告问过什么”，本文件说明“答案的技术事实依据是什么”。原有基础来源仍见 [[technical-source-register]]。

| ID | 来源 | 类型 | 主要支持 | 限制 |
|---|---|---|---|---|
| `SRC-RAG-001` | [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401) | NeurIPS 论文 | 参数化模型与外部非参数记忆/检索结合的 RAG 基础定义 | 原论文的模型和任务不等于今天所有 RAG 工程实现 |
| `SRC-RAG-SURVEY-001` | [Retrieval-Augmented Generation for Large Language Models: A Survey](https://arxiv.org/abs/2312.10997) | 综述论文 | Naive/Advanced/Modular RAG、检索/增强/生成与评测问题地图 | 综述用于知识地图，不把其中跨论文数字直接比较 |
| `SRC-BEIR-001` | [BEIR: A Heterogeneous Benchmark for Zero-shot Evaluation of Information Retrieval Models](https://arxiv.org/abs/2104.08663) | 论文/Benchmark | 异构检索评测、BM25 基线、dense/sparse/rerank 泛化与成本权衡 | BEIR 结果不能替代个人业务语料评测 |
| `SRC-ELASTIC-ALIAS-001` | [Elasticsearch Aliases](https://www.elastic.co/guide/en/elasticsearch/reference/current/aliases.html) | 官方文档 | alias 原子切换、写索引与无停机 reindex 语义 | 只证明 Elasticsearch API；其他向量库需自己的版本路由 |
| `SRC-ELASTIC-HYBRID-001` | [Elastic Hybrid Search](https://www.elastic.co/docs/solutions/search/hybrid-search) | 官方文档 | 全文与向量检索组合、RRF 融合 | 官方推荐不等于在任何数据集上必然最优 |
| `SRC-ELASTIC-RERANK-001` | [Elastic Ranking and Reranking](https://www.elastic.co/docs/solutions/search/ranking) | 官方文档 | 第一阶段候选与语义 rerank/LTR 的分层和成本 | 具体模型效果仍需目标语料消融 |
| `SRC-FAISS-METRIC-001` | [Faiss MetricType and distances](https://github.com/facebookresearch/faiss/wiki/MetricType-and-distances) | 官方项目 Wiki | L2、内积、归一化后 cosine 与距离关系 | ANN/量化会引入近似误差，公式不等于实际索引无误差 |
| `SRC-LANGCHAIN-AGENT-001` | [LangChain Agents](https://docs.langchain.com/oss/python/langchain/agents) | 官方文档 | 预构建模型+Tool Agent Loop，以及其基于 LangGraph 的实现关系 | 当前 API 可能演进，面试前应重新核验 |
| `SRC-LANGGRAPH-OVERVIEW-001` | [LangGraph Overview](https://docs.langchain.com/oss/python/langgraph/overview) | 官方文档 | 低层有状态编排、持久执行、流式和 HITL；可独立使用 | 使用框架不代表这些能力已在项目中配置 |

## 不能扩大为项目事实

1. Elastic 官方支持 alias 原子切换，不代表 ESGQA 已实现双索引热更新。
2. Faiss 给出距离语义，不代表 ESGQA 当前 Embedding、归一化和索引参数已经最优。
3. LangGraph 官方提供持久执行能力，不代表 ESGQA 已配置 checkpoint 或故障恢复。
4. BEIR 展示不同检索方法在异构数据上的权衡，不提供 ESGQA 的 Recall@K。
5. RAG 论文报告的实验结果不能用于个人项目简历数字。


