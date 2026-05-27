# Multi-Agent系统中的角色分工如何设计？有哪些典型角色模式？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- Multi-Agent角色分工设计核心是按能力边界和任务属性拆分职责，确保每个Agent专注特定领域，通过协作完成复杂目标。
- 本质类似微服务架构，核心在于寻找合理的系统边界，避免能力重叠导致决策混乱。

## 机制与原理
- **单一职责原则**：每个Agent仅对一类任务负责，Prompt和工具集保持高度内聚。
- **能力导向映射**：梳理任务链路，提取核心能力需求（如意图理解、检索、决策），将其映射为对应角色。
- **协作互补机制**：角色间形成双向互动与交叉验证（如执行者与评审者互补），提升系统整体容错能力。
- **典型角色模式**：
  - **规划者**：将自然语言需求转化为结构化执行计划（如DAG图），负责意图理解、任务分解与依赖管理。
  - **执行者**：特定领域专家，配备专属工具集和知识库（如Python执行器对接解释器，SQL执行器对接数据库）。
  - **评审者**：提供独立视角，使用专门的验证逻辑和规则检查输出质量，避免执行者陷入思维定式。
  - **协调者**：管理多Agent间的通信和任务分配，决定是采用中心化调度还是点对点协商。
- **协作通信机制**：
  - **消息传递**：类似RPC调用，参数和结果清晰可追踪，需注意消息格式（JSON结构化 vs 自然语言）的理解成本。
  - **共享内存/黑板模式**：所有Agent读写同一状态白板，适合异步多轮迭代优化，工程难点在于并发控制（需加锁或版本控制）。

## 对比速记
| 协作模式 | 机制原理 | 优势 | 劣势/挑战 |
| :--- | :--- | :--- | :--- |
| **中心化调度** | 所有消息经过协调者节点分配 | 可控性强，流程清晰 | 协调者可能成为性能瓶颈 |
| **点对点协商** | Agent之间直接进行通信交互 | 灵活性极高 | 容易失控，协调成本高 |
| **共享内存/黑板** | 各Agent异步向公共黑板读写信息 | 适合多轮迭代，无需严格先后顺序 | 并发修改冲突，需设计触发条件 |

## 代码示例
通过抽象基类定义角色能力边界，不同角色重写执行逻辑以体现能力差异：

```java
// 基础角色抽象类，封装通用属性与工具匹配逻辑
public abstract class BaseAgent {
    private String roleName;
    private String capability;
    private List<Tool> tools;
    private LLMClient llmClient;

    public BaseAgent(String roleName, String capability, List<Tool> tools) {
        this.roleName = roleName;
        this.capability = capability;
        this.tools = tools;
        this.llmClient = new LLMClient();
    }

    public abstract AgentResponse execute(AgentRequest request);

    protected Tool findMatchingTool(Task task) {
        return tools.stream()
            .filter(tool -> tool.canHandle(task))
            .findFirst()
            .orElseThrow(() -> new RuntimeException("No matching tool found"));
    }
}

// 规划者角色：调用LLM进行推理与任务拆解
public class PlannerAgent extends BaseAgent {
    public PlannerAgent() {
        super("Planner", "任务分解与规划", new ArrayList<>());
    }

    @Override
    public AgentResponse execute(AgentRequest request) {
        String prompt = String.format(
            "你是一个任务规划专家。请将以下目标拆解成可执行的步骤，包含具体动作和依赖关系：\n目标: %s", 
            request.getUserInput()
        );
        String llmResponse = llmClient.chat(prompt);
        List<Task> tasks = parseToTaskList(llmResponse); // 解析文本为任务列表
        return new AgentResponse(tasks, "规划完成");
    }
}

// 执行者角色：调用外部工具执行具体动作
public class ExecutorAgent extends BaseAgent {
    public ExecutorAgent(List<Tool> tools) {
        super("Executor", "执行具体任务", tools);
    }

    @Override
    public AgentResponse execute(AgentRequest request) {
        Tool tool = findMatchingTool(request.getTask());
        Object result = tool.execute(request.getTask().getParams());
        return new AgentResponse(result, "执行成功");
    }
}
```

## 工程要点
- **角色粒度权衡**：过粗会导致Prompt过于复杂、模型逻辑混淆；过细则增加系统通信与协调开销。判断标准为：当两个职责需要完全不同的上下文或工具集时必须拆分，仅是流程先后关系可合并。
- **动态与静态角色选择**：流程稳定的场景使用静态角色保证可预测性；需求多变的场景（如电商大促临时增加监控）使用动态角色，但需额外处理生命周期管理与状态传递。
- **避免职责不清晰**：若两个Agent能处理同类请求，协调器将无法路由。需定义明确的输入输出Contract，严格区分能力边界（如查询Agent只返回原始数据，分析Agent只做计算统计）。
- **插件化扩展能力**：新增能力应通过开发独立Tool并注册给对应Agent实现，避免修改Agent核心逻辑。
- **处理角色冲突**：当多Agent给出不同方案时，可引入优先级机制、决策仲裁者，或基于各Agent输出的置信度分数进行自动裁决。
- **评估与迭代**：从任务完成质量、系统运行效率（等待/通信耗时）、维护扩展成本三个维度评估角色设计合理性。建议先用最小化角色配置跑通主流程，再根据瓶颈逐步调整。
