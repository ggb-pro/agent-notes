# Agent在执行过程中如何进行动态重规划？触发条件有哪些？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- Agent的动态重规划是在执行过程中检测到当前计划不可行或低效时，重新生成执行路径的自适应机制。
- 赋予Agent容错性和环境适应能力，避免在错误路径上持续消耗资源。

## 机制与原理
- **闭环控制循环**：标准执行流程为 Planning -> Execute -> Observe -> Reflect 循环，重规划发生在 Reflect 阶段发现问题之后。
- **信息差异**：初始规划基于静态任务描述和理想假设；重规划基于真实的执行反馈，信息更充分但面临已消耗部分资源的压力。
- **触发条件分层**：
  - **执行异常**：区分偶发异常（如网络抖动，应重试）与系统性失败（如API不存在或参数错误，必须重规划）。
  - **状态偏离**：Observe阶段发现环境状态与Planning时的假设不一致（如商品已下架），导致前置条件失效。
  - **资源约束**：Token预算即将耗尽或时间窗口不足，必须调整策略以保证在限制内完成。
  - **效率低下**：当前策略虽能运行但过于低效，主动优化执行路径。
- **决策策略**：
  - **局部修正**：问题孤立，仅调整失败节点及其下游依赖，保留有效中间结果。快速低成本，但可能治标不治本。
  - **全局重规划**：前提假设崩塌，完全抛弃原计划重新生成。能找到更优解，但成本高且浪费已有进展。
  - **混合策略**：先尝试局部修正，修正N次失败后再触发全局重规划。

## 对比速记
| 框架/模式 | 重规划机制 | 适用场景 |
| :--- | :--- | :--- |
| **ReAct** | 隐式重规划。将历史Thought-Action-Observation喂给LLM，在下一轮Thought中分析失败并生成替代Action。 | 交互式任务，需根据反馈灵活调整（如客服对话）。 |
| **AutoGPT** | 显式重规划。维护动态任务队列，失败任务被重新分解、调整优先级或插入新任务。 | 目标明确但路径复杂、多依赖步骤的任务（如数据分析）。 |

## 代码示例
```java
// 生产环境安全重规划器：包含次数上限、冷却时间与策略去重机制
public class SafeReplanner {
    private static final int MAX_REPLAN_COUNT = 3;
    private static final long REPLAN_COOLDOWN_MS = 5000;

    private int replanCount = 0;
    private long lastReplanTime = 0;
    private Set<String> triedStrategies = new HashSet<>();

    public Optional<Plan> replan(Plan currentPlan, ErrorObservation error, State currentState) {
        // 1. 检查重规划次数限制
        if (replanCount >= MAX_REPLAN_COUNT) {
            return Optional.empty();
        }
        // 2. 检查冷却时间，防止过度重规划导致震荡
        long now = System.currentTimeMillis();
        if (now - lastReplanTime < REPLAN_COOLDOWN_MS) {
            return Optional.empty();
        }

        Plan newPlan = generateNewPlan(currentPlan, error, currentState);

        // 3. 策略去重：计算特征签名，避免陷入死循环
        String strategySignature = computeStrategySignature(newPlan);
        if (triedStrategies.contains(strategySignature)) {
            return Optional.empty();
        }

        replanCount++;
        lastReplanTime = now;
        triedStrategies.add(strategySignature);
        return Optional.of(newPlan);
    }
}
```

## 工程要点
- **异常转换**：用try-catch包裹Action，将异常封装为Agent能理解的ErrorObservation，避免整个执行流程崩溃。
- **性能开销**：重规划不仅消耗LLM推理时间，还会打断执行流导致中间计算作废。应通过前置检查减少触发频率，或预设备选方案加快切换速度。
- **用户体验连贯性**：重规划可能导致Agent行为突变（如突然改口），对外交互需设计合理的解释逻辑，避免用户困惑。
- **人机协作边界**：涉及资金转账等敏感操作时，重规划前应加入人工审批；连续重规划失败达到上限时，应升级给人类处理。
