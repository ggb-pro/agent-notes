# LangChain的核心组件有哪些？各自的作用是什么？

> **难度**: 简单 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- LangChain是一个AI应用开发框架，其组件设计遵循分层架构思想，从底层模型到上层应用，职责边界明确。
- 核心组件包括：Models、Prompts、Chains、Memory、Agents、Tools、Vector Stores与Retrievers。

## 机制与原理
- **Models**：提供统一抽象层，屏蔽不同大模型厂商的接口差异，使开发者无需为切换模型重写业务代码。
- **Prompts**：实现提示工程的标准化与模板化，将业务逻辑与提示内容分离，支持动态参数化生成。
- **Chains**：体现分治思想，将复杂任务拆解为可管理的小步骤并串联，各节点输入输出明确，便于调试与复用。
- **Memory**：解决多轮对话的状态管理问题，支持滑动窗口、摘要提取等不同的上下文保持策略。
- **Agents**：具备自主决策能力，能根据当前状态和目标动态选择执行路径与工具，实现从“流程自动化”到“智能决策”的升级。
- **Vector Stores & Retrievers**：处理向量存储与文档检索，是RAG架构的核心，支持从海量文档中快速检索相关信息。

## 对比速记
- **LangChain vs LlamaIndex**：LangChain强调组件化与可编排性，类似乐高积木系统；LlamaIndex专注于数据索引与检索，在RAG场景优化更深。
- **Chain vs Agent**：Chain是确定性的预设工作流，适合需要稳定可靠输出的场景；Agent具备动态推理能力，适合复杂决策但具不可预测性。

## 代码示例
```java
// 典型的RAG智能客服系统核心链路实现
public class CustomerServiceChain {
    private VectorStore productKnowledge;
    private ConversationBufferMemory chatMemory;
    private PromptTemplate responseTemplate;
    private BaseLLM llm;

    public String handleQuery(String userQuery, String sessionId) {
        // 1. 检索相关商品信息
        List<Document> relevantDocs = productKnowledge.similaritySearch(userQuery, 3);
        // 2. 获取对话历史
        String chatHistory = chatMemory.getHistory(sessionId);
        // 3. 构建完整提示
        String prompt = responseTemplate.format(Map.of(
            "query", userQuery,
            "context", relevantDocs.toString(),
            "history", chatHistory
        ));
        // 4. 模型生成与记忆更新
        String response = llm.generate(prompt);
        chatMemory.addMessage(sessionId, userQuery, response);
        return response;
    }
}
```

## 工程要点
- **Memory策略选择**：长对话场景需避免直接使用BufferMemory导致超出上下文限制，应采用SummaryMemory提取摘要。
- **检索性能优化**：单一向量检索效果有限，推荐采用“粗筛（如按类别）+ 语义相似度匹配”的分层检索策略。
- **优雅降级机制**：必须设计容错方案，如向量检索失败时自动降级为关键词匹配，模型生成异常时返回预设标准回复。
- **自定义组件设计**：需实现标准基础接口以保证生态集成，同时内置配置、监控埋点与降级策略，确保生产环境稳定性。
