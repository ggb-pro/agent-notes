# 如何设计Agent的反思（Reflection）机制？何时触发反思？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- Agent的反思机制是嵌入执行流程中的元认知层，用于评估自身行为质量并动态调整策略。
- **三要素**：反思对象（单次/多步结果）、评估标准（格式/逻辑/目标契合度）、改进动作（重试/修正/切换策略）。
- **本质区别**：错误处理是被动响应异常（如API失败重试），反思是主动评估决策质量（即使执行成功也评估是否真正解决问题）。

## 机制与原理
- **架构模块**：由触发器（规则/模型驱动）、评估器（LLM/预定义规则分析轨迹）、记忆更新器（沉淀经验）、策略调整器（决定下步动作）四个核心模块构成。
- **触发时机**：
  - **被动触发**：明确异常信号（工具失败、格式报错、用户负面反馈）。
  - **主动触发**：无明确错误但需定期检查（多步任务阶段性回顾、置信度低于阈值）。
- **反思粒度**：
  - **单步反思**：关注单次工具调用或生成质量（如参数合理性）。
  - **任务级反思**：关注整体链路完成度与效率（如是否偏离最终目标）。
  - **长期经验总结**：跨任务周期沉淀高频规则（如构建“场景-策略-结果”知识图谱）。
- **记忆存储**：短期记忆存入当前会话上下文；长期记忆持久化至向量数据库或提炼为if-then规则库。
- **协同机制**：反思是CoT（推理透明化）和ReAct（推理执行交替）之上的元层能力，负责评估二者的质量并指导下轮迭代。

## 对比速记
| 机制 | 核心作用 | 关注点 |
| :--- | :--- | :--- |
| **错误处理** | 被动响应系统异常 | API超时、网络断连等技术故障 |
| **CoT** | 显式化推理过程 | 推理步骤的透明度与逻辑性 |
| **ReAct** | 推理与行动结合 | 思考与工具调用的交替循环 |
| **反思** | 主动评估决策质量 | 结果是否达标、策略是否最优、经验沉淀 |

## 代码示例
```java
class ReflectionEngine {
    private LLMClient llmClient;
    private int maxReflectionCount = 3; // 防止死循环

    // 1. 触发条件判断
    public boolean shouldReflect(StepResult result) {
        return !result.isSuccess() || result.getConfidenceScore() < 0.7 || result.getOutput() == null;
    }

    // 2. 执行反思评估
    public ReflectionResult performReflection(String taskGoal, StepResult stepResult) {
        String prompt = String.format(
            "任务目标: %s\n执行动作: %s\n执行结果: %s\n成功状态: %s\n置信度: %.2f\n" +
            "请评估是否达成目标、原因及改进建议。\n" +
            "返回JSON: {\"achieved\": boolean, \"issue\": \"...\", \"suggestion\": \"...\"}",
            taskGoal, stepResult.getAction(), stepResult.getOutput(), stepResult.isSuccess(), stepResult.getConfidenceScore()
        );
        String llmResponse = llmClient.chat(prompt);
        return parseReflectionResponse(llmResponse); // 解析为包含下一步动作(RETRY/REPLAN等)的对象
    }

    // 3. 带反思的任务执行主循环
    public Object executeWithReflection(String taskGoal, Task task) {
        int count = 0;
        while (count < maxReflectionCount) {
            StepResult result = task.execute();
            if (!shouldReflect(result)) return result.getOutput(); // 质量达标直接返回
            
            ReflectionResult reflection = performReflection(taskGoal, result);
            adjustTask(task, reflection); // 根据反思调整任务
            count++;
        }
        throw new RuntimeException("反思次数超限，任务失败");
    }
}
```

## 工程要点
- **死循环防范**：设计明确的收敛条件（如量化目标差距），若连续两次反思无改善则强制终止；引入多维度评估体系避免陷入局部最优。
- **成本控制**：采用分层反思策略，第一层低成本规则预检（校验状态码/格式），通过后再进入第二层高成本LLM深度分析；仅对高风险/关键节点启用。
- **效果评估**：通过A/B实验对比任务成功率与平均步数；监控“反思改进成功率”（触发反思且带来正向收益的比例）以调优触发阈值；追踪长期经验复用率。
