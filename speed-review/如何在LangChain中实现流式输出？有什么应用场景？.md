# 如何在LangChain中实现流式输出？有什么应用场景？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- 流式输出是一种基于观察者模式（事件监听）的增量数据推送机制。
- LLM在生成每个token时触发回调事件，将传统的“等待完整响应”转变为“实时渐进式输出”。
- 核心价值：显著降低用户感知延迟，解决长文本生成的等待焦虑，并降低服务端峰值内存占用。

## 机制与原理
- **参数启用**：初始化LLM时设置 `streaming=True`，激活底层生成循环的钩子。
- **回调分发**：模型每生成一个新token，就会调用已注册回调函数的 `on_llm_new_token` 方法。
- **处理模式**：从传统的“客户端主动拉取完整结果”转变为“服务端主动推送增量数据”。
- **架构层级**：位于LLM抽象层与应用层之间，通过统一的 `callbacks` 参数向上层（Chain/Agent）提供可组合的流式能力。

## 对比速记
- **传统批量输出**：同步阻塞，黑盒等待，需在内存中累积完整响应后返回，首字节响应时间长。
- **流式输出**：异步增量，透明生成，边生成边推送到前端，首字节响应时间极短。

## 代码示例
```java
// 结合 WebSocket 实现实时流式输出推送到前端
public class WebSocketStreamingHandler extends BaseCallbackHandler {
    private final WebSocketSession session;

    public WebSocketStreamingHandler(WebSocketSession session) {
        this.session = session;
    }

    @Override
    public void onLlmNewToken(String token) {
        try {
            // 构造实时消息并通过 WebSocket 推送增量 token
            StreamMessage message = new StreamMessage(token, false, false);
            String jsonMessage = objectMapper.writeValueAsString(message);
            session.sendMessage(new TextMessage(jsonMessage));
        } catch (Exception e) {
            log.error("Failed to send streaming token", e);
        }
    }

    @Override
    public void onLlmEnd(LlmResult result) {
        // 发送结束标记
        sendControlMessage(new StreamMessage("", false, true));
    }
}
```

## 工程要点
- **网络协议**：需结合 WebSocket 或 SSE (Server-Sent Events) 等支持服务端主动推送的协议。
- **资源释放**：WebSocket 连接必须在 `onLlmEnd` 或 `onLlmError` 中确保正确释放，避免内存泄漏。
- **负载均衡**：WebSocket 的有状态特性要求网关层配置会话亲和性。
- **性能防抖**：回调函数内严禁执行重计算逻辑，避免阻塞 LLM 的 token 生成主流程。
- **背压处理**：当前端处理速度慢于 token 生成速度时，需设计缓冲机制防止内存溢出。
