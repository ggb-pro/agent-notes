# 什么是Plan-and-Execute模式？如何实现任务的分层规划？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- Plan-and-Execute是一种将复杂任务分解为规划和执行两个阶段的Agent架构模式。
- Planner负责集中生成结构化执行计划，Executor按步骤机械执行，从而降低大模型处理复杂任务时的推理负担与Token消耗。

## 机制与原理
- **工作流**：用户目标输入 -> Planner生成任务依赖图 -> Executor按序逐步调用工具 -> 汇总输出最终结果。
- **分层规划**：本质是递归分解机制。将复杂子任务作为新目标再次触发规划，形成树状任务层级，直到叶子节点成为可直接执行的原子操作。
- **反馈循环**：Executor在执行异常（如API失败、数据格式错误）时将信息反馈给Planner，触发局部重试或动态修正后续计划。
- **设计哲学**：通过职责分离管理复杂度，带来更强的可控性（进度可追踪）、可解释性（计划可预览）和可测试性（规划与执行可独立测试）。

## 对比速记
- **Plan-and-Execute vs ReAct**：
  - ReAct是“走一步看一步”，每步都要调用LLM推理，Token消耗大，适合探索性任务。
  - Plan-and-Execute是“一次性想清楚路径再执行”，大幅减少LLM调用频率，适合步骤确定、逻辑清晰的复杂任务。
- **Plan-and-Execute vs 传统工作流引擎**：
  - 传统工作流的流程是开发者预先写死的，缺乏动态适应能力。
  - Plan-and-Execute的规划由大模型动态生成，能根据上下文和异常反馈实时调整计划。

## 代码示例
```java
// 分层规划的核心递归实现
public class HierarchicalPlanner extends Planner {
    private static final int MAX_DEPTH = 3;

    @Override
    public List<Task> generatePlan(String userGoal) {
        return generatePlanRecursive(userGoal, 0);
    }

    private List<Task> generatePlanRecursive(String goal, int depth) {
        List<Task> tasks = super.generatePlan(goal);
        if (depth >= MAX_DEPTH) {
            return tasks;  // 达到最大深度，停止递归
        }

        List<Task> expandedTasks = new ArrayList<>();
        for (Task task : tasks) {
            if (task.needsDecomposition()) {
                // 复杂任务继续分解
                List<Task> subTasks = generatePlanRecursive(task.getDescription(), depth + 1);
                // 如果父任务有依赖，第一个子任务继承这些依赖
                if (!subTasks.isEmpty() && !task.getDependencies().isEmpty()) {
                    subTasks.get(0).getDependencies().addAll(task.getDependencies());
                }
                expandedTasks.addAll(subTasks);
            } else {
                expandedTasks.add(task);
            }
        }
        return expandedTasks;
    }
}
```

## 工程要点
- **结构化输出**：通过Prompt引导或Function Calling机制，让LLM输出包含任务ID、工具名称、依赖关系（dependencies）的JSON格式计划。
- **依赖调度**：Executor需通过拓扑排序（如Kahn算法）解析任务依赖图，无依赖的任务可并行执行，提升执行效率。
- **参数解析**：支持在任务参数中使用表达式（如 `${task_1.output}` ）动态引用前置任务的执行结果。
- **粒度控制**：分解粒度是Trade-off，建议设定最大递归深度（如3层），确保叶子节点能通过一次工具调用完成。
- **容错与成本**：
  - 避免规划过度细化，防止稍有偏差导致整个长计划失效。
  - 设定单次任务的最大重试次数与超时时间，限制最大重新规划次数（如3次）以防成本失控。
- **分布式扩展**：在多节点部署时，建议使用Redis等中心化存储维护任务状态，结合消息队列进行任务调度，并设计失败接管机制。
