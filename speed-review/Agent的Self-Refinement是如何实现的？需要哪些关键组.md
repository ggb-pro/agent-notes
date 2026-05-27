# Agent的Self-Refinement是如何实现的？需要哪些关键组件？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- **定义**：Agent通过自我反馈循环来迭代优化输出结果，将“生成-批判-改进”的闭环内化到执行流程中。
- **核心前提**：模型本身需具备基础的生成与评估能力。
- **适用场景**：有明确质量标准但单次生成难达标的任务（如代码生成、复杂推理、文案创作），简单任务（如查天气）不适用。

## 机制与原理
- **三大组件**：通常用同一个LLM切换不同Prompt角色实现，以控制成本。
  - **Generator（生成器）**：快速生成初始候选答案。
  - **Critic（评估器）**：扮演挑剔的评审，给出结构化反馈（如具体问题位置、改进建议）和评分，而非模糊评价。
  - **Refiner（改进器）**：带着明确的改进目标和上一轮的上下文，针对性修正而非全盘重写。
- **终止策略（双保险）**：
  - **质量阈值**：Critic综合评分达到设定阈值（如0.85）即停止。
  - **工程保底**：设置最大迭代次数（通常2-3轮），超过未达标则降级处理（如人工介入）。

## 对比速记
- **vs Fine-tuning / RLHF**：Self-Refinement无需人工标注数据或训练模型，利用模型自身能力闭环，适合快速迭代。
- **vs 多次独立调用**：独立调用是靠随机性碰运气；Self-Refinement是包含前文诊断的有向迭代。
- **vs Self-Consistency**：Self-Consistency是生成多个答案投票选最优（广度）；Self-Refinement是对单条路径进行深度优化。

## 代码示例
```java
public class SelfRefinementAgent {
    private final LLMClient llmClient;
    private final int maxIterations = 3;
    private final double qualityThreshold = 0.85;

    public String execute(String task) {
        String output = generate(task);
        for (int i = 0; i < maxIterations; i++) {
            CriticResult evaluation = critique(output, task);
            // 达标或无具体问题则终止
            if (evaluation.getScore() >= qualityThreshold || evaluation.getIssues().isEmpty()) {
                break;
            }
            // 带着反馈重新生成
            output = refineWithFeedback(output, evaluation.getIssues());
        }
        return output;
    }

    private CriticResult critique(String output, String task) {
        // Prompt设定：严格评审专家，规定评估维度（准确性、完整性、健壮性）
        // 强制要求输出结构化JSON：{"score": 0.0-1.0, "issues": ["具体问题1"]}
        // ...(调用llmClient并解析JSON)
    }

    private String refineWithFeedback(String previousOutput, List<String> issues) {
        // Prompt设定：保留合理部分，仅针对具体问题修正，不全盘重写
        // ...(调用llmClient)
    }
}
```

## 工程要点
- **Prompt设计**：Critic的Prompt必须包含具体检查项（如SQL注入风险、空指针判断），引导其指出具体位置和改进建议。
- **避免过拟合**：迭代次数通常不超过3轮，第4轮易出现为迎合评分而改出新问题的情况。
- **成本控制**：可缓存前几轮输出用于相似任务；能用规则检查（如语法校验）完成的评估，优先于LLM调用。
- **引入外部工具**：对于可客观验证的任务（如代码/SQL生成），结合沙箱测试或单元测试作为客观评分，突破模型自身评估能力的天花板。
