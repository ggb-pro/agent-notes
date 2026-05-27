# AI Agent 社招面试知识地图

> 基于 86 篇 Agent 技术笔记的体系化整理，覆盖 Agent 理论、框架、工程化全链路。
>
> 所有笔记位于 `speed-review/` 目录，每篇 3-4KB，1-2 分钟速记复习。点击下方链接直达。

---

## 一、知识体系全景图

```
Agent 面试知识体系
├── 1. 基础概念 ─────────────────────── 必考，回答要精准
│   ├── AI Agent 定义与本质区别
│   ├── LLM-based Agent 特点
│   ├── 工具使用 Agent (Tool-using Agent)
│   └── 代码解释器 (Code Interpreter)
│
├── 2. 核心推理模式 ─────────────────── 高频对比题（ReAct vs CoT vs ToT）
│   ├── ReAct（推理+行动）
│   ├── Plan-and-Execute（分层规划）
│   ├── ReWOO（无观察推理）
│   ├── Reflexion（自我反思）
│   ├── Graph of Thoughts（图推理）
│   ├── Self-Refinement（自优化循环）
│   ├── Reflection 机制（反思触发条件）
│   └── Exploration vs Exploitation（探索利用平衡）
│
├── 3. 记忆系统 ─────────────────────── 架构设计题
│   ├── 短期记忆 / 长期记忆设计
│   ├── 长期记忆持久化 & 向量数据库选型
│   ├── LangChain Memory 组件
│   └── 对话历史窗口策略
│
├── 4. Multi-Agent 系统 ─────────────── 系统设计高频考点
│   ├── 基础概念与架构
│   ├── Agent 间通信协议
│   ├── 角色分工设计
│   ├── 层级式架构 (Manager-Worker)
│   ├── 任务分配 & 负载均衡
│   ├── 投票 & 共识机制
│   ├── 一致性问题
│   ├── 冲突检测与解决
│   ├── 辩论 (Debate) 模式
│   ├── 涌现行为 (Emergent Behavior)
│   └── 社会模拟 (Social Simulation)
│
├── 5. 框架：LangChain ─────────────── 工程实践题
│   ├── LangChain 核心（是什么 / 组件 / LCEL）
│   ├── Agent 执行器
│   ├── Memory / 缓存 / 输出解析器
│   ├── 文档处理
│   ├── RAG 实现
│   ├── 流式输出
│   ├── 工具集成规范
│   ├── 多轮对话构建
│   └── 问答机器人实战
│
├── 6. 框架：LangGraph ─────────────── 状态编排题
│   ├── LangGraph 基础（与 LangChain 关系）
│   ├── 核心概念（图 / 节点 / 边）
│   ├── 条件边 (Conditional Edge)
│   ├── 状态定义 & 管理
│   ├── 编译 & 执行过程
│   ├── 循环 & 迭代
│   ├── 并行执行
│   ├── 持久化机制
│   ├── 人机交互 (Human-in-the-loop)
│   ├── 超时控制
│   ├── 错误 & 异常处理
│   ├── 调试工具
│   └── 数据分析流程实战
│
├── 7. MCP 协议 ─────────────────────── 新兴标准，加分项
│   ├── MCP 是什么 & 解决什么问题
│   ├── 基本架构 (Client-Server / JSON-RPC 2.0)
│   ├── Resource 管理
│   ├── 传输方式
│   ├── 安全机制
│   └── MCP 开发实战
│
├── 8. 评测体系 ─────────────────────── 展现专业深度
│   ├── Agent 性能评估指标
│   ├── 响应质量评估 & 改进
│   ├── AgentBench 评测框架
│   ├── WebArena 网页操作评测
│   ├── AutoGPT 原理 & BabyAGI 对比
│   └── 测试用例库构建（单元 / 集成）
│
├── 9. 工程化与生产 ─────────────────── 区分度最高的部分
│   ├── 错误处理（执行过程中的错误类型 & 应对）
│   ├── 调试常见问题 & 方法
│   ├── 容错设计（对话模块）
│   ├── 降级策略（模型不可用时）
│   ├── 成本计算 & 效果-成本平衡
│   ├── 监控指标 & 实时追踪
│   ├── 提示词迭代优化 & A/B 测试
│   ├── 网络搜索功能集成
│   ├── 文件处理 Agent 实战
│   └── Agentic Workflow（与传统工作流对比）
│
├── 10. 安全与可信 ─────────────────── 合规方向必考
│   ├── 可解释性实现
│   ├── 安全性威胁 & 防范
│   ├── 可解释性与可控性权衡
│   └── 幻觉问题 & 验证策略
│
└── 11. 常见问题与失败模式 ─────────── 排查能力考察
    ├── 失败案例分析 & 失败模式
    ├── 规划能力（动态重规划 / 元认知 / 部分可观察）
    └── NLG 生成策略（模板 / 检索 / 生成）
```

---

## 二、面试复习路线（推荐顺序）

### Phase 1：基础打底（1-2 天）

| 序号 | 主题 | 笔记 | 面试问法 |
|------|------|------|----------|
| 1 | AI Agent 定义 | [什么是AI Agent？与传统软件程序有什么本质区别？](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AFAI%20Agent%EF%BC%9F%E4%B8%8E%E4%BC%A0%E7%BB%9F%E8%BD%AF%E4%BB%B6%E7%A8%8B%E5%BA%8F%E6%9C%89%E4%BB%80%E4%B9%88%E6%9C%AC%E8%B4%A8%E5%8C%BA%E5%88%AB%EF%BC%9F.md>) | "什么是AI Agent？和传统程序有什么区别？" |
| 2 | LLM Agent 特点 | [基于大语言模型的AI Agent有什么特点？](<speed-review/%E5%9F%BA%E4%BA%8E%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%EF%BC%88LLM%EF%BC%89%E7%9A%84AI%20Agent%E6%9C%89%E4%BB%80%E4%B9%88%E7%89%B9%E7%82%B9%EF%BC%9F%E7%9B%B8%E6%AF%94%E4%BC%A0%E7%BB%9FAI%20A.md>) | "为什么现在大家都用LLM做Agent？" |
| 3 | 工具使用 | [什么是工具使用AI Agent？](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AF%E5%B7%A5%E5%85%B7%E4%BD%BF%E7%94%A8AI%20Agent%EF%BC%88Tool-using%20Agent%EF%BC%89%EF%BC%9F%E5%B7%A5.md>) | "Agent怎么使用外部工具？" |
| 4 | 代码解释器 | [什么是代码解释器？](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AF%E4%BB%A3%E7%A0%81%E8%A7%A3%E9%87%8A%E5%99%A8%EF%BC%88Code%20Interpreter%EF%BC%89%EF%BC%9F%E5%AE%83%E5%9C%A8AI%20Age.md>) | "代码解释器在Agent中起什么作用？" |

**核心记忆点：**
- Agent = 感知 + 决策 + 执行 的闭环，不是简单的 API 调用
- 与传统程序区别：确定性执行 → 概率性推理，硬编码分支 → 模型实时生成
- LLM Agent 特点：自然语言理解 + 推理 + 工具调用，few-shot 能力
- 工具使用三要素：工具描述、参数 schema、结果解析

---

### Phase 2：核心推理模式（2-3 天，面试最高频）

