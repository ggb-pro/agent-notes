# Agent系统的监控指标有哪些？如何实时追踪Agent的执行状态？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- Agent系统监控基于传统可观测性黄金三角（指标、日志、追踪），重心从资源消耗转移到决策质量与执行效果。
- 监控体系分为三层架构：底层资源层（CPU/内存）、中间运行时层（LLM调用/工具执行）、顶层业务层（任务完成率/用户满意度）。

## 机制与原理
- **执行层面**：监控规划步骤数与实际执行偏差；工具调用成功率（生产环境需稳定在95%以上）；连续重试次数及异常中断率。
- **性能层面**：精细化统计Token消耗（区分输入、输出、内部中间Token，输出Token成本通常为输入的3-5倍）；监控上下文窗口利用率（超过80%易导致关键信息丢失）。
- **实时追踪机制**：为每次Agent执行分配全局唯一TraceID，在推理前后、工具调用前后等关键节点生成SpanID，通过父子关系串联完整链路。
- **数据采集模式**：关键决策节点采用同步监控（毫秒级延迟，用于人工介入判断）；统计性指标采用异步批量聚合上报（不影响主流程性能）。
- **分级采样机制**：正常任务按低比例（如1%）采样；异常任务（如超时、失败）自动提升采样率至100%，确保异常场景可完整回溯。

## 对比速记
| 监控维度 | 传统软件系统重点 | Agent系统重点 |
| :--- | :--- | :--- |
| **核心关注点** | 资源稳定性（CPU、内存、网络） | 决策质量与执行效果 |
| **成本指标** | 服务器计算/存储开销 | LLM Token消耗量（占总开销70%+） |
| **链路追踪** | 数据库查询、微服务RPC调用 | LLM推理过程、工具调用、状态转换 |
| **告警阈值** | 基于固定资源水位线 | 基于业务容忍度倒推（如P99响应时间） |

## 代码示例
```java
// 基于AOP切面的Agent LLM调用无侵入监控埋点
@Aspect
@Component
public class AgentMonitorAspect {
    @Autowired private Tracer tracer;
    private final RingBuffer<MetricEvent> buffer = new RingBuffer<>(10000);

    @Around("@annotation(monitorLLMCall)")
    public Object monitorLLM(ProceedingJoinPoint joinPoint, MonitorLLMCall monitorLLMCall) throws Throwable {
        Span span = tracer.buildSpan("llm-call")
            .withTag("trace.id", MDC.get("traceId")).start();
        long startTime = System.currentTimeMillis();
        try {
            Object result = joinPoint.proceed();
            if (result instanceof LLMResponse) {
                LLMResponse response = (LLMResponse) result;
                span.setTag("tokens.input", response.getInputTokens());
                span.setTag("tokens.output", response.getOutputTokens());
                // 异步写入环形缓冲区，不影响主流程
                buffer.offer(new MetricEvent("llm.tokens.consumed", 
                    response.getInputTokens() + response.getOutputTokens(), System.currentTimeMillis()));
            }
            return result;
        } catch (Exception e) {
            span.setTag("error", true);
            throw e;
        } finally {
            span.setTag("duration.ms", System.currentTimeMillis() - startTime);
            span.finish();
        }
    }

    @Scheduled(fixedRate = 5000)
    public void flushMetrics() {
        // 每5秒批量拉取聚合数据推送给Prometheus Pushgateway
        pushGateway.pushAdd(aggregateMetrics(buffer.drain()), "agent-monitor");
    }
}
```

## 工程要点
- **Dashboard设计**：划分健康度总览（在线数/成功率/消耗率）、趋势图表（叠加历史基线与告警线）、实时告警滚动三个区域，确保3秒内判断系统状态。
- **告警防抖策略**：使用Prometheus的`for`子句要求指标持续异常（如5分钟）才触发；同类告警设置10分钟静默期；多Agent并发场景采用聚合条件（如“3/5节点同时失败才告警”）。
- **监控容错机制**：上报采用fire-and-forget模式，若监控组件不可用，数据暂存本地有限容量缓冲区（如10000条），溢出则丢弃早期数据，确保Agent主流程稳定。
- **异常根因定位**：成功率突降时，通过Grafana确认范围 -> Jaeger筛选失败Trace看火焰图定位慢Span -> ELK检索具体错误堆栈。
- **成本突增排查**：输入Token突增通常因上下文未合理清理截断，输出Token突增多因Prompt变动导致推理冗长。
