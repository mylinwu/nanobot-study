# 02｜agents 运行逻辑与流程图

这一章是全书最关键的一章：你会看到一个任务从进入系统到输出结果的完整链路。

## 1. 高层流程（宏观）

```mermaid
flowchart TD
    A[用户输入目标] --> B[任务解析与约束提取]
    B --> C[Planner 生成执行计划]
    C --> D[Orchestrator 调度执行]
    D --> E[Executor 调用工具/模型]
    E --> F[结果评估与反思]
    F -->|通过| G[汇总与结构化输出]
    F -->|不通过| C
```

### 通俗解释

- Planner 像“项目经理”：决定先做什么后做什么。
- Orchestrator 像“调度中枢”：盯住流程和状态。
- Executor 像“执行团队”：真的去搜、算、查、写。
- Reflection 像“质检员”：检查结果是否达标。

---

## 2. 细粒度时序（微观）

```mermaid
sequenceDiagram
    participant U as User
    participant O as Orchestrator
    participant P as Planner
    participant M as Memory/Retriever
    participant E as Executor
    participant T as Tools
    participant R as Reflector

    U->>O: 提交任务
    O->>P: 生成计划(目标/约束/上下文)
    P-->>O: 返回步骤列表
    loop 每个步骤
        O->>M: 读取历史记忆/知识
        O->>E: 下发当前步骤
        E->>T: 调用工具(API/搜索/DB/代码)
        T-->>E: 工具结果
        E-->>O: 步骤产出
        O->>R: 质量评估(完整性/正确性/格式)
        R-->>O: 通过或修正建议
    end
    O-->>U: 最终结果
```

---

## 3. 关键状态机（执行生命周期）

常见状态：

- `PENDING`：待执行
- `RUNNING`：执行中
- `SUCCESS`：成功
- `FAILED`：失败
- `RETRYING`：重试中
- `BLOCKED`：阻塞（例如缺少凭证）

为什么要状态机？

- 便于恢复（断点续跑）
- 便于监控（知道卡在哪）
- 便于重试（仅重跑失败步骤）

---

## 4. 三条必须理解的“流”

1. **控制流**：谁决定下一步做什么。
2. **数据流**：每一步输入输出怎么传递。
3. **反馈流**：失败如何回到计划层修正。

只要你把这三条流看清楚，整个 Agent 系统就不再神秘。

