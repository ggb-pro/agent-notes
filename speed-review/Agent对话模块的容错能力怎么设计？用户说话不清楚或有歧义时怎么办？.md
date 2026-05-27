# Agent对话模块的容错能力怎么设计？用户说话不清楚或有歧义时怎么办？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- 对话系统容错能力核心在于多层识别与主动澄清机制，目标是让对话继续，而非卡死在失败节点。
- **关键特征**：多层防御、置信度阈值触发、主动消歧、状态可回滚、动态降级。

## 机制与原理
- **预防层**：针对语音ASR错误加纠错模块；针对文本做分词归一化（如拆分无空格连写字符）、繁简转换与全半角统一。
- **识别层**：综合评估意图置信度、槽位完整度和上下文一致性。若意图冲突触发消歧，槽位缺失触发追问，上下文冲突触发确认。
- **补救层**：根据已提供信息个性化澄清；对TopK候选结果做合并过滤（结合上下文与历史偏好）；检测到连续负面词汇或达最大轮次限制时，主动转人工。
- **状态管理**：维护上下文记忆栈与对话快照。检测到否定词（如“不对”、“改成”）时，精准定位并覆盖对应槽位，同时阻断错误传播，标记依赖该槽位的后续操作为待重算。
- **动态阈值调整**：不同意图设置差异化阈值（高频低风险操作如查询设0.7，高风险如退款设0.85）。通过滑动窗口统计近期反馈，自适应微调阈值。

## 对比速记
- **多意图关系处理**：并列关系（如查订单+改地址）让用户选择优先级；递进关系（如查订单+取消）按业务流程串行执行。
- **兜底话术分级策略**：第一次失败用通用兜底（“没听清，请再说一遍”）；第二次用领域兜底（列出高频业务选项）；第三次用智能推荐（结合用户历史行为猜测意图）或转人工。

## 代码示例
```java
// 动态置信度阈值调整管理器
public class ConfidenceThresholdManager {
    private volatile double intentThreshold = 0.75;
    private final ConcurrentLinkedQueue<FeedbackEvent> recentFeedback = new ConcurrentLinkedQueue<>();
    private static final int SAMPLE_SIZE = 1000;

    public void adjustThreshold() {
        List<FeedbackEvent> samples = getRecentSamples(SAMPLE_SIZE);
        if (samples.size() < SAMPLE_SIZE) return;

        double clarifyRejectRate = calculateClarifyRejectRate(samples);
        double transferRate = calculateTransferRate(samples);

        if (clarifyRejectRate > 0.3) {
            // 过度澄清，提高阈值减少误触发
            intentThreshold = Math.min(0.9, intentThreshold + 0.02);
        } else if (transferRate > 0.15) {
            // 转人工过多，降低阈值增强理解能力
            intentThreshold = Math.max(0.6, intentThreshold - 0.02);
        }
    }
    // ... 省略样本获取与指标计算方法 ...
}
```

## 工程要点
- **指标监控**：重点追踪澄清触发率（目标<10%）、澄清成功率、降级率（目标<5%）。设置实时告警（如10分钟内降级率超5%）。
- **Badcase复盘**：每周人工抽查降级案例，针对高频失败原因（如方言、新词）补充训练集或优化规则。
- **场景化权衡**：根据业务紧急度与风险调整容错策略。高安全场景（如银行）倾向严格二次确认；低风险场景（如音乐播放）可大胆推测快速响应。
