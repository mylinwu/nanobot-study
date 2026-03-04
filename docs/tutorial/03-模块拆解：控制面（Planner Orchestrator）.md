# 03｜模块拆解：控制面（Planner / Orchestrator）

> 本章目标：你不仅知道“Planner/Orchestrator 是什么”，还知道它们在工程里是如何一步步长出来的。

## 0. 模块要解决的真实需求

用户说：“帮我对比 5 个开源 Agent 框架，输出可落地选型建议，附风险和里程碑。”

这类任务天然包含：

- 多步骤：检索、筛选、对比、总结。
- 强约束：要有来源、要结构化、要可执行。
- 可追踪：出错时要知道错在哪一步。

如果没有控制面，执行会非常混乱。

---

## 1. 渐进式推演：Planner

### V1：单步 Planner（能跑就行）

**需求**：先把“目标”变成“一个可执行动作”。

**实现**：直接把用户输入封装成一个 step。

```python
plan = [{"id": "s1", "task": user_goal}]
```

**问题**：

- 没有拆分，复杂任务会失败。
- 无法定位“哪一段失败”。

---

### V2：多步线性 Planner（可解释）

**升级动机**：把复杂任务拆成 3~8 个可验证步骤。

**实现要点**：

- 每一步都有 `done_when`（完成标准）。
- 按固定顺序执行。

```json
{
  "steps": [
    {"id": "s1", "task": "检索候选框架", "done_when": "得到>=20条来源"},
    {"id": "s2", "task": "按维度打标签", "done_when": "每框架>=8个维度"},
    {"id": "s3", "task": "生成建议", "done_when": "输出含结论+风险+路线图"}
  ]
}
```

**收益**：流程可解释。

**新问题**：

- 缺少依赖表达（分支任务不友好）。
- 无法并行。

---

### V3：DAG Planner（可并行）

**升级动机**：让可独立步骤并行执行，降低总时延。

**实现要点**：

- step 支持 `depends_on`。
- Orchestrator 根据依赖拓扑排序。

```json
{
  "steps": [
    {"id": "s1", "task": "收集资料"},
    {"id": "s2", "task": "抽取性能数据", "depends_on": ["s1"]},
    {"id": "s3", "task": "抽取生态数据", "depends_on": ["s1"]},
    {"id": "s4", "task": "综合决策", "depends_on": ["s2", "s3"]}
  ]
}
```

**收益**：时延优化明显。

**新问题**：

- 失败后的重规划仍较粗糙。

---

### V4：自适应 Planner（可修正）

**升级动机**：执行中发现信息不足，要能回退并补步骤。

**实现要点**：

- 引入 `replan_trigger`（如：证据不足、工具连续失败）。
- 仅重规划失败子图，不重跑全链路。

**伪代码**：

```python
if step.failed and step.retry_exhausted:
    patch_plan = planner.replan(context, failed_step=step)
    dag.merge(patch_plan)
```

**收益**：稳定性大幅提升。

---

## 2. 渐进式推演：Orchestrator

### V1：串行调度器

```python
for step in plan:
    run(step)
```

- 好处：简单。
- 坏处：慢，且没有状态持久化。

### V2：带状态机与重试

每个 step 引入状态：`PENDING/RUNNING/SUCCESS/FAILED/RETRYING/BLOCKED`。

- 失败可重试 N 次。
- 支持超时与取消。

### V3：并行调度 + 幂等执行

- 可并行 step 放入 worker pool。
- 每步有 `idempotency_key`，避免重复写入。

### V4：可恢复编排（断点续跑）

- Orchestrator 周期性 checkpoint。
- 进程重启后按 task_id 恢复。

---

## 3. 控制面核心数据结构（建议）

```json
{
  "task_id": "t_001",
  "goal": "...",
  "plan_version": 3,
  "steps": [{"id": "s1", "status": "SUCCESS", "output_ref": "obj://..."}],
  "events": [{"ts": 1710000000, "type": "STEP_STARTED", "step_id": "s1"}],
  "metrics": {"latency_ms": 3200, "retry_count": 1}
}
```

---

## 4. 你在源码里应重点验证什么

1. Planner 输出是否可验证（不是纯文本描述）。
2. Orchestrator 是否有可恢复能力。
3. 重试策略是否有退避（避免雪崩）。
4. replan 是否只重算局部。

