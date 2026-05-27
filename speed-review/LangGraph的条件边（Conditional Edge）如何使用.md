# LangGraph的条件边（Conditional Edge）如何使用？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- 条件边用于根据节点执行结果动态决定下一个执行路径，实现工作流的智能分支。
- 核心机制是通过 `add_conditional_edges()` 方法，接收三个关键参数：源节点、条件函数和目标节点映射。

## 机制与原理
- **执行阶段**：分为状态获取、条件判断和路径选择。源节点执行完毕后，LangGraph将最新图状态传递给条件函数。
- **状态依赖**：条件函数本质是一个纯函数，不修改状态，仅读取状态中的关键信息做出路由决策。
- **异常保护**：条件函数必须返回在目标映射中存在的键，否则会抛出异常，以确保图的完整性和执行的可预测性。

## 对比速记
- **普通边**：静态路由，在图构建时就固定了执行路径。
- **条件边**：动态路由，在运行时才根据实际数据内容（如AI模型输出、用户输入类型）决定走向。

## 代码示例
```java
// 图构建时添加条件边
graph.addConditionalEdges(
    "input_node",
    this::routeMessage, // 条件函数：接收状态，返回字符串键值
    Map.of(
        "qa_node", "question_handler",    // 映射：键 -> 目标节点
        "chat_node", "casual_chat",
        "default_node", "fallback_handler"
    )
);

// 条件函数示例
public String routeMessage(GraphState state) {
    String userInput = state.getUserInput();
    if (userInput.contains("问题")) return "qa_node";
    else if (userInput.contains("闲聊")) return "chat_node";
    else return "default_node";
}
```

## 工程要点
- **职责解耦**：前置节点负责意图识别等重计算，并将结果存入状态；条件函数仅负责查表或简单比较，避免成为性能瓶颈。
- **健壮性设计**：必须为异常情况（如状态数据缺失、无效输入）提供合理的默认路由分支，实现优雅降级。
- **严格测试**：开发阶段需覆盖条件函数所有可能的返回值，确保每个返回值在路径映射中都有对应的目标节点。