| 序号 | 主题 | 笔记 | 对比关系 |
|------|------|------|----------|
| 5 | ReAct | [什么是ReAct模式？](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AFReAct%E6%A8%A1%E5%BC%8F%EF%BC%9F%E5%AE%83%E5%A6%82%E4%BD%95%E6%8F%90%E5%8D%87AI%20Agent%E7%9A%84%E6%8E%A8%E7%90%86%E8%83%BD%E5%8A%9B%EF%BC%9F.md>) | vs CoT: 闭卷→开卷 |
| 6 | Plan-and-Execute | [什么是Plan-and-Execute模式？](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AFPlan-and-Execute%E6%A8%A1%E5%BC%8F%EF%BC%9F%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E4%BB%BB%E5%8A%A1%E7%9A%84%E5%88%86%E5%B1%82%E8%A7%84%E5%88%92%EF%BC%9F.md>) | vs ReAct: 分层 vs 交替 |
| 7 | ReWOO | [ReWOO](<speed-review/ReWOO%EF%BC%88Reasoning%20WithOut%20Observation.md>) | vs ReAct: 规划执行分离 |
| 8 | Reflexion | [Reflexion机制](<speed-review/Reflexion%E6%9C%BA%E5%88%B6%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%EF%BC%9F%E5%A6%82%E4%BD%95%E9%80%9A%E8%BF%87%E8%87%AA%E6%88%91%E5%8F%8D%E6%80%9D%E6%8F%90%E5%8D%87Agent%E6%80%A7%E8%83%BD%EF%BC%9F.md>) | 基于失败轨迹反思 |
| 9 | GoT | [Graph of Thoughts](<speed-review/Graph%20of%20Thoughts%EF%BC%88GoT%EF%BC%89%E5%A6%82%E4%BD%95%E8%A1%A8%E7%A4%BA%E5%A4%8D%E6%9D%82%E6%8E%A8%E7%90%86%E8%BF%87%E7%A8%8B%EF%BC%9F%E6%9C%89%E4%BB%80.md>) | vs CoT/ToT: DAG 结构 |
| 10 | Self-Refinement | [Self-Refinement](<speed-review/Agent%E7%9A%84Self-Refinement%E6%98%AF%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E7%9A%84%EF%BC%9F%E9%9C%80%E8%A6%81%E5%93%AA%E4%BA%9B%E5%85%B3%E9%94%AE%E7%BB%84.md>) | Generator-Critic-Refiner |
| 11 | Reflection | [反思机制设计](<speed-review/%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1Agent%E7%9A%84%E5%8F%8D%E6%80%9D%EF%BC%88Reflection%EF%BC%89%E6%9C%BA%E5%88%B6%EF%BC%9F%E4%BD%95%E6%97%B6%E8%A7%A6%E5%8F%91%E5%8F%8D%E6%80%9D%EF%BC%9F.md>) | 反思触发条件设计 |
| 12 | Explore vs Exploit | [探索与利用](<speed-review/Agent%E7%9A%84%E6%8E%A2%E7%B4%A2%E4%B8%8E%E5%88%A9%E7%94%A8%EF%BC%88Exploration%20vs%20Exploita.md>) | ε-greedy / UCB / Thompson |

**面试高频对比题速记：**

```
ReAct vs Chain-of-Thought:
  CoT = 纯思维推理（闭卷考试）
  ReAct = 思考+行动+观察循环（开卷考试，可主动获取信息）

ReAct vs Plan-and-Execute:
  ReAct = 逐步交替推理和行动，灵活但可能冗余
  Plan-Execute = 先完整规划再逐步执行，适合结构化任务

ReAct vs ReWOO:
  ReAct = 思考→行动→观察交替串行
  ReWOO = 先完整规划所有步骤，再批量执行，支持并行，减少LLM调用

CoT vs ToT vs GoT:
  CoT = 线性推理链
  ToT = 树形搜索，多条路径探索
  GoT = DAG 图，节点可合并/复用，最灵活

Reflexion vs Self-Refinement:
  Reflexion = 基于失败轨迹的语言化反思，跨轮次记忆
  Self-Refinement = Generator-Critic-Refiner 同轮次内迭代优化
```

---

### Phase 3：记忆与规划（1-2 天）

| 序号 | 主题 | 笔记 |
|------|------|------|
| 13 | 记忆机制 | [AI Agent的记忆机制如何设计？](<speed-review/AI%20Agent%E7%9A%84%E8%AE%B0%E5%BF%86%E6%9C%BA%E5%88%B6%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1%EF%BC%9F%E7%9F%AD%E6%9C%9F%E8%AE%B0%E5%BF%86%E5%92%8C%E9%95%BF%E6%9C%9F%E8%AE%B0%E5%BF%86%E7%9A%84%E5%8C%BA%E5%88%AB%EF%BC%9F.md>) |
| 14 | 长期记忆持久化 | [长期记忆如何持久化？](<speed-review/Agent%E7%9A%84%E9%95%BF%E6%9C%9F%E8%AE%B0%E5%BF%86%E5%A6%82%E4%BD%95%E6%8C%81%E4%B9%85%E5%8C%96%EF%BC%9F%E5%90%91%E9%87%8F%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9A%84%E9%80%89%E6%8B%A9%E6%A0%87%E5%87%86%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F.md>) |
| 15 | 对话历史窗口 | [什么是对话历史窗口？](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AF%E5%AF%B9%E8%AF%9D%E5%8E%86%E5%8F%B2%E7%AA%97%E5%8F%A3%EF%BC%9F%E4%BF%9D%E7%95%99%E5%A4%9A%E5%B0%91%E8%BD%AE%E5%AF%B9%E8%AF%9D%E5%90%88%E9%80%82%EF%BC%9F.md>) |
| 16 | 规划能力 | [Agent的规划能力](<speed-review/AI%20Agent%E7%9A%84%E8%A7%84%E5%88%92%E8%83%BD%E5%8A%9B%E6%98%AF%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E7%9A%84%EF%BC%9F%E6%9C%89%E5%93%AA%E4%BA%9B%E8%A7%84%E5%88%92%E7%AD%96%E7%95%A5%EF%BC%9F.md>) |
| 17 | 动态重规划 | [动态重规划](<speed-review/Agent%E5%9C%A8%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B%E4%B8%AD%E5%A6%82%E4%BD%95%E8%BF%9B%E8%A1%8C%E5%8A%A8%E6%80%81%E9%87%8D%E8%A7%84%E5%88%92%EF%BC%9F%E8%A7%A6%E5%8F%91%E6%9D%A1%E4%BB%B6%E6%9C%89%E5%93%AA%E4%BA%9B%EF%BC%9F.md>) |
| 18 | 元认知能力 | [元认知能力](<speed-review/Agent%E7%9A%84%E5%85%83%E8%AE%A4%E7%9F%A5%E8%83%BD%E5%8A%9B%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F%E5%A6%82%E4%BD%95%E8%AF%84%E4%BC%B0%E8%87%AA%E8%BA%AB%E7%9A%84%E8%83%BD%E5%8A%9B%E8%BE%B9%E7%95%8C%EF%BC%9F.md>) |
| 19 | 部分可观察 | [部分可观察问题](<speed-review/%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86Agent%E7%9A%84%E9%83%A8%E5%88%86%E5%8F%AF%E8%A7%82%E5%AF%9F%E9%97%AE%E9%A2%98%EF%BC%9F%E4%BF%A1%E6%81%AF%E4%B8%8D%E5%AE%8C%E6%95%B4%E6%97%B6%E5%A6%82%E4%BD%95%E5%86%B3%E7%AD%96%EF%BC%9F.md>) |

