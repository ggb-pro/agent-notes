# 什么是Plan-and-Execute模式？如何实现任务的分层规划？

> **难度**: 中等 | **分类**: AI Agent理论与框架 | **标签**: AI

## 核心回答

Plan-and-Execute模式是一种将复杂任务分解为规划和执行两个阶段的Agent架构模式。它先通过规划器生成完整的任务执行计划，再由执行器按步骤执行，这种模式能有效降低大模型在处理复杂任务时的推理负担。

核心工作流程是这样的：规划阶段，LLM接收用户目标后生成分步计划，输出类似"查询数据库获取用户信息→分析消费记录→生成推荐报告"这样的步骤序列；执行阶段，Agent按计划逐步调用工具完成每个子任务，并将结果传递给下一步。

分层规划的实现主要依赖递归分解机制。当某个子任务仍然复杂时，可以将其作为新目标再次触发规划过程，形成树状的任务层级。比如处理"生成季度财务分析"时，可以先规划出"数据采集-数据清洗-统计分析-报告生成"四个一级任务，然后对"统计分析"再细分为"计算同比增长率-计算环比增长率-异常值检测"等二级任务。实现上通常需要维护一个任务队列和状态管理器，记录每个任务的依赖关系和完成状态。ReWOO、LLMCompiler等框架都采用这种模式，它们的优势在于计划可预先生成、便于并行执行和异常重试，相比ReAct模式的逐步推理更适合处理确定性强、步骤清晰的复杂任务。

## 扩展分析

设计理念与核心机制
在Agent处理复杂任务时，其实面临一个核心矛盾——大模型很聪明但也很贵，每次调用都要消耗大量Token。如果像ReAct那样每执行一步都要重新推理下一步该干啥，处理一个十几步的任务可能要调用十几次大模型。Plan-and-Execute模式的设计思路就是把这个过程解耦，规划的时候集中用一次大模型把路线图画出来，执行的时候就按图索骥，不需要反复调用。这和ReAct模式最大的区别是，ReAct是走一步看一步，每次执行后都要重新思考下一步怎么办；而Plan-and-Execute是一次性想清楚整个路径，然后闷头把事情做完。

这种设计特别适合处理那些步骤相对确定的复杂任务。比如生成一份用户行为分析报告，其实步骤就那么几步——拉数据、清洗、统计、生成图表、写结论，这些步骤在开始前就能规划清楚。用Plan-and-Execute模式，大模型只需要在最开始调用一次做规划，后面就是机械执行，既省Token成本，执行过程也更可控。

整个架构的核心是Planner和Executor这两个角色的职责边界。Planner的工作是接收用户目标，调用大模型生成结构化的执行计划，这个计划通常是一个包含步骤序列、工具调用、数据依赖关系的数据结构。Executor的职责则单纯得多，它就是个执行引擎，按照计划里定义的顺序调用各种工具和服务，把每一步的输出作为下一步的输入传递下去。可以用一个简单的类比来理解：Planner就像项目经理制定工作计划，Executor像执行团队按计划干活，两者的关键区别是一个需要智能决策，一个只需要机械执行。

生成任务列表

依赖

依赖

用户目标

规划器

任务依赖图

执行器

任务1: 数据采集

任务2: 数据清洗

任务3: 统计分析

最终结果

分层规划本质上是个递归分解的过程。最顶层是用户的原始目标，比如"生成本季度营销活动效果分析"。Planner接到这个目标后，会将它分解成第一层任务序列，可能包括"提取活动数据""计算关键指标""生成可视化图表""撰写分析结论"这几个步骤。关键在于，当Planner发现某个子任务仍然很复杂，比如"计算关键指标"这一步涉及多个维度的数据处理时，它可以把这个子任务再次作为输入触发新一轮规划，生成第二层的细分步骤：计算ROI、计算转化率、计算用户获取成本、进行同比环比分析。这个过程可以一直递归下去，直到每个最底层的执行单元都简单到可以直接调用某个工具或API完成为止。

