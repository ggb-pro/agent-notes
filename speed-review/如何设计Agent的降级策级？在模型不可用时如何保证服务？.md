# 如何设计Agent的降级策级？在模型不可用时如何保证服务？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- Agent降级策略是为应对大模型API不可用或响应异常而建立的多层防护机制。
- **关键特征**：面临能力边界突变（从理解复杂意图退化为执行固定逻辑），需保证多轮对话的状态连续性，实现平滑过渡。

## 机制与原理
- **模型层降级（第一道防线）**：主力模型超时或限流时，通过模型路由组件切换至备用模型（如GPT-4切至GPT-3.5或本地Llama）。要求统一输入输出格式，实现业务层无感知。
- **能力层降级（第二道防线）**：模型完全不可用时，退化为基于规则的系统。通过预定义决策树和关键词匹配处理高频场景（如查订单、退换货）。
- **服务层降级（最后防线）**：关闭复杂工具调用，简化为单步查询或直接返回缓存结果。极端情况下返回友好提示并引导转人工。
- **触发与恢复机制**：采用熔断器模式，监控P99延迟、错误率分布和业务成功率。使用滑动窗口（如60秒）避免误判。恢复时采用灰度放量逐步切回主模型。
- **状态持久化**：降级极易丢失上下文，必须将关键槽位信息和对话历史持久化至Redis等缓存，保证降级后能延续对话。

## 对比速记
| 降级层级 | 适用场景 | 核心机制 | 用户体验影响 |
| :--- | :--- | :--- | :--- |
| **模型层** | 暂时性抖动、响应慢 | 切换备用/开源模型 | 极小（可能回答质量略降） |
| **能力层** | 模型API完全不可用 | 规则引擎、关键词匹配 | 中等（只能处理固定流程） |
| **服务层** | 系统压力极大/全面故障 | 返回缓存、限流、友好错误提示 | 大（功能受限或转人工） |

## 代码示例
```java
// 降级上下文状态管理：保证降级过程中的对话连续性
public class ConversationContext implements Serializable {
    private String sessionId;
    private Map<String, String> slots; // 存储已收集的槽位信息
    private List<Message> history; // 对话历史
    private DegradationLevel currentLevel; // 当前降级级别
    private long lastUpdateTime;

    public void saveToCache() {
        redisTemplate.opsForValue().set(
            "session:" + sessionId,
            JSON.toJSONString(this),
            Duration.ofHours(1)
        );
    }

    public void restoreFromCache(String sessionId) {
        String json = redisTemplate.opsForValue().get("session:" + sessionId);
        if (json != null) {
            ConversationContext ctx = JSON.parseObject(json, ConversationContext.class);
            this.slots = ctx.getSlots();
            this.history = ctx.getHistory();
        }
    }
}
```

## 工程要点
- **阈值设定**：根据历史数据分位数设定（如取过去一个月P95延迟的1.5倍），上线后通过AB测试结合用户留存率和投诉率持续调优。
- **用户预期管理**：降级时需在响应中明确标识能力受限（如“当前仅支持基础查询”），将技术降级包装为用户引导。
- **异步补偿机制**：记录降级期间未完成的复杂查询，引导用户留言，待模型恢复后异步处理并主动推送结果。
- **混沌工程演练**：在测试环境定期模拟模型故障（如随机断开连接），验证降级决策引擎和预案的有效性。
- **成本与业务权衡**：评估各场景业务优先级。简单查询可接受降级，复杂核心业务（如金融咨询）宁可限流排队或直接转人工，避免答非所问。