**核心记忆点：**
- 短期记忆 = 上下文窗口内的对话/状态；长期记忆 = 向量数据库持久化
- 向量数据库选型：Milvus(大规模分布式) / Pinecone(托管) / Weaviate(混合检索) / Chroma(轻量)
- 对话窗口策略：滑动窗口 / 摘要压缩 / 优先级保留
- 动态重规划触发条件：执行失败、环境变化、新信息、资源约束、超时

---

### Phase 4：Multi-Agent 系统（2-3 天，系统设计重点）

| 序号 | 主题 | 笔记 |
|------|------|------|
| 20 | MAS 基础 | [什么是Multi-Agent系统？](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AF%E5%A4%9AAI%20Agent%E7%B3%BB%E7%BB%9F%EF%BC%88Multi-Agent%20System%EF%BC%89%EF%BC%9F.md>) |
| 21 | Agent 通信 | [Agent间通信](<speed-review/Agent%E4%B9%8B%E9%97%B4%E5%A6%82%E4%BD%95%E8%BF%9B%E8%A1%8C%E6%9C%89%E6%95%88%E9%80%9A%E4%BF%A1%EF%BC%9F%E6%B6%88%E6%81%AF%E6%A0%BC%E5%BC%8F%E5%92%8C%E5%8D%8F%E8%AE%AE%E5%A6%82%E4%BD%95%E5%AE%9A%E4%B9%89%EF%BC%9F.md>) |
| 22 | 角色分工 | [角色分工设计](<speed-review/Multi-Agent%E7%B3%BB%E7%BB%9F%E4%B8%AD%E7%9A%84%E8%A7%92%E8%89%B2%E5%88%86%E5%B7%A5%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1%EF%BC%9F%E6%9C%89%E5%93%AA%E4%BA%9B%E5%85%B8%E5%9E%8B%E8%A7%92%E8%89%B2%E6%A8%A1%E5%BC%8F%EF%BC%9F.md>) |
| 23 | 层级式架构 | [层级式架构](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AF%E5%B1%82%E7%BA%A7%E5%BC%8FMulti-Agent%E6%9E%B6%E6%9E%84%EF%BC%9FManager-Worker%E6%A8%A1.md>) |
| 24 | 任务分配 | [任务分配算法](<speed-review/%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1Multi-Agent%E7%9A%84%E4%BB%BB%E5%8A%A1%E5%88%86%E9%85%8D%E7%AE%97%E6%B3%95%EF%BC%9F%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%EF%BC%9F.md>) |
| 25 | 投票机制 | [投票机制](<speed-review/Multi-Agent%E7%B3%BB%E7%BB%9F%E4%B8%AD%E7%9A%84%E6%8A%95%E7%A5%A8%E6%9C%BA%E5%88%B6%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1%EF%BC%9F%E5%A6%82%E4%BD%95%E8%81%9A%E5%90%88%E4%B8%8D%E5%90%8CAgent.md>) |
| 26 | 一致性 | [一致性问题](<speed-review/Multi-Agent%E7%B3%BB%E7%BB%9F%E7%9A%84%E4%B8%80%E8%87%B4%E6%80%A7%E9%97%AE%E9%A2%98%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%EF%BC%9F%E5%85%B1%E8%AF%86%E7%AE%97%E6%B3%95%E5%A6%82%E4%BD%95%E5%BA%94%E7%94%A8%EF%BC%9F.md>) |
| 27 | 冲突解决 | [冲突检测与解决](<speed-review/Agent%E5%8D%8F%E4%BD%9C%E4%B8%AD%E7%9A%84%E5%86%B2%E7%AA%81%E5%A6%82%E4%BD%95%E6%A3%80%E6%B5%8B%E5%92%8C%E8%A7%A3%E5%86%B3%EF%BC%9F%E6%9C%89%E5%93%AA%E4%BA%9B%E5%86%B2%E7%AA%81%E8%A7%A3%E5%86%B3%E7%AD%96%E7%95%A5%EF%BC%9F.md>) |
| 28 | 辩论模式 | [辩论模式](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AFAgent%E7%9A%84%E8%BE%A9%E8%AE%BA%EF%BC%88Debate%EF%BC%89%E6%A8%A1%E5%BC%8F%EF%BC%9F%E5%A6%82%E4%BD%95%E9%80%9A%E8%BF%87%E5%A4%9AAgent%E8%BE%A9%E8%AE%BA%E6%8F%90.md>) |
| 29 | 涌现行为 | [涌现行为](<speed-review/Multi-Agent%E7%B3%BB%E7%BB%9F%E7%9A%84%E6%B6%8C%E7%8E%B0%E8%A1%8C%E4%B8%BA%EF%BC%88Emergent%20Behavio.md>) |
| 30 | 社会模拟 | [社会模拟](<speed-review/Agent%E7%A4%BE%E4%BC%9A%E6%A8%A1%E6%8B%9F%EF%BC%88Social%20Simulation%EF%BC%89%E6%9C%89%E4%BB%80%E4%B9%88%E5%BA%94%E7%94%A8%EF%BC%9F%E5%A6%82.md>) |

**架构模式速记：**

```
层级式 (Manager-Worker):
  Manager 负责任务拆解和分发，Worker 负责执行
  适合：明确分工的场景（代码审查、文档处理）

扁平式 (Peer-to-Peer):
  Agent 平等协作，通过消息总线通信
  适合：创意生成、多角度分析

辩论式 (Debate):
  多 Agent 对同一问题给出不同方案，通过辩论收敛到最优解
  适合：高风险决策、代码审查

混合式:
  上层层级管理 + 下层扁平协作
  适合：复杂业务系统
```

---

### Phase 5：框架实战（2 天）

#### LangChain 核心

| 序号 | 主题 | 笔记 |
|------|------|------|
| 31 | LangChain 是什么 | [LangChain是什么？](<speed-review/LangChain%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F%E5%AE%83%E8%A7%A3%E5%86%B3%E4%BA%86LLM%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E7%9A%84%E5%93%AA%E4%BA%9B%E9%97%AE%E9%A2%98%EF%BC%9F.md>) |
| 32 | 核心组件 | [核心组件](<speed-review/LangChain%E7%9A%84%E6%A0%B8%E5%BF%83%E7%BB%84%E4%BB%B6%E6%9C%89%E5%93%AA%E4%BA%9B%EF%BC%9F%E5%90%84%E8%87%AA%E7%9A%84%E4%BD%9C%E7%94%A8%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F.md>) |
| 33 | Agent 执行器 | [Agent执行器](<speed-review/LangChain%E7%9A%84Agent%E6%89%A7%E8%A1%8C%E5%99%A8%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84%EF%BC%9F.md>) |
| 34 | LCEL | [LCEL](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AFLCEL%EF%BC%88LangChain%20Expression%20Langua.md>) |
| 35 | Memory 组件 | [Memory组件](<speed-review/LangChain%E7%9A%84Memory%E7%BB%84%E4%BB%B6%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%EF%BC%9F%E6%9C%89%E5%93%AA%E4%BA%9BMemory%E7%B1%BB%E5%9E%8B%EF%BC%9F.md>) |
| 36 | 缓存机制 | [缓存机制](<speed-review/LangChain%E7%9A%84%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%EF%BC%9F%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96%E6%80%A7%E8%83%BD%EF%BC%9F.md>) |
| 37 | 输出解析器 | [输出解析器](<speed-review/LangChain%E7%9A%84%E8%BE%93%E5%87%BA%E8%A7%A3%E6%9E%90%E5%99%A8%E6%9C%89%E4%BB%80%E4%B9%88%E4%BD%9C%E7%94%A8%EF%BC%9F%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%E7%BB%93%E6%9E%84%E5%8C%96%E8%BE%93%E5%87%BA%EF%BC%9F.md>) |
| 38 | 文档处理 | [文档处理](<speed-review/LangChain%E7%9A%84%E6%96%87%E6%A1%A3%E5%A4%84%E7%90%86%E5%8A%9F%E8%83%BD%E5%8C%85%E6%8B%AC%E5%93%AA%E4%BA%9B%EF%BC%9F%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%E5%A4%A7%E6%96%87%E6%A1%A3%EF%BC%9F.md>) |

