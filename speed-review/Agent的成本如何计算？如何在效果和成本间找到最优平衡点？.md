# Agent的成本如何计算？如何在效果和成本间找到最优平衡点？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- **Agent成本**：由LLM API调用、外部工具调用、存储及计算资源消耗共同构成的系统级开销。
- **成本与效果平衡点**：基于经济学边际分析，在任务完成率、准确率等效果指标与系统运行成本之间寻找ROI（投资回报率）最高的配置拐点。

## 机制与原理
- **LLM调用成本**：遵循公式 `成本 = (输入Token数 × 输入单价) + (输出Token数 × 输出单价)`。Agent的多轮迭代会导致历史上下文不断累积，造成Token消耗呈指数级膨胀。
- **工具调用成本**：包含外部API按请求次数、返回条数或并发计算的计费。因参数错误重试或缺乏去重逻辑导致的无效调用，通常可占总成本的20%-30%。
- **存储与计算成本**：包含对话历史持久化、向量数据库维护（RAG）、缓存开销，以及高并发场景下容器集群和冷启动带来的云资源消耗。
- **边际收益递减效应**：从70%提升至85%的任务完成率成本较低；但从85%提升至95%往往需要更换强模型并增加迭代轮次，成本可能翻三倍以上。
- **全链路监控机制**：在请求入口、模型调用层、工具调用层埋点。区分系统提示词、历史上下文、当前输入的Token占比，并实时监控单次对话成本、P95异常消耗等指标。

## 对比速记
| 业务场景 | 效果与成本平衡策略 | 核心特征 |
| :--- | :--- | :--- |
| **智能客服** | 分层路由与保底策略 | 80%简单问题走规则/弱模型，复杂问题路由至强模型，偏向控制平均成本。 |
| **代码生成** | 效果优先，多轮迭代 | 准确度要求极高（代码必须能跑通），允许增加迭代和测试修正轮次，用户对价格敏感度低。 |
| **数据分析** | 前重后轻策略 | 首轮用强模型做深度理解与规划，后续具体SQL执行与数据处理交由脚本或弱模型完成。 |

## 代码示例
```java
// 混合模型路由策略：基于意图分类和置信度动态选择模型
public class AgentRouter {
    private IntentClassifier intentClassifier;
    private LightModel lightModel;

    public ModelType routeToModel(String userQuery, ConversationContext context) {
        IntentType intent = intentClassifier.classify(userQuery);

        if (intent.isSimple()) {
            return ModelType.RULE_ENGINE;  // 简单任务：规则引擎处理
        }

        if (intent.isModerate()) {
            String response = lightModel.generate(userQuery, context);
            double confidence = calculateConfidence(response);
            // 中等任务：根据弱模型置信度决定是否升级
            return confidence > 0.8 ? ModelType.GPT_35 : ModelType.GPT_4;
        }

        return ModelType.GPT_4; // 复杂任务：直接使用强模型
    }
}
```

## 工程要点
- **Prompt瘦身**：删除冗余描述，仅保留角色定义和输出格式。精选2个高质量Few-shot示例的效果通常不亚于5个平庸示例，但Token消耗减半。
- **CoT控制**：对简单任务关闭链式思考或限制最大思考长度，可减少约30%的Token消耗，准确率影响极小。
- **熔断与限流**：设置单会话成本上限和迭代次数硬顶（如复杂任务上限4轮），超限强制中断转人工，防止单点异常烧光预算。
- **A/B测试验证**：通过Hash分流对比不同配置，确保除变量（如模型选择）外其他条件一致，并做分层采样防止样本复杂度不均。
- **动态扩展性**：路由策略应支持动态配置，在业务高峰期自动降级模型档位，低峰期提升体验。
