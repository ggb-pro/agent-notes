# MCP协议中的Resource和Tool有什么区别？各自的使用场景？

> **难度**: 困难 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- **Resource**：静态数据源，提供可读取的信息内容（如文件、数据库记录）。交互为单向读取，回答“这里有什么数据”。
- **Tool**：可执行的功能单元，代表动作或计算过程（如发送邮件、执行计算）。交互为双向执行，回答“我能帮你做什么事”。

## 机制与原理
- **数据流向**：Resource是单向的数据读取（Pull模式）；Tool是双向的参数传入与结果返回（Call模式）。
- **幂等性**：Resource响应通常是幂等的（同URI同时间返回相同数据）；Tool响应通常是非幂等的，可能产生副作用（如修改数据库、发邮件）。
- **架构解耦**：Resource扩展了AI模型的“感知能力”（获取上下文），Tool扩展了AI模型的“行动能力”（执行业务操作）。

## 对比速记
| 维度 | Resource | Tool |
| :--- | :--- | :--- |
| **本质** | 数据容器 | 能力单元/动作 |
| **交互模式** | Pull（通过URI请求数据） | Call（传入参数调用功能） |
| **副作用** | 无（纯读取） | 有（可能改变系统状态） |
| **缓存友好度** | 高（数据相对稳定） | 低（涉及实时计算与副作用） |
| **权限控制重点**| 数据访问权限 | 操作执行权限 |
| **典型场景** | 获取配置、读取用户档案 | 发送通知、执行复杂计算 |

## 代码示例
```java
// Resource的典型实现模式：直接返回数据
@MCPResource("user://profile/{userId}")
public UserProfile getUserProfile(@PathParam("userId") String userId) {
    return userService.getProfile(userId);  
}

// Tool的典型实现模式：执行计算或业务逻辑
@MCPTool("calculatePrice")
public PriceResult calculatePrice(@Param("productId") String productId,
                                 @Param("coupons") List<String> coupons) {
    return priceCalculator.calculate(productId, coupons);  
}
```

## 工程要点
- **选型判断**：获取已存在的信息用Resource；需根据参数实时计算或改变系统状态用Tool。
- **错误处理**：Resource错误多为数据不存在或权限问题；Tool错误更复杂，需处理业务逻辑异常、外部依赖失败及重试策略。
- **监控指标**：Resource侧重访问频率、响应时间、缓存命中率；Tool侧重执行成功率、业务错误率、执行时长。
- **落地策略**：优先将稳定的数据查询包装为Resource，核心业务操作包装为Tool，渐进式集成。
