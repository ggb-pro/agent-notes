# 什么是层级式Multi-Agent架构？Manager-Worker模式如何实现？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- **定义**：Manager Agent负责任务分解、规划和结果聚合，Worker Agent专注执行具体子任务的星型协作架构。
- **特征**：
  - Manager接收请求，分析复杂度并拆解子任务，分配给不同能力的Worker并行执行。
  - Worker之间相对独立，不直接通信，所有协调均通过Manager中转。
  - 复杂场景下可引入多层Manager形成树状结构（如顶层管模块拆分，子层管文件调度）。

## 机制与原理
- **任务生命周期**：任务接收 -> 规划阶段（LLM生成执行计划） -> 分发阶段 -> 并行执行 -> 聚合阶段（冲突解决与格式统一）。
- **Worker注册机制**：Worker启动时通过接口暴露能力边界（如skillType、支持操作列表、最大并发数），形成Worker注册表供Manager查询。
- **动态规划（大脑）**：Manager将用户请求和可用Worker能力描述拼接为Prompt，利用LLM推理生成包含依赖关系的JSON执行计划。
- **结果聚合与容错**：Manager收集子结果进行一致性检查与合并，支持配置容错阈值（如允许20%失败率）实现降级输出。

## 对比速记
- **层级式 vs 平面式**：
  - **层级式**：星型结构，通信复杂度O(N)，Manager中心决策，适合任务边界清晰、可明确拆解的场景（如工作流自动化）。
  - **平面式**：网状结构，通信复杂度O(N²)，Agent对等协商，适合需要高频协商、反复迭代收敛的任务（如多智能体博弈）。
- **Manager-Worker vs 微服务**：微服务是业务能力的垂直拆分且去中心化；Manager-Worker是任务执行的分工模式，Worker通常无状态，由Manager集中控制。
- **Manager-Worker vs MapReduce**：MapReduce处理同质批处理作业，强调数据分片；Manager-Worker处理异构任务，存在复杂的依赖关系，应用场景更通用。

## 代码示例
```java
// Manager核心调度逻辑：并发分发与结果聚合
public class ManagerAgent {
    private Map<String, WorkerCapability> workerRegistry = new ConcurrentHashMap<>();
    private TaskPlanner planner;
    private ExecutorService executorService;

    public String handleRequest(String userRequest) {
        // 1. 调用LLM生成执行计划
        ExecutionPlan plan = planner.generatePlan(userRequest, workerRegistry);

        // 2. 线程池并发执行子任务
        List<Future<TaskResult>> futures = new ArrayList<>();
        for (SubTask subTask : plan.getSubTasks()) {
            futures.add(executorService.submit(() -> {
                Worker worker = findBestWorker(subTask.getRequiredSkill());
                return worker.execute(subTask);
            }));
        }

        // 3. 结果聚合与容错处理
        return aggregateResults(futures, plan);
    }

    private String aggregateResults(List<Future<TaskResult>> futures, ExecutionPlan plan) {
        List<TaskResult> results = new ArrayList<>();
        int successCount = 0;
        for (Future<TaskResult> future : futures) {
            try {
                TaskResult result = future.get(30, TimeUnit.SECONDS);
                results.add(result);
                if (result.isSuccess()) successCount++;
            } catch (Exception e) {
                results.add(TaskResult.failed(e.getMessage()));
            }
        }
        // 容错阈值：允许部分失败
        if (successCount < futures.size() * 0.8) {
            return "任务执行失败，成功率过低";
        }
        return mergeResults(results, plan); // 冲突解决与合并
    }
}
```

## 工程要点
- **解除性能瓶颈**：避免Manager同步阻塞，引入消息队列解耦，Manager将子任务投入队列后快速返回，Worker异步拉取执行。
- **并发与限流控制**：针对调用外部API的Worker，必须使用信号量（Semaphore）等机制限制并发度，防止触发上游限流。
- **异常处理机制**：局部容错（失败后尝试分配给同类冗余Worker）与全局降级（剔除失败任务，部分完成整体流程）相结合。
- **动态扩缩容**：监控任务队列堆积情况，动态调整Worker实例数量或并发上限，缩容时需保证任务优雅下线。
- **可观测性**：对任务执行状态、耗时、成功率建立严密监控，Worker失败超阈值时主动触发告警。
