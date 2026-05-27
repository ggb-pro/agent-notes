# 什么是Agent的工具学习（Tool Learning）？如何让Agent学会使用新工具？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- **定义**：Tool Learning是一种声明式的智能交互机制，Agent通过理解工具的描述文档，自主决策何时及如何调用外部API来满足用户需求。
- **关键特征**：
  - 声明式驱动：仅需提供工具说明书（如JSON Schema），无需硬编码的if-else规则。
  - 智能理解：能自动进行意图识别、参数映射和结果解析。
  - 动态规划：具备多步工具链的编排与任务分解能力。

## 机制与原理
- **调用流程**：意图识别 -> 任务分解 -> 工具选择 -> 参数提取 -> 执行API -> 解析结果 -> (若未完成则选择下一个工具) -> 生成回复。
- **工具描述规范**：需包含清晰的功能说明、严格的参数Schema（类型、必填、enum约束）以及返回值定义。
- **学习路径**：
  - **Prompt Engineering**：零成本快速验证，将工具文档直接拼入Prompt；但工具过多易导致Prompt膨胀和选择混淆。
  - **Few-shot**：提供成功调用案例，利用上下文学习提升模式识别和调用准确率。
  - **微调训练**：针对复杂业务逻辑（如特殊折扣规则），构建监督数据集微调模型，内化领域知识；代价是标注和维护成本高。
- **多步规划**：复合需求需引入ReAct框架，让Agent输出“思考-行动-观察”的结构化流程，动态决策下一步工具。

## 对比速记
| 对比维度 | 传统插件系统 | Tool Learning |
| :--- | :--- | :--- |
| **驱动方式** | 命令式，硬编码规则 | 声明式，基于文档理解 |
| **触发机制** | 关键词匹配或规则引擎 | LLM意图识别与智能推理 |
| **扩展性** | 新需求需修改代码逻辑 | 仅需提供新工具的描述文档 |

## 代码示例
```python
from langchain.agents import initialize_agent, Tool, AgentType
from langchain.llms import OpenAI
import requests

def query_weather(city: str) -> str:
    """调用天气API获取指定城市的天气"""
    response = requests.get(f"https://api.weather.com/v1/current?city={city}")
    data = response.json()
    return f"{city}当前温度{data['temperature']}度，{data['weather']}"

# 定义工具列表（description是Agent理解用途的唯一依据）
tools = [
    Tool(
        name="WeatherQuery",
        func=query_weather,
        description="查询指定城市的实时天气信息。输入参数为城市名称字符串，如'北京'或'上海'"
    )
]

# 初始化Agent（采用ReAct框架）
llm = OpenAI(temperature=0)
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

# 执行调用
response = agent.run("北京今天天气怎么样？")
```

## 工程要点
- **架构抽象**：避免让Agent直接处理相似工具（如“微信支付”与“支付宝”），应抽象为高层统一工具（如“执行支付”），降低决策复杂度。
- **分层兜底与容错**：
  - 参数错误：Schema验证失败时不重试，应追问用户补充信息。
  - 超时重试：需判断操作幂等性，非幂等操作（如支付）重试前必须先查询状态，避免重复执行。
- **安全性控制**：建立工具白名单与分级授权，高风险操作需二次确认；严格校验生成参数，防止SQL/命令注入；设置调用频率上限防死循环。
- **可观测性**：记录完整的调用链（请求ID、工具名、参数、耗时、状态），接入分布式追踪（TraceID/SpanID），监控成功率与P99延迟。
- **幻觉防范**：强制要求Agent只能从注册列表中选择工具，且必须基于真实API返回结果回复，禁止捏造工具或虚构输出。
