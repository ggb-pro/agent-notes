# ReWOO（Reasoning WithOut Observation）模式与ReAct有什么区别？各自的适用场景？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- **ReAct**：交替式执行模式，形成“思考-行动-观察”的串行循环，每步依赖上一步的实际观察结果。
- **ReWOO**：规划与执行分离模式，先一次性生成完整计划和工具调用序列，批量执行后统一求解。

## 机制与原理
- **ReAct机制**：LLM生成Thought决定下一步 -> 执行Action调用工具 -> 获取Observation -> 将结果喂给LLM继续思考。每次行动必须等待LLM处理结果，存在串行性能瓶颈。
- **ReWOO机制**：包含三个核心组件：Planner（一次性规划带依赖关系的执行计划）、Worker（并行调用工具）、Solver（基于全部结果给出最终答案）。中间工具调用无需LLM参与。
- **性能数据**：在HotpotQA等多步推理测试中，ReAct平均调用6-8次大模型，ReWOO仅需2次（1次规划+1次总结）。API调用成本ReWOO通常仅为ReAct的1/3。

## 对比速记
| 维度 | ReAct | ReWOO |
| :--- | :--- | :--- |
| **执行方式** | 串行交替（思考-行动-观察） | 规划与执行分离，支持并行调用 |
| **LLM调用次数** | 多（6-8次及以上） | 少（通常2次） |
| **延迟与成本** | 高（串行累加，Token消耗大） | 低（并行执行，总体吞吐量高） |
| **灵活性与容错** | 高，可基于中间结果动态调整策略 | 低，初始规划有误或工具异常时难以纠正 |
| **适用场景** | 探索性推理、多轮交互、不确定性高的任务 | 流程清晰、工具调用路径可预测的标准化任务 |

## 代码示例
```java
public class AgentRouter {
    public AgentResponse process(UserQuery query) {
        QueryIntent intent = intentClassifier.classify(query);

        if (intent.isPredictable() && intent.getComplexity() < 5) {
            try {
                // 可预测任务：尝试ReWOO模式
                AgentResponse response = new ReWOOAgent().execute(query);
                if (response.getConfidence() > 0.8) return response;
            } catch (ToolExecutionException e) {
                // 工具异常降级
            }
            return fallbackToReAct(query); // 质量不达标降级
        }
        // 不确定性高的任务直接走ReAct
        return new ReActAgent().execute(query);
    }
}
```

## 工程要点
- **动态路由与降级**：通过意图识别路由执行模式。ReWOO执行中若工具失败或置信度低，应自动降级切换至ReAct重新执行。
- **缓存策略**：ReWOO可对同类任务缓存执行计划（仅替换参数）；ReAct可缓存具体工具的返回结果（如短期内不变的查询数据）。
- **重试机制**：ReWOO重新规划成本高，仅在底层工具调用层面重试；ReAct可让LLM基于失败信息重新思考并更换执行方向。
- **混合架构**：实际项目常混合使用，如先用ReWOO快速检查标准项，发现异常再切ReAct深度排查。
- **指标权衡**：Token消耗、延迟、准确率构成不可能三角。需根据业务核心诉求（如金融重准确率，推荐重延迟与成本）做技术选型。