#### LangChain 实战

| 序号 | 主题 | 笔记 |
|------|------|------|
| 39 | RAG 实现 | [RAG实现](<speed-review/%E5%A6%82%E4%BD%95%E5%9C%A8LangChain%E4%B8%AD%E5%AE%9E%E7%8E%B0RAG%EF%BC%88%E6%A3%80%E7%B4%A2%E5%A2%9E%E5%BC%BA%E7%94%9F%E6%88%90%EF%BC%89%EF%BC%9F.md>) |
| 40 | 流式输出 | [流式输出](<speed-review/%E5%A6%82%E4%BD%95%E5%9C%A8LangChain%E4%B8%AD%E5%AE%9E%E7%8E%B0%E6%B5%81%E5%BC%8F%E8%BE%93%E5%87%BA%EF%BC%9F%E6%9C%89%E4%BB%80%E4%B9%88%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF%EF%BC%9F.md>) |
| 41 | 工具集成 | [工具集成](<speed-review/%E5%A6%82%E4%BD%95%E5%9C%A8LangChain%E4%B8%AD%E9%9B%86%E6%88%90%E5%A4%96%E9%83%A8%E5%B7%A5%E5%85%B7%EF%BC%9FTool%E7%9A%84%E5%AE%9A%E4%B9%89%E8%A7%84%E8%8C%83%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F.md>) |
| 42 | 多轮对话 | [多轮对话](<speed-review/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8LangChain%E6%9E%84%E5%BB%BA%E5%A4%9A%E8%BD%AE%E5%AF%B9%E8%AF%9D%E7%B3%BB%E7%BB%9F%EF%BC%9F.md>) |
| 43 | 问答机器人 | [问答机器人](<speed-review/%E5%A6%82%E4%BD%95%E7%94%A8LangChain%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84%E9%97%AE%E7%AD%94%E6%9C%BA%E5%99%A8%E4%BA%BAAgent%EF%BC%9F.md>) |

#### LangGraph

| 序号 | 主题 | 笔记 |
|------|------|------|
| 44 | LangGraph 基础 | [LangGraph是什么？](<speed-review/LangGraph%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F%E5%AE%83%E4%B8%8ELangChain%E6%9C%89%E4%BB%80%E4%B9%88%E5%85%B3%E7%B3%BB%EF%BC%9F.md>) |
| 45 | 核心概念 | [核心概念](<speed-review/LangGraph%E7%9A%84%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5%E6%9C%89%E5%93%AA%E4%BA%9B%EF%BC%9F%E5%9B%BE%E3%80%81%E8%8A%82%E7%82%B9%E3%80%81%E8%BE%B9%E7%9A%84%E4%BD%9C%E7%94%A8%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F.md>) |
| 46 | 条件边 | [条件边](<speed-review/LangGraph%E7%9A%84%E6%9D%A1%E4%BB%B6%E8%BE%B9%EF%BC%88Conditional%20Edge%EF%BC%89%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8.md>) |
| 47 | 状态管理 | [状态管理](<speed-review/%E5%A6%82%E4%BD%95%E5%9C%A8LangGraph%E4%B8%AD%E5%AE%9A%E4%B9%89%E7%8A%B6%E6%80%81%EF%BC%9F%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86%E7%9A%84%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%9F.md>) |
| 48 | 编译执行 | [编译执行](<speed-review/LangGraph%E7%9A%84%E7%BC%96%E8%AF%91%E5%92%8C%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B%E6%98%AF%E6%80%8E%E6%A0%B7%E7%9A%84%EF%BC%9F.md>) |
| 49 | 循环迭代 | [循环和迭代](<speed-review/%E5%A6%82%E4%BD%95%E5%9C%A8LangGraph%E4%B8%AD%E5%AE%9E%E7%8E%B0%E5%BE%AA%E7%8E%AF%E5%92%8C%E8%BF%AD%E4%BB%A3%EF%BC%9F.md>) |
| 50 | 并行执行 | [并行执行](<speed-review/%E5%A6%82%E4%BD%95%E5%9C%A8LangGraph%E4%B8%AD%E5%A4%84%E7%90%86%E5%B9%B6%E8%A1%8C%E6%89%A7%E8%A1%8C%EF%BC%9F.md>) |
| 51 | 持久化 | [持久化机制](<speed-review/LangGraph%E7%9A%84%E6%8C%81%E4%B9%85%E5%8C%96%E6%9C%BA%E5%88%B6%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F%E5%A6%82%E4%BD%95%E4%BF%9D%E5%AD%98%E6%89%A7%E8%A1%8C%E7%8A%B6%E6%80%81%EF%BC%9F.md>) |
| 52 | 人机交互 | [人机交互](<speed-review/LangGraph%E7%9A%84%E4%BA%BA%E6%9C%BA%E4%BA%A4%E4%BA%92%E5%8A%9F%E8%83%BD%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%EF%BC%9F.md>) |
| 53 | 超时控制 | [超时控制](<speed-review/%E5%A6%82%E4%BD%95%E5%9C%A8LangGraph%E4%B8%AD%E5%AE%9E%E7%8E%B0%E8%B6%85%E6%97%B6%E6%8E%A7%E5%88%B6%EF%BC%9F.md>) |
| 54 | 错误处理 | [错误和异常](<speed-review/LangGraph%E4%B8%AD%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%E9%94%99%E8%AF%AF%E5%92%8C%E5%BC%82%E5%B8%B8%EF%BC%9F.md>) |
| 55 | 调试 | [调试工具](<speed-review/%E5%A6%82%E4%BD%95%E8%B0%83%E8%AF%95LangGraph%E5%B7%A5%E4%BD%9C%E6%B5%81%EF%BC%9F%E6%9C%89%E5%93%AA%E4%BA%9B%E8%B0%83%E8%AF%95%E5%B7%A5%E5%85%B7%EF%BC%9F.md>) |
| 56 | 数据分析实战 | [数据分析流程](<speed-review/%E5%A6%82%E4%BD%95%E7%94%A8LangGraph%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AA%E5%A4%9A%E6%AD%A5%E9%AA%A4%E7%9A%84%E6%95%B0%E6%8D%AE%E5%88%86%E6%9E%90%E6%B5%81%E7%A8%8B%EF%BC%9F.md>) |

**LangChain vs LangGraph 选型指南：**
- 简单链式调用（文档摘要、翻译）→ LangChain
- 需要状态管理、循环、条件分支 → LangGraph
- LangChain 负责任务执行，LangGraph 负责流程编排