树的根节点是最终目标，第一层子节点是主要任务，第二层是子任务，叶子节点是可直接执行的原子操作。这个分解过程需要控制机制，实现上需要维护一个任务注册表，记录每个任务的当前状态——待执行、执行中、已完成、执行失败。同时还要维护任务之间的依赖关系，比如任务B必须等任务A完成后才能开始，任务C和任务D可以并行执行。这样Executor在执行时就能根据依赖图动态调度，既保证了逻辑正确性，又能最大化并行度提升效率。

生成季度营销分析

提取活动数据

计算关键指标

生成可视化图表

撰写分析结论

计算ROI

计算转化率

计算获客成本

同比环比分析

查询订单表

查询成本表

执行ROI计算

传统工作流引擎确实也有任务编排和执行的概念，但它的流程定义是开发者预先写死的，缺乏动态适应能力。Plan-and-Execute模式的关键区别在于，规划本身是由大模型动态生成的。面对同一个目标，在不同上下文下，Planner可能生成完全不同的执行计划。比如要生成销售报告，如果发现某个数据源不可用，Planner可以调整计划改用备用数据源或者修改分析维度。这种动态规划能力是传统工作流引擎不具备的。

还有一个关键点是规划与执行之间的反馈循环机制。当Executor在执行某个步骤时发现实际情况和预期不符，比如API调用失败、数据格式不匹配、中间结果异常等，它需要将这些信息反馈给Planner。Planner根据反馈重新评估，可能会修正后续计划，或者在当前步骤插入额外的处理逻辑。这个反馈循环让整个系统兼具规划的高效性和执行的灵活性。规划阶段降低了大模型调用频率，反馈机制又保证了面对异常情况时的适应能力，这是这个模式在工程实践中能真正落地的关键设计。

从设计思想的角度，Plan-and-Execute体现了一种重要的工程哲学——通过职责分离来管理复杂度。Planner专注于高层决策，不用关心底层执行细节；Executor专注于可靠执行，不需要具备全局规划能力。这种分离带来的好处非常明显：可控性更强，因为有了明确的执行计划，整个过程的进度和状态变得可追踪；可解释性更好，用户可以在执行前看到完整的计划，理解系统打算如何处理他的请求；可测试性也提升了，你可以单独测试Planner生成的计划是否合理，也可以单独测试Executor在给定计划下的执行逻辑是否正确。

工程实现与落地实践
在规划阶段，最核心的其实是设计好Planner的Prompt，让大模型能够输出结构化的计划。实现上可以通过精心设计的Prompt引导模型输出JSON格式的任务列表，每个任务包含名称、描述、依赖的前置任务ID、需要调用的工具名称这些字段。比如可以在Prompt里明确告诉模型，请将目标分解成3-5个主要步骤，如果某个步骤过于复杂可以标记需要进一步分解，然后用dependsOn字段指明它依赖哪些步骤完成后才能开始。

更优雅的做法是利用Function Calling机制。定义一套任务规划的Function Schema，把"创建任务""设置依赖关系""标记可并行执行"这些操作定义成函数，让模型通过函数调用的方式来构建执行计划。这样做的好处是输出格式更可控，也更容易做参数校验。

用LangGraph实现Plan-and-Execute，核心是构建一个状态图，其中Planner节点负责生成任务列表并更新到状态中，Executor节点从状态里读取下一个待执行任务，执行完后更新任务状态。关键是要设计好状态结构，通常会在State里维护一个tasks数组存所有任务，一个task_status字典记录每个任务的完成情况，还有一个results字典保存每个任务的输出结果。

先定义任务的数据结构，这是整个系统的基础：

