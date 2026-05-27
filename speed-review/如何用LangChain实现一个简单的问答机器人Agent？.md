# 如何用LangChain实现一个简单的问答机器人Agent？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- **基础问答机器人**：基于ConversationChain实现，依赖LLM、Prompt模板和Memory组件维持上下文。
- **Agent模式**：具备决策能力的智能体，能根据用户问题动态判断并调用外部工具（如搜索、计算器）。
- **RAG模式**：检索增强生成，先从VectorStore检索知识库文档，再让LLM基于检索内容回答，适合企业内部问答。

## 机制与原理
- **设计哲学**：将LLM调用过程标准化，使复杂的AI能力可以像搭积木一样组合使用。
- **数据流转**：标准的管道模式，即 用户输入 -> Prompt模板格式化 -> Memory注入上下文 -> LLM推理 -> Output Parser输出结构化结果 -> Memory更新状态。
- **Memory机制**：上下文管理的核心。
  - `ConversationBufferMemory`：保存全部历史，适合短会话。
  - `ConversationSummaryMemory`：定期总结历史，解决Token长度限制。
  - `ConversationBufferWindowMemory`：仅保留最近几轮对话，平衡性能与连贯性。
- **Tool集成**：定义标准函数调用协议。每个Tool包含`name`（标识）、`description`（供Agent判断何时调用）和`run`（执行操作）。

## 对比速记
- **Chain vs Agent**
  - **Chain**：预定义处理流程，执行路径固定（如：检索→排序→生成→格式化）。
  - **Agent**：具备决策能力，根据问题内容动态选择工具，执行路径由推理过程决定。

## 代码示例
```java
// Agent模式核心实现
public class SmartAgent {
    private AgentExecutor agent;

    public void setupAgent() {
        OpenAI llm = OpenAI.builder()
            .apiKey(System.getenv("OPENAI_API_KEY"))
            .temperature(0.3) // Agent推理场景建议较低temperature保证稳定
            .build();

        // 定义工具集
        List<Tool> tools = Arrays.asList(
            Tool.builder()
                .name("search")
                .description("搜索商品信息，输入商品关键词，返回商品列表")
                .func(keyword -> productService.search(keyword))
                .build()
        );

        // 初始化Agent
        this.agent = AgentExecutor.fromAgentAndTools()
            .agent(ZeroShotReactDescription.create(llm, tools))
            .tools(tools)
            .maxIterations(3) // 防止无限循环
            .build();
    }
}
```

## 工程要点
- **超时与异常降级**：LLM调用需设置超时时间（如30秒），捕获异常并降级到基于关键词匹配的模板回复。
- **并发控制**：使用信号量（Semaphore）限制LLM的并发请求数，防止瞬间耗尽API配额。
- **多级缓存**：使用本地缓存（如Caffeine）缓存相同或相似问题的回答，减少重复调用开销。
- **边界与安全**：在Prompt中明确能力边界，建立置信度评估机制，对退款、投诉等敏感或低置信度问题主动转接人工。
- **大规模扩展**：通过消息队列处理非实时请求，部署本地化小模型处理简单问题，复杂场景才调用云端大模型以控制成本。
