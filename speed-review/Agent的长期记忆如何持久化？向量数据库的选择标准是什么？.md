# Agent的长期记忆如何持久化？向量数据库的选择标准是什么？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心定义
- Agent的长期记忆持久化通过将文本（对话历史、用户偏好等）经Embedding模型转换为向量，存入向量数据库，后续通过语义检索召回相关记忆。
- 核心解决痛点：突破传统数据库精确匹配的限制，实现针对海量历史记录的快速语义相似度检索。

## 机制与原理
- **分层记忆架构**：短期记忆（当前对话上下文，存内存/Redis）与长期记忆（跨会话保留，存向量数据库）。
- **向量化原理**：将文本映射到高维空间（如768/1536维），语义相近的文本在空间中距离更近，通过余弦/欧氏距离计算相似度。
- **底层索引结构**：传统数据库B+树针对范围查询优化；向量数据库采用HNSW图索引（适合高维稠密向量，对数级查询）或IVF聚类索引（空间分区局部搜索）。
- **混合检索机制**：实际业务常需结合语义与业务规则。采用“向量粗排召回候选 -> 元数据过滤（如时间/用户ID） -> 结合时间衰减/重要性进行加权精排”的链路。

## 对比速记
| 数据库类型 | 索引结构 | 查询优化目标 | 典型应用场景 |
| :--- | :--- | :--- | :--- |
| **传统数据库** | B+树 | 范围查询优化 (如 `BETWEEN`) | 精确匹配、条件筛选 |
| **向量数据库** | HNSW图 / IVF聚类 | 高维相似度计算 (找Top-K) | 语义理解、相关性召回 |

| 向量数据库方案 | 核心特点 | 适用场景 |
| :--- | :--- | :--- |
| **Chroma** | 嵌入式方案，无需额外部署 | 个人Demo、技术验证 |
| **Qdrant** | 提供Docker镜像和RESTful API | 团队协作、中小型项目 |
| **Milvus** | 分层架构，极致性能，千万级个位数ms延迟 | 百万日活、大规模企业级应用 |
| **Weaviate** | 原生支持GraphQL，支持向量与图谱联合查询 | 知识图谱类复杂Agent应用 |
| **Pinecone** | 全托管零运维，自动扩缩容 | 创业团队快速交付（需警惕账单爆发） |

## 代码示例
```java
// 混合检索与打分重排机制实现
public class HybridSearchService {
    private final VectorDatabase vectorDB;

    public List<Memory> searchMemories(SearchRequest request) {
        // 1. 向量粗排：召回3倍候选集
        List<VectorMatch> candidates = vectorDB.similaritySearch(
            request.getQueryEmbedding(),
            request.getTopK() * 3
        );

        // 2. 元数据过滤（时间范围、用户ID等）
        Stream<VectorMatch> filtered = candidates.stream()
            .filter(m -> isWithinTimeRange(m, request.getTimeRange()))
            .filter(m -> matchesUserSegment(m, request.getUserId()));

        // 3. 混合打分重排（向量相似度 + 时间新近性 + 重要性）
        return filtered
            .map(m -> new ScoredMemory(m, calculateHybridScore(m, request)))
            .sorted(Comparator.comparing(ScoredMemory::getScore).reversed())
            .limit(request.getTopK())
            .collect(Collectors.toList());
    }

    private double calculateHybridScore(VectorMatch match, SearchRequest req) {
        double vectorScore = match.getSimilarity();
        double recencyScore = Math.exp(-ageInDays / 30.0); // 30天时间衰减模型
        double importanceScore = (double) match.getMetadata().getOrDefault("importance", 0.5);
        // 权重配比：相似度60% + 新近性30% + 重要性10%
        return 0.6 * vectorScore + 0.3 * recencyScore + 0.1 * importanceScore;
    }
}
```

## 工程要点
- **性能调优**：索引参数（如HNSW的M和efConstruction）是时空权衡，读多写少场景可增大参数换取查询性能。
- **缓存策略**：针对热点查询（如FAQ）引入LRU缓存，需在记忆更新时主动失效相关缓存以保证一致性。
- **新用户冷启动**：准备通用基础记忆库，前期以通用记忆为主（70%），随交互增加逐步降低权重，平滑过渡到个性化记忆。
- **成本优化**：对三个月以上未访问的冷数据采用PQ乘积量化压缩（如1536维float32降至128维int8）；对稳定用户画像采用离线批处理预计算，降低Embedding API调用量。
- **数据迁移**：跨库迁移需双写并行运行，对比验证准确率无下降后灰度切换。
- **多模态演进**：使用CLIP等跨模态模型将图文映射到统一向量空间，检索时可对多模态特征分配不同权重。
