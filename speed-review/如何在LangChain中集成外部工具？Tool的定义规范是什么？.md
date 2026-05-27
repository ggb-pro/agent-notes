# 如何在LangChain中集成外部工具？Tool的定义规范是什么？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- LangChain通过Tool类实现LLM与外部世界的标准化交互，弥补其实时数据获取和具体操作能力的不足。
- **三要素**：`name`（标识工具）、`description`（指导LLM何时及如何使用）、`func/_run`（实际执行逻辑）。

## 机制与原理
- **工具选择机制**：Agent将用户输入和所有可用工具的description一起发送给LLM，LLM基于语义理解自动匹配并选择最合适的工具。
- **定义方式**：`@tool`装饰器适合简单的函数式工具；继承`BaseTool`类适合需要异步操作或复杂状态管理的场景。
- **参数规范**：通过定义`args_schema`（基于Pydantic）规范输入参数格式，帮助LLM理解参数结构并减少调用错误。
- **执行控制**：可设置`return_direct`参数控制工具执行后是否跳过Agent的整合逻辑，直接将结果返回给用户。

## 对比速记
- **@tool装饰器**：代码极简，适合单一功能的轻量级工具。
- **BaseTool继承**：结构化强，支持复杂的连接管理、状态维护及参数校验（args_schema）。

## 代码示例
```python
from langchain.tools import BaseTool
from langchain.pydantic_v1 import BaseModel, Field
import requests

# 1. 定义参数校验结构
class CalculatorInput(BaseModel):
    expression: str = Field(description="数学表达式，如 '2+3*4'")

# 2. 继承BaseTool实现带安全控制和异常处理的工具
class ApiTool(BaseTool):
    name = "weather_api"
    description = "获取指定城市的实时天气信息"
    args_schema = CalculatorInput # 示例复用，实际应为对应的Input结构

    def _run(self, city: str) -> str:
        try:
            # 生产环境必须设置timeout
            response = requests.get(
                "http://api.weather.com/v1/current",
                params={"city": city},
                timeout=5
            )
            response.raise_for_status()
            return f"天气信息：{response.json()}"
        except requests.Timeout:
            return "API调用超时，请稍后重试"
        except requests.RequestException as e:
            return f"API调用失败：{str(e)}"
```

## 工程要点
- **Description设计**：需准确且具引导性（如明确适用场景、参数含义），避免过于模糊导致LLM误判。
- **容错与降级**：需实现多层次的重试策略（如指数退避）、熔断机制和降级方案（如返回本地缓存）。
- **安全性防护**：严格校验用户输入，数据库操作必须使用参数化查询防注入；API密钥严禁硬编码，需使用环境变量或密钥管理服务。
- **连接池管理**：对于数据库等I/O密集型工具，需使用连接池（如contextmanager）确保资源的高效释放。
- **可观测性**：所有工具调用需记录完整日志（输入参数、执行时间、返回结果），以便问题排查和性能分析。
