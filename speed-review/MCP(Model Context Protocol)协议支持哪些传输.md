# MCP(Model Context Protocol)协议支持哪些传输方式？各有什么特点？

> **难度**: 困难 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- MCP（模型上下文协议）是一种应用层协议，通过抽象底层传输细节，支持多种通信方式。
- 协议采用分层设计，上层统一使用JSON-RPC消息格式，底层传输机制可根据实际部署场景灵活替换。

## 机制与原理
- **stdio传输**：基于操作系统标准输入输出流的进程间通信。客户端启动工具服务器进程并建立管道，零网络开销，延迟在微秒级，仅限同主机通信。
- **SSE传输**：基于HTTP的单向流式推送。服务器通过持久连接持续推送`text/event-stream`格式数据，客户端通过HTTP POST发送消息。原生兼容Web，防火墙穿透能力强。
- **WebSocket传输**：提供全双工通信能力。建立连接后客户端与服务器可随时双向主动发消息，网络传输开销低，适合高频双向交互。

## 对比速记
| 传输方式 | 通信机制 | 延迟 / 性能 | 适用场景 |
| :--- | :--- | :--- | :--- |
| **stdio** | 进程间管道通信 | 极低（微秒级） | 本地AI工具集成、单机部署 |
| **SSE** | HTTP单向流式推送 | 较高（HTTP开销） | 监控数据推送、Web兼容场景 |
| **WebSocket**| 全双工网络通信 | 较低（网络传输） | 实时协作、分布式系统高频交互 |

## 代码示例
```java
// stdio传输的Java实现示例（展示MCP本地管道通信机制）
public class StdioMCPClient {
    private Process serverProcess;
    private BufferedWriter writer;
    private BufferedReader reader;

    public void connect(String serverCommand) throws IOException {
        ProcessBuilder pb = new ProcessBuilder(serverCommand.split(" "));
        serverProcess = pb.start();

        writer = new BufferedWriter(new OutputStreamWriter(serverProcess.getOutputStream()));
        reader = new BufferedReader(new InputStreamReader(serverProcess.getInputStream()));
    }

    public void sendMessage(String jsonMessage) throws IOException {
        writer.write(jsonMessage);
        writer.newLine();
        writer.flush();
    }

    public String receiveMessage() throws IOException {
        return reader.readLine();
    }
}
```

## 工程要点
- **架构扩展**：单机达到性能瓶颈时，可平滑切换至分布式的WebSocket架构，业务逻辑代码无需修改。
- **传输安全**：stdio本地运行相对安全；SSE和WebSocket在网络传输时必须引入TLS加密和身份认证机制。
- **高可用与降级**：网络传输需设计重连机制与降级策略（如WebSocket断开后自动降级为HTTP轮询）。
- **跨平台兼容**：Windows与Linux的stdio管道处理机制存在差异，需在应用层做抽象封装以屏蔽系统差异。
