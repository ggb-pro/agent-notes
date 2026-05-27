# AgentBench评测框架包含哪些维度？如何设计Agent的benchmark？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- **AgentBench**：清华团队提出的评估LLM作为Agent能力的框架，通过多环境真实交互考察Agent表现，跳出传统静态QA评测局限。
- **Benchmark设计核心**：需兼顾环境的多样性、交互的真实性、任务的渐进性，并建立清晰的状态转移、终止条件与多维评分机制。

## 机制与原理
- **8大评测维度**：涵盖OS（命令生成执行）、DB（SQL查询优化）、KG（多跳推理）、横向思维谜题（非常规推理）、数字卡牌游戏（不完全信息决策）、ALFWorld（物理环境导航）、WebShop（电商交互）、Web浏览（开放式检索）。
- **环境设计哲学**：采用半真实环境（如固定模拟网站），在保证任务真实性的同时确保评测的可复现性。
- **数据构建策略**：分层构建。简单任务用规则生成保证覆盖度，复杂任务从真实场景采集并人工标注关键节点，基于最终状态验证而非逐步动作标注。
- **随机性处理**：固定随机种子做多次采样评估稳定性；明确区分确定性输出任务与多样性输出任务，采用不同评估标准（如严格匹配或语义相似度）。

## 对比速记
| 评估维度 | 传统静态QA评测 | AgentBench 动态交互评测 |
| :--- | :--- | :--- |
| **评测环境** | 孤立的文本问答对 | 真实/半真实的操作系统、数据库、网页等 |
| **交互模式** | 单轮输入输出 | 多轮交互，根据环境反馈动态调整策略 |
| **考察重点** | 知识储备与生成准确度 | 工具使用、推理决策、多步规划与异常处理能力 |

## 代码示例
```java
// Agent评测环境的沙箱化设计示例
public class AgentBenchmarkEnvironment {
    private Map<String, Order> mockOrderDatabase;
    private List<ActionLog> executionLogs;
    private int maxSteps = 50; // 防止无限循环

    public AgentBenchmarkEnvironment() {
        this.mockOrderDatabase = initMockDatabase(); // 初始化隔离的测试数据集
        this.executionLogs = new ArrayList<>();
    }

    // 模拟真实API行为（含边界情况）
    public QueryResult queryOrder(QueryRequest request) {
        executionLogs.add(new ActionLog("QUERY", request));
        if (Math.random() < 0.1) return QueryResult.timeout(); // 模拟网络延迟异常
        List<Order> results = mockOrderDatabase.values().stream()
            .filter(order -> matchQuery(order, request)).collect(Collectors.toList());
        return new QueryResult(results, executionLogs.size() < maxSteps);
    }

    // 多层次评分逻辑
    public BenchmarkScore evaluate(String taskId) {
        Task task = TaskRegistry.getTask(taskId);
        boolean success = task.verifyResult(mockOrderDatabase); // 基于最终状态验证
        double efficiency = Math.max(0, 1.0 - (executionLogs.size() - task.getOptimalSteps()) * 0.1);
        return new BenchmarkScore(success, efficiency, executionLogs);
    }
}
```

## 工程要点
- **四步设计框架**：明确评测目标 -> 确定任务范围 -> 搭建评测环境 -> 制定评分标准。
- **任务设计**：需具备区分度，既要有基准任务保证基本功能，也要有边缘挑战性任务拉开能力差距。任务描述需做多样性变换，防止Agent过拟合特定句式。
- **环境搭建**：采用沙箱化环境，Mock接口需模拟真实环境的各种情况（如超时、字段缺失、分页），并维护清晰的环境状态变化记录。
- **多维评分体系**：不能仅看成功率。基础层看任务完成度，第二层看过程质量（步骤效率、错误处理），第三层看泛化能力（任务变体表现）。
- **防数据泄露**：评测集需有版本管理，定期更新并加入新困难样本，保证评测数据未在Agent训练集中出现过。
- **安全与成本**：设计对抗性测试集评估Agent安全意识（防止盲目执行危险命令）；日常回归采用自动化测试，复杂任务辅以人工评估，逐步积累实现自动化评分。
