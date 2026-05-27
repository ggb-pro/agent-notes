# 什么是ReAct模式？它如何提升AI Agent的推理能力？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- ReAct（Reasoning + Acting）是一种让AI Agent交替进行推理和行动的框架模式。
- 核心机制是让Agent在每个步骤中经历“思考-行动-观察”的循环，直到完成任务。

## 机制与原理
- **推理指导行动**：Agent分析当前情况并规划下一步动作（Reasoning负责“想什么”）。
- **行动改变环境**：执行具体操作（如调用API、查询数据库），获取新信息（Acting负责“做什么”）。
- **观察反馈闭环**：处理行动结果并更新上下文，基于新信息重新评估并调整策略。
- **动态信息获取**：Agent可根据推理需要主动获取外部信息，而非仅依赖初始输入。
- **错误自我纠正**：当结果不符预期时，Agent能重新推理并调整策略。

## 对比速记
- **ReAct vs Chain-of-Thought (CoT)**：CoT是基于初始信息的纯内部思维链推理（类似“闭卷考试”）；ReAct可在推理中主动介入环境获取新信息（类似“开卷考试”）。

## 代码示例
```java
public class ReActAgent {
    private final ThoughtEngine thoughtEngine;
    private final ActionExecutor actionExecutor;
    private final ObservationProcessor observer;

    public AgentResponse solve(String problem) {
        ThoughtContext context = new ThoughtContext(problem);
        int maxIterations = 10;

        for (int i = 0; i < maxIterations; i++) {
            // 推理阶段：分析当前情况，规划下一步
            ThoughtResult thought = thoughtEngine.reason(context);

            if (thought.isTaskComplete()) {
                return new AgentResponse(thought.getFinalAnswer());
            }

            // 行动阶段：执行具体操作
            ActionResult actionResult = actionExecutor.execute(thought.getPlannedAction());

            // 观察阶段：处理行动结果，更新上下文
            context = observer.processResult(actionResult, context);
        }

        return new AgentResponse("任务超时，无法完成");
    }
}
```

## 工程要点
- **Action空间设计**：需平衡能力完整性与复杂度可控性。每个Action要有清晰的输入输出定义和执行条件。
- **观察反馈设计**：不能仅返回API调用结果，应包含结果置信度、数据完整性评估及下一步行动建议。
- **循环与成本控制**：需设置最大循环次数、引入循环检测机制以防止死循环；同时需在任务完成准确率与大模型调用成本之间取得平衡。
