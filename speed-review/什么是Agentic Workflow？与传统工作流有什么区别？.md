# 什么是Agentic Workflow？与传统工作流有什么区别？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- **Agentic Workflow**：由AI代理自主决策和动态执行的工作流范式。只需给定目标，Agent会自主规划步骤、选择工具、评估结果并调整策略，执行过程动态且非线性。
- **关键特征**：
  - 依赖LLM的推理能力、工具调用和反馈循环。
  - 概率性决策，路径由Agent实时动态构建。
  - 适合处理开放式、复杂多变、需要综合判断的任务。

## 机制与原理
- **核心循环**：感知当前状态与目标 -> 推理下一步骤 -> 决策并选择行动/工具 -> 执行并评估结果。
- **ReAct模式**：Reasoning和Acting交替进行，将推理过程显性化（如：先思考需要调什么API，执行后观察结果再思考下一步）。
- **Plan-and-Execute模式**：先做全局规划列出步骤，再按计划逐步执行，适合复杂任务，但灵活性稍弱。
- **Reflection模式**：在执行中加入自我评估与反思，未达预期则调整策略重新执行。
- **三大核心能力**：LLM（推理引擎）、工具调用（操作外部系统）、记忆机制（存储历史与上下文）。

## 对比速记

| 维度 | 传统工作流 (DAG/状态机) | Agentic Workflow |
| :--- | :--- | :--- |
| **决策方式** | 确定性执行，基于预定义的if-else静态规则 | 概率性决策，基于LLM上下文推理动态选择 |
| **路径生成** | 需人工提前穷举并固化所有可能的路径 | 仅设定目标，Agent根据实时情况自主规划路径 |
| **适用场景** | 标准化、可重复、步骤固定的业务流程（如订单履约、请假审批） | 开放式、路径多变、需综合信息判断的任务（如复杂客诉、代码生成） |
| **相关技术** | 规则引擎、RPA（模拟人工操作的固定步骤）、BPM | LLM、ReAct、LangGraph、AutoGPT |

## 代码示例
```java
// Agentic Workflow的动态循环过程（伪代码示意）
public class AgentOrderHandler {
    private LLMService llm;
    private ToolRegistry tools;

    public void handleUserQuery(String userMessage) {
        String goal = "解决用户关于订单的问题: " + userMessage;
        AgentState state = new AgentState(goal, userMessage);

        while (!state.isGoalAchieved()) {
            // 1. Agent自主推理下一步行动
            String reasoning = llm.think(state.getContext());
            Action nextAction = llm.decide(reasoning, tools.available());

            // 2. 执行行动并更新状态
            ActionResult result = nextAction.execute();
            state.update(result);

            // 3. 评估是否需要继续
            state.evaluate(llm);
        }
        replyToUser(state.getFinalResponse());
    }
}
```

## 工程要点
- **可靠性保障**：设置行为边界；关键操作加入人工审核节点；记录每步推理便于回溯；连续失败时自动降级转人工。
- **成本与性能控制**：简单任务用小模型/规则引擎路由；缓存相似任务的推理路径；限制最大迭代次数防死循环；非实时任务采用异步执行。
- **系统集成**：采用渐进式集成，将Agent封装为独立API，先在边缘非核心业务试点，定义标准工具接口与现有微服务打通。
- **当前挑战**：决策过程可解释性差（不适用强监管领域）；概率性输出导致测试调试困难；多Agent协作编排缺乏成熟方法论。