---

### Phase 6：MCP 协议（0.5 天，加分项）

| 序号 | 主题 | 笔记 |
|------|------|------|
| 57 | MCP 概述 | [MCP是什么？](<speed-review/什么是MCP(Model Context Protocol>)协议？它解.md) |
| 58 | MCP 架构 | [基本架构](<speed-review/MCP(Model Context Protocol>)协议的基本架构是.md) |
| 59 | MCP Resource | [Resource管理](<speed-review/MCP(Model Context Protocol>)协议中的Reso.md) |
| 60 | MCP 传输 | [传输方式](<speed-review/MCP(Model Context Protocol>)协议支持哪些传输.md) |
| 61 | MCP 安全 | [安全机制](<speed-review/MCP(Model Context Protocol>)协议的安全机制包.md) |
| 62 | MCP 开发 | [MCP开发实战](<speed-review/如何使用MCP(Model Context Protocol>)协议开发.md) |

**核心记忆点：**
- MCP = Anthropic 推出的 LLM 与外部系统连接的标准化协议
- 架构：Client-Server 模式，JSON-RPC 2.0 通信
- 解决的核心问题：每对接一个外部系统就要写定制连接器 → 统一接口规范
- 传输方式：stdio（本地）/ SSE（远程 HTTP）
- 安全：传输加密、权限控制、沙盒执行、审计日志

---

### Phase 7：评测与工程化（2 天，区分度最高）

#### 评测

| 序号 | 主题 | 笔记 |
|------|------|------|
| 63 | 性能评估 | [性能评估指标](<speed-review/%E5%A6%82%E4%BD%95%E8%AF%84%E4%BC%B0AI%20Agent%E7%9A%84%E6%80%A7%E8%83%BD%EF%BC%9F%E6%9C%89%E5%93%AA%E4%BA%9B%E5%85%B3%E9%94%AE%E6%8C%87%E6%A0%87%EF%BC%9F.md>) |
| 64 | 响应质量 | [响应质量评估](<speed-review/%E5%A6%82%E4%BD%95%E8%AF%84%E4%BC%B0%E5%92%8C%E6%94%B9%E8%BF%9BAI%20Agent%E7%9A%84%E5%93%8D%E5%BA%94%E8%B4%A8%E9%87%8F%EF%BC%9F.md>) |
| 65 | AgentBench | [AgentBench评测](<speed-review/AgentBench%E8%AF%84%E6%B5%8B%E6%A1%86%E6%9E%B6%E5%8C%85%E5%90%AB%E5%93%AA%E4%BA%9B%E7%BB%B4%E5%BA%A6%EF%BC%9F%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1Agent%E7%9A%84benc.md>) |
| 66 | WebArena | [WebArena评测](<speed-review/WebArena%E8%AF%84%E6%B5%8B%E4%BB%BB%E5%8A%A1%E7%9A%84%E7%89%B9%E7%82%B9%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F%E5%A6%82%E4%BD%95%E8%AF%84%E4%BC%B0Agent%E7%9A%84%E7%BD%91%E9%A1%B5%E6%93%8D%E4%BD%9C%E8%83%BD%E5%8A%9B.md>) |
| 67 | AutoGPT / BabyAGI | [AutoGPT](<speed-review/AutoGPT%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F%E5%AE%83%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E8%87%AA%E4%B8%BB%E4%BB%BB%E5%8A%A1%E6%89%A7%E8%A1%8C%EF%BC%9F.md>) / [BabyAGI](<speed-review/BabyAGI%E4%B8%8EAutoGPT%E6%9C%89%E4%BB%80%E4%B9%88%E5%8C%BA%E5%88%AB%EF%BC%9F%E5%90%84%E8%87%AA%E7%9A%84%E4%BC%98%E7%BC%BA%E7%82%B9%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F.md>) |
| 68 | 测试用例 | [测试用例库](<speed-review/%E5%A6%82%E4%BD%95%E6%9E%84%E5%BB%BAAgent%E7%9A%84%E6%B5%8B%E8%AF%95%E7%94%A8%E4%BE%8B%E5%BA%93%EF%BC%9F%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95%E5%92%8C%E9%9B%86%E6%88%90%E6%B5%8B%E8%AF%95%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1%EF%BC%9F.md>) |

#### 生产工程化

| 序号 | 主题 | 笔记 |
|------|------|------|
| 69 | 错误处理 | [错误处理](<speed-review/AI%20Agent%E5%9C%A8%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B%E4%B8%AD%E5%8F%AF%E8%83%BD%E9%81%87%E5%88%B0%E5%93%AA%E4%BA%9B%E9%94%99%E8%AF%AF%EF%BC%9F%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%EF%BC%9F.md>) |
| 70 | 调试方法 | [调试问题](<speed-review/AI%20Agent%E5%BC%80%E5%8F%91%E4%B8%AD%E5%B8%B8%E8%A7%81%E7%9A%84%E8%B0%83%E8%AF%95%E9%97%AE%E9%A2%98%E6%9C%89%E5%93%AA%E4%BA%9B%EF%BC%9F%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%EF%BC%9F.md>) |
| 71 | 容错设计 | [容错设计](<speed-review/Agent%E5%AF%B9%E8%AF%9D%E6%A8%A1%E5%9D%97%E7%9A%84%E5%AE%B9%E9%94%99%E8%83%BD%E5%8A%9B%E6%80%8E%E4%B9%88%E8%AE%BE%E8%AE%A1%EF%BC%9F%E7%94%A8%E6%88%B7%E8%AF%B4%E8%AF%9D%E4%B8%8D%E6%B8%85%E6%A5%9A%E6%88%96%E6%9C%89%E6%AD%A7%E4%B9%89%E6%97%B6%E6%80%8E%E4%B9%88%E5%8A%9E%EF%BC%9F.md>) |
| 72 | 降级策略 | [降级策略](<speed-review/%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1Agent%E7%9A%84%E9%99%8D%E7%BA%A7%E7%AD%96%E7%BA%A7%EF%BC%9F%E5%9C%A8%E6%A8%A1%E5%9E%8B%E4%B8%8D%E5%8F%AF%E7%94%A8%E6%97%B6%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E6%9C%8D%E5%8A%A1%EF%BC%9F.md>) |
| 73 | 成本控制 | [成本计算与平衡](<speed-review/Agent%E7%9A%84%E6%88%90%E6%9C%AC%E5%A6%82%E4%BD%95%E8%AE%A1%E7%AE%97%EF%BC%9F%E5%A6%82%E4%BD%95%E5%9C%A8%E6%95%88%E6%9E%9C%E5%92%8C%E6%88%90%E6%9C%AC%E9%97%B4%E6%89%BE%E5%88%B0%E6%9C%80%E4%BC%98%E5%B9%B3%E8%A1%A1%E7%82%B9%EF%BC%9F.md>) |
| 74 | 监控指标 | [监控指标](<speed-review/Agent%E7%B3%BB%E7%BB%9F%E7%9A%84%E7%9B%91%E6%8E%A7%E6%8C%87%E6%A0%87%E6%9C%89%E5%93%AA%E4%BA%9B%EF%BC%9F%E5%A6%82%E4%BD%95%E5%AE%9E%E6%97%B6%E8%BF%BD%E8%B8%AAAgent%E7%9A%84%E6%89%A7%E8%A1%8C%E7%8A%B6%E6%80%81%EF%BC%9F.md>) |
| 75 | 提示词优化 | [提示词优化](<speed-review/%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83%E4%B8%ADAgent%E7%9A%84%E6%8F%90%E7%A4%BA%E8%AF%8D%E5%A6%82%E4%BD%95%E8%BF%AD%E4%BB%A3%E4%BC%98%E5%8C%96%EF%BC%9FA_B%E6%B5%8B%E8%AF%95%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1%EF%BC%9F.md>) |
| 76 | Agentic Workflow | [Agentic Workflow](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AFAgentic%20Workflow%EF%BC%9F%E4%B8%8E%E4%BC%A0%E7%BB%9F%E5%B7%A5%E4%BD%9C%E6%B5%81%E6%9C%89%E4%BB%80%E4%B9%88%E5%8C%BA%E5%88%AB%EF%BC%9F.md>) |

