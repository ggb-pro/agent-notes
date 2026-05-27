# 如何调试LangGraph工作流？有哪些调试工具？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- LangGraph工作流调试的核心是追踪状态在多个AI节点间的流转过程。
- 调试机制需监控全局状态的变化轨迹，并处理由AI模型输出带来的非确定性路由决策。

## 机制与原理
- **状态追踪**：在节点的入口和出口处记录状态快照，监控数据在工作流中的变化轨迹，快速定位引发异常的修改节点。
- **日志分层**：应用层日志记录业务逻辑，框架层监控运行状态，系统层关注资源。开发阶段使用DEBUG级别，生产环境使用INFO级别仅记录关键结果。
- **可视化分析**：通过执行图谱展示状态变化、条件分支和循环等动态信息，快速发现如意外死循环等条件判断问题。
- **性能监控**：利用时间轴视图识别执行瓶颈，区分延迟是来自模型调用还是数据处理，并辅以异步处理或缓存优化。

## 对比速记
- **开发阶段**：侧重快速迭代与实时反馈，简单的日志输出往往比复杂可视化工具更高效。
- **测试阶段**：侧重全面执行信息收集，需使用专业工具（如LangSmith）进行深度分析。
- **生产环境**：侧重性能监控与异常报警，需采用轻量级结构化日志配合告警系统，并实施日志采样。

## 代码示例
```java
// 节点处理与性能监控结合示例
public State processWithMonitoring(State state) {
    long startTime = System.currentTimeMillis();
    try {
        // 业务逻辑处理
        State result = actualProcess(state);
        
        long duration = System.currentTimeMillis() - startTime;
        if (duration > 1000) {  // 超过1秒记录警告
            logger.warn("节点执行耗时过长: {}ms, 状态大小: {}", duration, state.getDataSize());
        }
        return result;
    } catch (Exception e) {
        logger.error("节点执行异常, 耗时: {}ms", System.currentTimeMillis() - startTime, e);
        throw e;
    }
}
```

## 工程要点
- **核心工具**：LangSmith是最核心的调试工具，配置 `LANGCHAIN_TRACING_V2=true` 环境变量即可获取完整的执行轨迹、输入输出及错误信息。
- **结构图生成**：使用 `graph.getGraph().drawMermaid()` 方法生成工作流结构图，理清节点连接关系。
- **路由调试**：在路由函数中添加决策日志，明确记录每次路由选择的具体依据（如购物车商品数、会员类型）。
- **生产规范**：生产环境禁用DEBUG级别日志，采用结构化日志记录关键节点，配合告警系统实现异常自动发现。
- **标准排查流程**：本地复现问题 -> 基础日志定位大致范围 -> LangSmith深入分析具体节点 -> 验证修复效果并补充监控。
