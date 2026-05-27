# LangGraph的核心概念有哪些？图、节点、边的作用是什么？

> **难度**: 简单 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- **LangGraph**：基于图结构化的工作流编排框架，实现从命令式编程向声明式编程的转变，专为AI推理场景的不确定性设计。
- **图**：执行框架的容器，负责全局流程控制、状态持久化和回滚机制。
- **节点**：具体的处理单元（如LLM调用、数据处理），本质上是纯函数，接收当前状态并输出新的不可变状态。
- **边**：节点间的连接关系与智能状态分析器，负责流转规则和条件判断，实现业务逻辑与流程控制解耦。

## 机制与原理
- **分层设计**：图层面做全局控制，节点层面专注业务逻辑，边层面处理流转规则，职责边界明确。
- **状态驱动**：状态是开放式的数据结构，支持携带任意复杂信息，且状态的转换可以依赖AI模型的推理结果。
- **动态路由**：条件边会检查当前状态的特定字段（如置信度、风险等级），根据预设条件函数动态决定下一个执行节点。
- **多Agent协作**：通过统一的状态对象同步信息，支持子图状态隔离，实现多Agent既保持独立性又能协同工作。

## 对比速记
- **vs 传统状态机**：传统状态机只能在预设状态间切换（类似开关控制）；LangGraph支持开放式数据结构，能根据AI理解的语义动态调整行为策略。
- **vs 传统工作流（如Airflow）**：传统工作流面向数据处理，路径确定；LangGraph面向AI推理场景，每一步结果都可能影响后续路径，高度支持动态决策。
- **vs LangChain**：LangChain是基础组件库，解决“如何调用AI能力”；LangGraph是组件编排引擎，解决“如何组织AI工作流”。

## 代码示例
```java
// 状态对象设计（支持不可变性）
public class CustomerServiceState {
    private String userQuery;
    private String intentType;
    private Double confidenceScore;
    private String errorMessage;
    private boolean needsRetry;
    // 使用Builder模式保证状态的不可变性
    public CustomerServiceState toBuilder() { return new Builder(this); }
}

// 意图识别节点（纯函数：输入状态 -> 输出新状态）
public CustomerServiceState intentRecognition(CustomerServiceState state) {
    try {
        String intent = aiModel.classify(state.getUserQuery());
        return state.toBuilder().intentType(intent).confidenceScore(calculateConfidence(intent)).build();
    } catch (Exception e) {
        return state.toBuilder().errorMessage("识别失败: " + e.getMessage()).needsRetry(true).build();
    }
}

// 条件边的路由逻辑（数据驱动）
public String routeByConfidence(CustomerServiceState state) {
    if (state.getErrorMessage() != null) return "error_handling";
    if (state.getConfidenceScore() > 0.8) return "direct_answer";
    else if (state.getConfidenceScore() > 0.5) return "knowledge_search";
    else return "human_transfer";
}
```

## 工程要点
- **状态轻量化**：状态对象在节点间传递时只保留核心标识（如只传商品ID列表），具体详情由对应节点按需查询，避免冗余数据传输。
- **双层异常处理**：节点级异常通过try-catch捕获并写入状态对象供后续处理；图级别异常需设计专门的错误处理节点和全局回滚机制。
- **拓扑设计**：先梳理业务流程中的决策点（对应条件边），再识别可并行与必须串行的步骤，最终确定图的拓扑结构。