#### 安全与可信

| 序号 | 主题 | 笔记 |
|------|------|------|
| 77 | 可解释性 | [可解释性](<speed-review/AI%20Agent%E7%9A%84%E5%8F%AF%E8%A7%A3%E9%87%8A%E6%80%A7%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%EF%BC%9F%E4%B8%BA%E4%BB%80%E4%B9%88%E9%87%8D%E8%A6%81%EF%BC%9F.md>) |
| 78 | 安全性 | [安全性](<speed-review/AI%20Agent%E7%9A%84%E5%AE%89%E5%85%A8%E6%80%A7%E9%97%AE%E9%A2%98%E6%9C%89%E5%93%AA%E4%BA%9B%EF%BC%9F%E5%A6%82%E4%BD%95%E9%98%B2%E8%8C%83%E6%81%B6%E6%84%8F%E8%A1%8C%E4%B8%BA%EF%BC%9F.md>) |
| 79 | 可控性权衡 | [可控性权衡](<speed-review/Agent%E7%9A%84%E5%8F%AF%E8%A7%A3%E9%87%8A%E6%80%A7%E4%B8%8E%E5%8F%AF%E6%8E%A7%E6%80%A7%E5%A6%82%E4%BD%95%E6%9D%83%E8%A1%A1%EF%BC%9F%E5%A6%82%E4%BD%95%E5%9C%A8%E8%87%AA%E4%B8%BB%E6%80%A7%E5%92%8C%E5%AE%89%E5%85%A8%E6%80%A7%E9%97%B4%E5%B9%B3%E8%A1%A1%EF%BC%9F.md>) |
| 80 | 幻觉问题 | [幻觉问题](<speed-review/AI%20Agent%E7%9A%84%E5%B9%BB%E8%A7%89%E9%97%AE%E9%A2%98%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%EF%BC%9F%E6%9C%89%E5%93%AA%E4%BA%9B%E9%AA%8C%E8%AF%81%E7%AD%96%E7%95%A5%EF%BC%9F.md>) |
| 81 | 失败分析 | [失败案例分析](<speed-review/Agent%E5%A4%B1%E8%B4%A5%E6%A1%88%E4%BE%8B%E5%A6%82%E4%BD%95%E5%88%86%E6%9E%90%EF%BC%9F%E5%B8%B8%E8%A7%81%E7%9A%84%E5%A4%B1%E8%B4%A5%E6%A8%A1%E5%BC%8F%E6%9C%89%E5%93%AA%E4%BA%9B%EF%BC%9F.md>) |
| 82 | 工具学习 | [工具学习](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AFAgent%E7%9A%84%E5%B7%A5%E5%85%B7%E5%AD%A6%E4%B9%A0%EF%BC%88Tool%20Learning%EF%BC%89%EF%BC%9F%E5%A6%82%E4%BD%95%E8%AE%A9Age.md>) |
| 83 | 网络搜索 | [网络搜索](<speed-review/%E5%A6%82%E4%BD%95%E7%BB%99AI%20Agent%E6%B7%BB%E5%8A%A0%E7%BD%91%E7%BB%9C%E6%90%9C%E7%B4%A2%E5%8A%9F%E8%83%BD%EF%BC%9F%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E4%BB%80%E4%B9%88%EF%BC%9F.md>) |
| 84 | 文件处理 | [文件处理Agent](<speed-review/%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6%E5%A4%84%E7%90%86AI%20Agent%E9%9C%80%E8%A6%81%E8%80%83%E8%99%91%E5%93%AA%E4%BA%9B%E6%8A%80%E6%9C%AF%E7%82%B9%EF%BC%9F.md>) |
| 85 | NLG 策略 | [NLG策略](<speed-review/%E8%87%AA%E7%84%B6%E8%AF%AD%E8%A8%80%E7%94%9F%E6%88%90%EF%BC%88NLG%EF%BC%89%E5%9C%A8%E5%AF%B9%E8%AF%9D%E9%87%8C%E6%80%8E%E4%B9%88%E7%94%A8%EF%BC%9F%E6%A8%A1%E6%9D%BF%E3%80%81%E6%A3%80%E7%B4%A2%E3%80%81%E7%94%9F%E6%88%90%E5%90%84%E6%9C%89%E4%BB%80%E4%B9%88%E4%BC%98%E7%BC%BA%E7%82%B9%EF%BC%9F.md>) |

---

## 三、面试高频题 Top 20（按出现频率排序）

### 第一梯队：几乎必考

