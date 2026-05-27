# 如何在LangChain中实现RAG（检索增强生成）？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- RAG（检索增强生成）通过引入外部知识库（非参数化记忆），解决传统LLM知识更新滞后和领域知识不足的问题。
- 核心流程：用户查询 -> 向量化 -> 数据库检索相关文档 -> 构建增强Prompt -> LLM生成答案。

## 机制与原理
- **文档处理**：使用DocumentLoader标准化加载多格式文档，利用TextSplitter按自然边界切分文档。
- **向量存储**：Embedding模型将文本转为高维向量，相似语义在向量空间中形成聚类，随后存入Vector Stores（如Chroma、FAISS）。
- **检索机制**：实质是在高维空间进行最近邻搜索（如余弦相似度）。通过VectorStoreRetriever配置返回文档数量（k值）和相似度阈值。
- **生成阶段**：将检索到的文档片段作为上下文注入PromptTemplate，显式约束LLM“仅基于上下文回答”以防幻觉。

## 对比速记
- **传统LLM**：依赖参数化记忆，知识固化在模型权重中，更新成本高。
- **RAG系统**：引入非参数化记忆，通过外部知识库实现知识动态更新，降低微调成本。

## 代码示例
```java
// 1. 文档切分（设置重叠度以防边界信息丢失）
TextSplitter splitter = new RecursiveCharacterTextSplitter()
    .setChunkSize(800)
    .setChunkOverlap(100);
List<Document> documents = splitter.splitDocuments(rawDocuments);

// 2. 向量化存储与检索器构建
VectorStore vectorStore = Chroma.fromDocuments(documents, new OpenAIEmbeddings());
VectorStoreRetriever retriever = vectorStore.asRetriever()
    .setSearchType("similarity")
    .setSearchKwargs(Map.of("k", 3));

// 3. 构建增强Prompt与RAG链
PromptTemplate promptTemplate = PromptTemplate.fromTemplate(
    "基于以下上下文回答问题。如无相关信息请明确说明。\n上下文：{context}\n问题：{question}");
RetrievalQA ragChain = new RetrievalQA(new LLMChain(new ChatOpenAI(), promptTemplate), retriever);
```

## 工程要点
- **参数调优**：`chunk_size`需依场景调整（简洁文本500-800，技术手册1200-1500），`chunk_overlap`保障语义连续性；`temperature`建议设为0.1-0.3以保证稳定性。
- **检索优化**：单一向量检索可能遗漏关键词，建议结合BM25算法进行混合检索，平衡语义理解与精确匹配。
- **性能架构**：百万级文档检索易成瓶颈，可采用“关键词粗筛+向量精排”分层策略，并对高频查询建立缓存。
- **技术选型**：初期轻量级验证选Chroma；高并发性能要求选FAISS+Redis；免运维云端托管选Pinecone。
- **持续运维**：需监控检索召回率与生成质量，定期分析Bad Case，并确保知识库更新时及时清理缓存。