```java
public class Task {
    private String id;
    private String description;
    private String toolName;
    private Map<String, Object> parameters;
    private List<String> dependencies;  // 依赖的前置任务ID
    private TaskStatus status;
    private boolean needsDecomposition;  // 是否需要进一步分解

    public Task(String id, String description) {
        this.id = id;
        this.description = description;
        this.dependencies = new ArrayList<>();
        this.status = TaskStatus.PENDING;
        this.parameters = new HashMap<>();
    }

    // Getter和Setter方法
    public String getId() { return id; }
    public String getDescription() { return description; }
    public List<String> getDependencies() { return dependencies; }
    public TaskStatus getStatus() { return status; }
    public void setStatus(TaskStatus status) { this.status = status; }
    public boolean needsDecomposition() { return needsDecomposition; }
}

enum TaskStatus {
    PENDING, RUNNING, COMPLETED, FAILED
}
```

这个Task类包含了执行一个任务所需的所有信息。dependencies字段是关键，它定义了任务之间的依赖关系，Executor会根据这个来决定执行顺序。needsDecomposition字段用于标记复杂任务，触发递归分解。

接下来实现Planner的核心逻辑，这部分最能体现对Prompt工程的理解：

```
public class Planner {
    private LLMClient llmClient;

    public List<Task> generatePlan(String userGoal) {
        String prompt = buildPlanningPrompt(userGoal);
        String response = llmClient.chat(prompt);
        return parsePlanFromResponse(response);
    }

    private String buildPlanningPrompt(String goal) {
        return String.format("""
            你是一个任务规划专家。用户的目标是：%s

```

            请将这个目标分解为可执行的步骤序列，以JSON格式输出：
            [
              {
                "id": "task_1",
                "description": "具体要做什么",
                "toolName": "需要调用的工具名称",
                "parameters": {"参数名": "参数值"},
                "dependencies": [],
                "needsDecomposition": false
              }
            ]

            规划要求：
            - 每个任务应该足够原子化，能通过一次工具调用完成
            - 用dependencies字段标明任务依赖关系，被依赖的任务必须先执行
            - 如果某个步骤过于复杂需要进一步拆解，将needsDecomposition设为true
            - 识别可以并行执行的任务，它们的dependencies不应相互包含

            可用工具列表：
            - database_query: 查询数据库
            - api_call: 调用外部API
            - data_analysis: 数据统计分析
            - report_generator: 生成报告文档
            """, goal);
    }

```java
    private List<Task> parsePlanFromResponse(String response) {
        // 解析JSON响应转换为Task对象列表
        // 实际项目中使用Jackson或Gson等JSON库
        List<Task> tasks = new ArrayList<>();
        // JSON解析逻辑...
        return tasks;
    }
}
```

这个Prompt里特意加了几个约束条件。第一个是要求任务原子化，这样执行器处理起来简单；第二个是明确依赖关系的表达方式；第三个是给模型一个递归分解的提示。实际项目中还会在Prompt里加上可用工具的列表和每个工具的功能说明，让模型知道可以调用哪些能力。

然后是Executor的实现，这部分要体现对并发执行和依赖管理的理解：

```java
public class Executor {
    private Map<String, Tool> toolRegistry;
    private Map<String, Object> taskResults;
    private ExecutorService executorService;

    public Executor() {
        this.toolRegistry = new HashMap<>();
        this.taskResults = new ConcurrentHashMap<>();
        this.executorService = Executors.newFixedThreadPool(10);
    }

    public void execute(List<Task> tasks) {
        Map<String, Task> taskMap = tasks.stream()
            .collect(Collectors.toMap(Task::getId, t -> t));

        // 按拓扑排序确定执行顺序
        List<Task> sortedTasks = topologicalSort(tasks);

        for (Task task : sortedTasks) {
            executeTask(task, taskMap);
        }

        executorService.shutdown();
    }

    private void executeTask(Task task, Map<String, Task> taskMap) {
        // 等待依赖任务完成
        for (String depId : task.getDependencies()) {
            Task depTask = taskMap.get(depId);
            waitForTaskCompletion(depTask);
        }

        task.setStatus(TaskStatus.RUNNING);

        try {
            // 准备参数：可能需要引用前置任务的结果
            Map<String, Object> params = resolveParameters(
                task.getParameters(), taskResults
            );

            // 执行任务
            Tool tool = toolRegistry.get(task.getToolName());
            Object result = tool.execute(params);

            taskResults.put(task.getId(), result);
            task.setStatus(TaskStatus.COMPLETED);

        } catch (Exception e) {
            task.setStatus(TaskStatus.FAILED);
            handleTaskFailure(task, e);
        }
    }

    private List<Task> topologicalSort(List<Task> tasks) {
        // Kahn算法实现拓扑排序
        Map<String, Integer> inDegree = new HashMap<>();
        Map<String, List<String>> graph = new HashMap<>();

        // 初始化入度和邻接表
        for (Task task : tasks) {
            inDegree.put(task.getId(), task.getDependencies().size());
            for (String dep : task.getDependencies()) {
                graph.computeIfAbsent(dep, k -> new ArrayList<>()).add(task.getId());
            }
        }

        // 找出入度为0的任务
        Queue<Task> queue = new LinkedList<>();
        Map<String, Task> taskMap = tasks.stream()
            .collect(Collectors.toMap(Task::getId, t -> t));

        for (Task task : tasks) {
            if (inDegree.get(task.getId()) == 0) {
                queue.offer(task);
            }
        }

        List<Task> result = new ArrayList<>();
        while (!queue.isEmpty()) {
            Task current = queue.poll();
            result.add(current);

            // 更新后续任务的入度
            if (graph.containsKey(current.getId())) {
                for (String nextId : graph.get(current.getId())) {
                    int degree = inDegree.get(nextId) - 1;
                    inDegree.put(nextId, degree);
                    if (degree == 0) {
                        queue.offer(taskMap.get(nextId));
                    }
                }
            }
        }

        return result;
    }

    private Map<String, Object> resolveParameters(
            Map<String, Object> params,
            Map<String, Object> results) {
        Map<String, Object> resolved = new HashMap<>();
        for (Map.Entry<String, Object> entry : params.entrySet()) {
            Object value = entry.getValue();
            // 如果参数值是引用前置任务结果的表达式，如"${task_1.output}"
            if (value instanceof String && ((String) value).startsWith("${")) {
                String ref = ((String) value).substring(2, ((String) value).length() - 1);
                value = results.get(ref);
            }
            resolved.put(entry.getKey(), value);
        }
        return resolved;
    }

    private void waitForTaskCompletion(Task task) {
        while (task.getStatus() != TaskStatus.COMPLETED
               && task.getStatus() != TaskStatus.FAILED) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }

    private void handleTaskFailure(Task task, Exception e) {
        System.err.println("任务执行失败: " + task.getId() + ", 错误: " + e.getMessage());
        // 可以在这里实现重试逻辑或者触发重新规划
    }
}
```

拓扑排序的目的是找到一个执行顺序，保证每个任务执行时它的依赖都已经完成。实现上用经典的Kahn算法，维护每个任务的入度，每次选择入度为0的任务执行，执行完后更新后续任务的入度。

现在把整个流程串起来：

```java
public class PlanExecuteAgent {
    private Planner planner;
    private Executor executor;

    public PlanExecuteAgent() {
        this.planner = new Planner();
        this.executor = new Executor();
    }

    public String handle(String userGoal) {
        System.out.println("接收到目标: " + userGoal);

        // 第一阶段：规划
        List<Task> plan = planner.generatePlan(userGoal);

        // 展示计划给用户确认
        System.out.println("\n生成的执行计划：");
        for (Task task : plan) {
            System.out.println("- [" + task.getId() + "] " + task.getDescription());
            if (!task.getDependencies().isEmpty()) {
                System.out.println("  依赖: " + String.join(", ", task.getDependencies()));
            }
        }

        // 第二阶段：执行
        System.out.println("\n开始执行任务...");
        executor.execute(plan);

        // 汇总结果
        return summarizeResults(executor.getTaskResults());
    }

    private String summarizeResults(Map<String, Object> taskResults) {
        StringBuilder summary = new StringBuilder("\n执行完成，结果汇总：\n");
        for (Map.Entry<String, Object> entry : taskResults.entrySet()) {
            summary.append("- ").append(entry.getKey())
                   .append(": ").append(entry.getValue()).append("\n");
        }
        return summary.toString();
    }
}
```

注意在规划和执行之间加了一个展示计划的步骤，这在实际应用中很重要。用户能看到系统打算怎么做，既提升了可解释性，也给了用户干预的机会。如果计划不合理，用户可以及时叫停，避免浪费执行成本。

对于分层规划的实现，可以通过递归逻辑来处理：

```java
public class HierarchicalPlanner extends Planner {
    private static final int MAX_DEPTH = 3;

    @Override
    public List<Task> generatePlan(String userGoal) {
        return generatePlanRecursive(userGoal, 0);
    }

    private List<Task> generatePlanRecursive(String goal, int depth) {
        List<Task> tasks = super.generatePlan(goal);

        if (depth >= MAX_DEPTH) {
            return tasks;  // 达到最大深度，停止递归
        }

        List<Task> expandedTasks = new ArrayList<>();
        for (Task task : tasks) {
            if (task.needsDecomposition()) {
                // 复杂任务继续分解
                System.out.println("分解任务: " + task.getDescription() + " (深度: " + (depth + 1) + ")");
                List<Task> subTasks = generatePlanRecursive(
                    task.getDescription(), depth + 1
                );
                // 更新子任务的依赖关系
                updateSubTaskDependencies(task, subTasks);
                expandedTasks.addAll(subTasks);
            } else {
                expandedTasks.add(task);
            }
        }
        return expandedTasks;
    }

    private void updateSubTaskDependencies(Task parentTask, List<Task> subTasks) {
        // 如果父任务有依赖，第一个子任务继承这些依赖
        if (!subTasks.isEmpty() && !parentTask.getDependencies().isEmpty()) {
            Task firstSubTask = subTasks.get(0);
            firstSubTask.getDependencies().addAll(parentTask.getDependencies());
        }
    }

    private boolean isComplex(Task task) {
        // 根据任务描述长度、涉及的操作数量等判断复杂度
        return task.getDescription().length() > 100
               || task.getDescription().contains("并且")
               || task.getDescription().contains("然后");
    }
}
```

这段代码展示了递归分解的核心思路。isComplex方法可以根据任务描述的长度、涉及的工具数量等指标来判断。实际项目中更好的做法是让模型自己标注哪些任务需要进一步分解，在Task类里加个needsDecomposition字段，这比写规则更灵活。

分解粒度其实是个trade-off。太粗的话每个任务内部逻辑复杂，执行时缺乏灵活性；太细的话任务数量爆炸，规划和调度的开销就上来了。通常用目标树的方式来控制，设定一个最大深度比如3层，叶子节点的任务要能在一次工具调用内完成。同时在规划时评估每个任务的复杂度，如果某个任务预估需要超过5个原子操作，那就触发递归分解。

执行过程中必须要有监控机制，不能规划完就不管了。通常会在每个任务执行完后加个检查点，验证输出结果是否符合预期。比如某个数据拉取任务，执行完要检查返回的数据量是否在合理范围、字段是否完整。如果发现异常，有两种处理策略：一是局部重试，给这个任务再执行一次的机会；二是触发重新规划，把异常信息反馈给Planner，让它调整后续步骤。假设在生成分析报告时，发现某个数据源不可用，Planner可以决定是改用备用数据源，还是跳过这部分数据只用其他维度的信息，或者直接告诉用户这份报告无法完成。这种决策能力是这个模式的核心价值。

常见的坑需要特别注意。第一个坑是规划过度细化，有些同学在设计Planner的Prompt时，让模型把所有细节都规划出来，结果生成了几十上百个任务，不仅Token消耗巨大，执行时稍有偏差整个计划就失效了。正确的做法是只规划到关键步骤，把细节的灵活性留给执行层。第二个坑是执行偏差没有容错机制，现实中工具调用经常会有超时、数据格式变化这些情况，如果没有异常处理和回退策略，整个任务链就会卡死。建议是每个任务都要定义超时时间和最大重试次数，关键任务还要准备Plan B的执行路径。第三个是成本控制问题，Plan-and-Execute虽然比ReAct省Token，但如果规划和重规划过于频繁，成本也会失控。实践中要设定规划预算，比如单次任务最多允许重新规划3次，超过就强制终止或降级到人工介入。

深度思考与演进方向
Plan-and-Execute模式和人类处理复杂任务的方式确实有相似之处。人类在处理复杂任务时也会先在脑子里构建一个大致的执行框架，比如写论文我们会先列提纲，装修房子会先做整体规划。这个模式本质上是在模拟人类的双系统认知——System 2负责深思熟虑的规划，System 1负责快速的执行响应。区别在于人类的规划和执行是高度交织的，可以随时调整；而当前的AI系统在这种动态平衡上还有很大改进空间。

如果要把这个模式部署到分布式环境，最大的挑战是状态一致性和任务调度。Planner生成的任务依赖图需要在多个执行节点之间共享，可以考虑用Redis这类中心化状态存储来维护任务状态，避免分布式锁的复杂性。任务调度可以借鉴消息队列的思路，把每个任务包装成消息推到队列里，执行节点从队列拉取任务执行，这样既能负载均衡又便于横向扩展。关键是要设计好失败恢复机制，某个节点挂了，它未完成的任务要能被其他节点接管。像Temporal这类工作流引擎就是专门解决这类问题的，虽然它不是专为AI Agent设计，但很多思想可以借鉴。

Plan-and-Execute并不是银弹，它在可预测性和灵活性之间做了明确的取舍。当面对一个新任务时，需要先评估两个维度——任务的确定性和执行成本。如果步骤逻辑非常清晰，比如数据ETL流程、定期报表生成这类，Plan-and-Execute的价值就很明显。但如果是探索性任务，需要根据中间结果频繁调整方向，比如研发新算法、问题根因分析，强行用这个模式反而会增加复杂度，这时候ReAct或者人机协作可能更合适。

最近多Agent协作成了热点，不同Agent各自负责不同的子任务，通过规划层的统一调度来完成复杂目标。这其实是Plan-and-Execute在组织架构上的延伸——不是一个Agent内部分规划和执行两个阶段，而是有专门的Planner Agent和多个Executor Agent。这种设计的好处是每个Agent可以更专注，Executor Agent甚至可以是领域专用模型，既提升了效果又降低了成本。当然挑战也很明显，Agent之间的通信协议、冲突解决机制、整体性能监控都需要精心设计。

Plan-and-Execute模式背后体现的其实是一种更通用的工程哲学——通过分治和抽象来管理复杂度。这个思想在软件工程的各个层面都有体现，从微服务的服务拆分到前端的组件化设计。AI Agent领域的特殊之处在于，"规划"这个环节本身就需要智能，不是简单的规则引擎能搞定的。随着大模型推理能力的提升，未来的方向可能是更动态的规划-执行-反思循环，Planner不再是一次性输出完整计划，而是持续根据执行反馈优化后续路径，这样能兼具效率和适应性。这种演进既保留了提前规划带来的成本优势，又融入了动态调整的灵活性，是真正能在复杂场景中落地的技术方向。