| # | 题目 | 关键答法 | 对应笔记 |
|---|------|----------|----------|
| 1 | **什么是 AI Agent？和传统程序有什么区别？** | 感知-决策-执行闭环 vs 输入-处理-输出线性；概率性推理 vs 确定性执行 | [基础概念 #1](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AFAI%20Agent%EF%BC%9F%E4%B8%8E%E4%BC%A0%E7%BB%9F%E8%BD%AF%E4%BB%B6%E7%A8%8B%E5%BA%8F%E6%9C%89%E4%BB%80%E4%B9%88%E6%9C%AC%E8%B4%A8%E5%8C%BA%E5%88%AB%EF%BC%9F.md>) |
| 2 | **ReAct 模式是什么？和 CoT 有什么区别？** | Reasoning+Acting 交替；CoT 闭卷→ReAct 开卷；思考-行动-观察循环 | [推理模式 #5](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AFReAct%E6%A8%A1%E5%BC%8F%EF%BC%9F%E5%AE%83%E5%A6%82%E4%BD%95%E6%8F%90%E5%8D%87AI%20Agent%E7%9A%84%E6%8E%A8%E7%90%86%E8%83%BD%E5%8A%9B%EF%BC%9F.md>) |
| 3 | **Agent 的记忆机制怎么设计？** | 短期记忆（上下文窗口）+ 长期记忆（向量数据库）；对话窗口策略 | [记忆 #13](<speed-review/AI%20Agent%E7%9A%84%E8%AE%B0%E5%BF%86%E6%9C%BA%E5%88%B6%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1%EF%BC%9F%E7%9F%AD%E6%9C%9F%E8%AE%B0%E5%BF%86%E5%92%8C%E9%95%BF%E6%9C%9F%E8%AE%B0%E5%BF%86%E7%9A%84%E5%8C%BA%E5%88%AB%EF%BC%9F.md>) |
| 4 | **Multi-Agent 系统怎么设计？** | 层级/扁平/混合架构；角色分工；通信协议；冲突解决 | [MAS #20](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AF%E5%A4%9AAI%20Agent%E7%B3%BB%E7%BB%9F%EF%BC%88Multi-Agent%20System%EF%BC%89%EF%BC%9F.md>) |
| 5 | **LangChain 和 LangGraph 有什么区别？** | LangChain = 任务执行链；LangGraph = 状态编排图；简单用前者，复杂用后者 | [框架 #31](<speed-review/LangChain%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F%E5%AE%83%E8%A7%A3%E5%86%B3%E4%BA%86LLM%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E7%9A%84%E5%93%AA%E4%BA%9B%E9%97%AE%E9%A2%98%EF%BC%9F.md>) / [#44](<speed-review/LangGraph%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F%E5%AE%83%E4%B8%8ELangChain%E6%9C%89%E4%BB%80%E4%B9%88%E5%85%B3%E7%B3%BB%EF%BC%9F.md>) |

### 第二梯队：高频出现

| # | 题目 | 关键答法 | 对应笔记 |
|---|------|----------|----------|
| 6 | **Agent 怎么调用外部工具？** | Tool 描述 + 参数 schema + 结果解析；Function Calling | [基础 #3](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AF%E5%B7%A5%E5%85%B7%E4%BD%BF%E7%94%A8AI%20Agent%EF%BC%88Tool-using%20Agent%EF%BC%89%EF%BC%9F%E5%B7%A5.md>) |
| 7 | **Agent 的幻觉问题怎么解决？** | RAG 增强、结果验证、Self-Consistency、多 Agent 交叉验证 | [安全 #80](<speed-review/AI%20Agent%E7%9A%84%E5%B9%BB%E8%A7%89%E9%97%AE%E9%A2%98%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%EF%BC%9F%E6%9C%89%E5%93%AA%E4%BA%9B%E9%AA%8C%E8%AF%81%E7%AD%96%E7%95%A5%EF%BC%9F.md>) |
| 8 | **如何评估 Agent 性能？** | 任务完成率、响应时间、Token 成本、准确率/F1；AgentBench/WebArena | [评测 #63](<speed-review/%E5%A6%82%E4%BD%95%E8%AF%84%E4%BC%B0AI%20Agent%E7%9A%84%E6%80%A7%E8%83%BD%EF%BC%9F%E6%9C%89%E5%93%AA%E4%BA%9B%E5%85%B3%E9%94%AE%E6%8C%87%E6%A0%87%EF%BC%9F.md>) |
| 9 | **Agent 的安全性怎么保障？** | Prompt 注入防护、最小权限、沙盒、审计日志、人工审核 | [安全 #78](<speed-review/AI%20Agent%E7%9A%84%E5%AE%89%E5%85%A8%E6%80%A7%E9%97%AE%E9%A2%98%E6%9C%89%E5%93%AA%E4%BA%9B%EF%BC%9F%E5%A6%82%E4%BD%95%E9%98%B2%E8%8C%83%E6%81%B6%E6%84%8F%E8%A1%8C%E4%B8%BA%EF%BC%9F.md>) |
| 10 | **如何做 Agent 的错误处理和降级？** | 重试 + 回退 + 降级到规则引擎 + 人工接管 | [工程 #69](<speed-review/AI%20Agent%E5%9C%A8%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B%E4%B8%AD%E5%8F%AF%E8%83%BD%E9%81%87%E5%88%B0%E5%93%AA%E4%BA%9B%E9%94%99%E8%AF%AF%EF%BC%9F%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%EF%BC%9F.md>) / [#72](<speed-review/%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1Agent%E7%9A%84%E9%99%8D%E7%BA%A7%E7%AD%96%E7%BA%A7%EF%BC%9F%E5%9C%A8%E6%A8%A1%E5%9E%8B%E4%B8%8D%E5%8F%AF%E7%94%A8%E6%97%B6%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E6%9C%8D%E5%8A%A1%EF%BC%9F.md>) |

### 第三梯队：常作追问

| # | 题目 | 关键答法 | 对应笔记 |
|---|------|----------|----------|
| 11 | **Plan-and-Execute vs ReAct？** | 前者先规划再执行适合结构化任务；后者交替更灵活 | [推理 #6](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AFPlan-and-Execute%E6%A8%A1%E5%BC%8F%EF%BC%9F%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E4%BB%BB%E5%8A%A1%E7%9A%84%E5%88%86%E5%B1%82%E8%A7%84%E5%88%92%EF%BC%9F.md>) |
| 12 | **什么是 Agentic Workflow？** | Agent 自主规划步骤、选工具、评估结果 vs 传统预定义流程 | [工程 #76](<speed-review/%E4%BB%80%E4%B9%88%E6%98%AFAgentic%20Workflow%EF%BC%9F%E4%B8%8E%E4%BC%A0%E7%BB%9F%E5%B7%A5%E4%BD%9C%E6%B5%81%E6%9C%89%E4%BB%80%E4%B9%88%E5%8C%BA%E5%88%AB%EF%BC%9F.md>) |
| 13 | **MCP 协议是什么？** | Anthropic 的 LLM-外部系统连接标准；Client-Server + JSON-RPC | [MCP #57](<speed-review/什么是MCP(Model Context Protocol>)协议？它解.md) |
| 14 | **Agent 间怎么通信？** | 消息总线 / 直接调用；结构化消息格式（JSON Schema）；同步/异步 | [MAS #21](<speed-review/Agent%E4%B9%8B%E9%97%B4%E5%A6%82%E4%BD%95%E8%BF%9B%E8%A1%8C%E6%9C%89%E6%95%88%E9%80%9A%E4%BF%A1%EF%BC%9F%E6%B6%88%E6%81%AF%E6%A0%BC%E5%BC%8F%E5%92%8C%E5%8D%8F%E8%AE%AE%E5%A6%82%E4%BD%95%E5%AE%9A%E4%B9%89%EF%BC%9F.md>) |
| 15 | **涌现行为是什么？如何利用？** | 整体>部分之和；监控+引导+抑制；有益强化、有害熔断 | [MAS #29](<speed-review/Multi-Agent%E7%B3%BB%E7%BB%9F%E7%9A%84%E6%B6%8C%E7%8E%B0%E8%A1%8C%E4%B8%BA%EF%BC%88Emergent%20Behavio.md>) |
| 16 | **Reflexion 和 Self-Refinement 区别？** | Reflexion 跨轮次记忆反思；Self-Refinement 同轮次内迭代 | [#8](<speed-review/Reflexion%E6%9C%BA%E5%88%B6%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%EF%BC%9F%E5%A6%82%E4%BD%95%E9%80%9A%E8%BF%87%E8%87%AA%E6%88%91%E5%8F%8D%E6%80%9D%E6%8F%90%E5%8D%87Agent%E6%80%A7%E8%83%BD%EF%BC%9F.md>) / [#10](<speed-review/Agent%E7%9A%84Self-Refinement%E6%98%AF%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E7%9A%84%EF%BC%9F%E9%9C%80%E8%A6%81%E5%93%AA%E4%BA%9B%E5%85%B3%E9%94%AE%E7%BB%84.md>) |
| 17 | **如何控制 Agent 成本？** | Token 计费模型、缓存、降级小模型、批量处理、Prompt 精简 | [工程 #73](<speed-review/Agent%E7%9A%84%E6%88%90%E6%9C%AC%E5%A6%82%E4%BD%95%E8%AE%A1%E7%AE%97%EF%BC%9F%E5%A6%82%E4%BD%95%E5%9C%A8%E6%95%88%E6%9E%9C%E5%92%8C%E6%88%90%E6%9C%AC%E9%97%B4%E6%89%BE%E5%88%B0%E6%9C%80%E4%BC%98%E5%B9%B3%E8%A1%A1%E7%82%B9%EF%BC%9F.md>) |
| 18 | **Agent 的可解释性怎么做？** | 推理链可视化、决策日志、注意力分析、CoT 路径追踪 | [安全 #77](<speed-review/AI%20Agent%E7%9A%84%E5%8F%AF%E8%A7%A3%E9%87%8A%E6%80%A7%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%EF%BC%9F%E4%B8%BA%E4%BB%80%E4%B9%88%E9%87%8D%E8%A6%81%EF%BC%9F.md>) |
| 19 | **如何设计 Agent 的监控？** | 分层：Agent 级（延迟/错误率）、交互级（消息频率）、系统级（吞吐/成功率） | [工程 #74](<speed-review/Agent%E7%B3%BB%E7%BB%9F%E7%9A%84%E7%9B%91%E6%8E%A7%E6%8C%87%E6%A0%87%E6%9C%89%E5%93%AA%E4%BA%9B%EF%BC%9F%E5%A6%82%E4%BD%95%E5%AE%9E%E6%97%B6%E8%BF%BD%E8%B8%AAAgent%E7%9A%84%E6%89%A7%E8%A1%8C%E7%8A%B6%E6%80%81%EF%BC%9F.md>) |
| 20 | **Agent 测试怎么做？** | 单元测试（工具/组件）+ 集成测试（端到端流程）+ 回归测试 | [评测 #68](<speed-review/%E5%A6%82%E4%BD%95%E6%9E%84%E5%BB%BAAgent%E7%9A%84%E6%B5%8B%E8%AF%95%E7%94%A8%E4%BE%8B%E5%BA%93%EF%BC%9F%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95%E5%92%8C%E9%9B%86%E6%88%90%E6%B5%8B%E8%AF%95%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1%EF%BC%9F.md>) |

---

## 四、面试回答框架

### 万能答题结构（STAR-Plus 变体）

```
1. 核心定义（一句话说清楚是什么）
2. 关键机制（2-3 个要点说明怎么工作）
3. 对比/类比（和相似技术的区别）
4. 实际应用（举一个具体场景）
5. 工程思考（成本/安全/可维护性的权衡）
```

### 示例：回答 "什么是 ReAct？"

> **定义**: ReAct 是 Reasoning + Acting 的结合，让 Agent 交替进行推理和行动。
>
> **机制**: 每轮经历 思考→行动→观察 循环。思考阶段分析现状规划下一步，行动阶段调用工具/API，观察阶段处理结果更新上下文。
>
> **对比**: 和 Chain-of-Thought 的区别是 CoT 只在"脑子里"推理（闭卷），ReAct 可以主动获取外部信息（开卷）。和 Plan-and-Execute 的区别是 ReAct 逐步交替更灵活，后者先完整规划再执行更适合确定性流程。
>
> **应用**: 电商客服场景——用户问"订单什么时候到"，Agent 先推理需要哪些信息，再查订单和物流接口，基于结果生成回答。
>
> **权衡**: 优点是灵活性和可解释性好，缺点是 LLM 调用次数多导致成本高，需要设置最大循环次数防止无限循环。

---

## 五、知识关联图谱

```
                        ┌─────────────┐
                        │  AI Agent   │
                        │   定义本质   │
                        └──────┬──────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
        ┌─────▼─────┐   ┌─────▼─────┐   ┌─────▼─────┐
        │  推理引擎   │   │  记忆系统   │   │  工具调用   │
        │            │   │            │   │            │
        │ ReAct      │   │ 短期(上下文) │   │ Function   │
        │ Plan-Exec  │   │ 长期(向量DB) │   │ Calling    │
        │ ReWOO      │   │ 对话窗口    │   │ MCP协议    │
        │ Reflexion  │   │            │   │ Code Intp. │
        │ GoT        │   └─────┬──────┘   └─────┬──────┘
        │ Self-Refine│         │                │
        └─────┬──────┘         │                │
              │                │                │
              └────────────────┼────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │   Agent 编排框架      │
                    │                     │
                    │  LangChain ── 链式   │
                    │  LangGraph ── 图状态  │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
        ┌─────▼─────┐   ┌─────▼─────┐   ┌─────▼─────┐
        │ Multi-Agent│   │   评测     │   │  生产工程化  │
        │            │   │            │   │            │
        │ 层级式      │   │ AgentBench │   │ 监控告警    │
        │ 扁平式      │   │ WebArena   │   │ 降级容错    │
        │ 辩论式      │   │ 测试用例    │   │ 成本控制    │
        │ 涌现行为    │   │            │   │ 安全防护    │
        └────────────┘   └────────────┘   └────────────┘
```

---

## 六、各主题关联矩阵（交叉复习）

|  | ReAct | 记忆 | 工具 | LangGraph | Multi-Agent | 安全 | 评测 |
|---|---|---|---|---|---|---|---|
| **ReAct** | - | 观察结果需记忆 | 行动=调用工具 | LangGraph 实现循环 | 多 Agent 各自 ReAct | 工具调用需安全检查 | 循环次数影响成本 |
| **记忆** | 观察需存储 | - | 长期记忆靠向量DB | 状态=全局记忆 | 共享记忆 vs 私有记忆 | 记忆可能泄露敏感信息 | 记忆检索准确率 |
| **工具** | ReAct 调用工具 | 工具结果存记忆 | - | 节点封装工具 | 工具分配给专门Agent | 工具权限控制 | 工具调用成功率 |
| **LangGraph** | 循环实现ReAct | 状态持久化 | 节点封装工具 | - | 多子图=多Agent | 人机交互做审核 | 执行追踪 |
| **Multi-Agent** | 各Agent可ReAct | 共享/独立记忆 | Agent专精不同工具 | 子图编排 | - | Agent间权限隔离 | 集体效率指标 |
| **安全** | 防Prompt注入 | 数据脱敏 | 工具沙盒 | 审批节点 | 防有害涌现 | - | 红队测试 |
| **评测** | 任务完成率 | 检索召回率 | 调用成功率 | 流程覆盖率 | 协作效率 | 安全测试 | - |

---

## 七、面试加分话术

1. **展现工程思维**: "Agent 的不确定性是核心挑战，需要通过降级策略、监控告警、人工兜底来保证生产可用性"
2. **展现架构视野**: "简单场景用 LangChain 链式调用，复杂场景需要 LangGraph 做状态编排，两者互补而非替代"
3. **展现成本意识**: "每次 LLM 调用都有成本，ReAct 模式虽然灵活但调用次数多，ReWOO 通过批量执行降低开销"
4. **展现安全意识**: "Agent 安全是多层防御：输入验证 → 权限最小化 → 沙盒执行 → 行为监控 → 审计日志 → 人工审核"
5. **展现系统思维**: "Multi-Agent 设计的关键不是单个 Agent 多强，而是通信协议、角色分工、冲突解决机制的系统性设计"

---

*共 86 篇笔记，按此知识地图复习约需 10-14 天完成一轮。建议第一轮按 Phase 顺序通读，第二轮针对 Top 20 高频题精练回答框架。*
