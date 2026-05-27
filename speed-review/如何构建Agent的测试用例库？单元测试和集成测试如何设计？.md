# 如何构建Agent的测试用例库？单元测试和集成测试如何设计？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- Agent测试用例库是针对概率性系统（LLM具有随机性）设计的质量保障体系，需覆盖意图识别、上下文管理、工具调用和响应生成。
- **核心特征**：应对非确定性输出、验证多步推理链路、隔离复杂外部依赖、处理多轮对话的状态爆炸。

## 机制与原理
- **三层测试金字塔**：
  - **单元测试**：隔离环境下验证单个组件（如Prompt生成器、工具函数、记忆模块），用Mock替代真实LLM和外部API，保证稳定性和低成本。
  - **集成测试**：验证组件协作与状态传递，如多轮对话的上下文保持、工具编排顺序。
  - **端到端测试(E2E)**：在沙盒环境接入真实API，模拟完整业务链路和异常恢复。
- **用例库分层组织**：
  - **原子能力层**：测试意图识别、参数提取等基础能力，输入输出明确，可用传统断言验证。
  - **组合能力层**：验证多工具编排的调用顺序和参数传递，记录工具调用序列作为Golden Path。
  - **场景化层**：模拟真实用户完整任务流程，覆盖正常路径、边界情况（如上下文溢出）和异常路径（如工具超时）。
- **幻觉处理机制**：识别易幻觉场景（如精确计算），测试Agent是否优先调用工具；对生成内容建立事实性交叉验证。
- **LLM评估(LaaS)**：对主观性强的质量维度（如语气、专业性），利用另一个LLM作为评判者进行打分。

## 对比速记
| 测试类型 | 测试目标 | 依赖策略 | 运行频率与成本 |
| :--- | :--- | :--- | :--- |
| **单元测试** | 单组件逻辑（Prompt、工具、记忆） | 本地Mock/小模型/模板匹配 | 每次代码提交运行，成本极低 |
| **集成测试** | 组件协作、上下文传递、工作流 | Mock核心LLM，部分真实组件 | 合并前运行，成本中等 |
| **端到端测试** | 完整业务场景、真实API交互 | 真实沙盒环境、缓存LLM响应 | 发版前定期运行，成本较高 |

## 代码示例
**带缓存的LLM Mock机制**（兼顾测试稳定性与API成本控制）：
```python
import hashlib, json, os

class CachedLLM:
    def __init__(self, real_llm, cache_dir):
        self.real_llm = real_llm
        self.cache_dir = cache_dir

    def generate(self, prompt):
        # 相同Prompt直接返回缓存，保证测试可复现且零API成本
        cache_key = hashlib.md5(prompt.encode()).hexdigest()
        cache_file = f"{self.cache_dir}/{cache_key}.json"

        if os.path.exists(cache_file):
            with open(cache_file) as f:
                return json.load(f)["response"]

        # 仅未命中缓存时调用真实模型
        response = self.real_llm.generate(prompt)
        with open(cache_file, 'w') as f:
            json.dump({"prompt": prompt, "response": response}, f)

        return response
```

## 工程要点
- **可测试性架构设计**：将Prompt模板独立为配置文件，工具调用接口定义清晰的输入输出Schema，实现Mock与真实调用的无缝替换。
- **数据集分层管理**：核心种子用例(Seed Cases)保底 + 规则生成的变体数据提升覆盖度 + 线上采样的真实脱敏数据发现长尾问题。
- **CI/CD 多级流水线**：单元测试快速拦截基础问题；集成测试验证组件契约；E2E测试仅在PR或发版前触发。
- **回归与监控闭环**：维护Benchmark Suite，监控任务完成率、Token消耗、响应延迟等指标；将线上失败case自动脱敏后补充至测试库。
- **成本控制**：日常回归采用采样策略（如抽取20%用例），大幅缩短日常测试时间。
