# LangChain的Memory组件如何工作？有哪些Memory类型？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- LangChain的Memory组件通过存储和管理对话历史，为无状态的大语言模型提供上下文连续性。
- **核心闭环**：交互前通过 `load_memory_variables()` 提取历史注入Prompt，交互后通过 `save_context()` 保存新对话。

## 机制与原理
- **ConversationBufferMemory**：直接存储完整对话历史，保证完整性，适合短对话场景。
- **ConversationBufferWindowMemory**：滑动窗口机制，仅保留最近K轮对话，解决Token消耗过多的问题。
- **ConversationSummaryMemory**：调用LLM对历史对话进行智能摘要（有损压缩），在保留核心信息的同时节省Token。
- **ConversationSummaryBufferMemory**：结合窗口与摘要优势，对早期对话进行摘要，保留近期完整对话，实现渐进式信息管理。
- **VectorStoreRetrieverMemory**：将对话向量化存储，基于语义相似度检索最相关的历史片段，适合海量历史长期记忆场景。
- **EntityMemory**：结构化记忆，专门提取和维护对话中的实体信息（如姓名、订单号），实现对话记录到知识图谱的转换。

## 对比速记
| Memory类型 | 核心机制 | 适用场景 | 成本控制 |
| :--- | :--- | :--- | :--- |
| **Buffer** | 保留全量历史 | 短对话、低频交互 | 低 (短对话) / 高 (长对话) |
| **BufferWindow** | 滑动窗口(保留K轮) | 成本敏感的常规聊天 | 稳定可控 |
| **Summary** | LLM定时总结摘要 | 长对话、需了解大体背景 | 中等 (需消耗摘要Token) |
| **SummaryBuffer** | 远期摘要+近期原文 | 复杂长线任务 | 兼顾效果与成本 |
| **VectorStore** | 向量语义检索 | 知识库问答、长期记忆 | 检索效率高 |
| **Entity** | 实体抽取与追踪 | 电商客服、个人助手 | 结构化程度高 |

## 代码示例
```java
// 1. 窗口Memory配置
ConversationBufferWindowMemory memory = ConversationBufferWindowMemory.builder()
    .k(5)  // 保留最近5轮对话
    .returnMessages(true) 
    .build();

// 2. 摘要Memory配置
ConversationSummaryMemory summaryMemory = ConversationSummaryMemory.builder()
    .llm(chatModel) 
    .maxTokenLimit(2000)  // 触发摘要的token阈值
    .build();

// 3. 向量检索Memory配置
VectorStoreRetrieverMemory vectorMemory = VectorStoreRetrieverMemory.builder()
    .vectorStore(vectorStore)
    .retriever(vectorStore.asRetriever(3))  // 检索top3相关片段
    .build();

// 4. Memory与Chain集成
ConversationChain chain = ConversationChain.builder()
    .llm(chatModel)
    .memory(memory)
    .build();
```

## 工程要点
- **存储介质选型**：短期高频会话状态存Redis；长期持久化记录存MySQL/PostgreSQL；向量化记忆存Pinecone/Weaviate等向量数据库。
- **高并发与防溢出**：设置TTL自动清理不活跃会话；按用户ID或会话ID进行哈希分片，实现水平扩展。
- **冷热数据分离**：将超过30天的历史对话迁移至低成本对象存储，保障系统核心响应速度。
- **容灾与降级**：当向量检索服务不可用时，降级到基础BufferMemory；外部存储异常时切换本地缓存维持基本会话。
- **分布式一致性**：优先保证可用性和分区容错性（AP），通过消息队列异步同步各节点EntityMemory数据，接受最终一致性。
