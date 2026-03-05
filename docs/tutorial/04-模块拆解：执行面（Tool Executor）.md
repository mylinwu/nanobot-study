# 04｜模块拆解：执行面（Tool / Executor）

## 0. 先定义执行面的职责边界

执行面只做三件事：

1. 把“step 目标”翻译为“可执行动作”。
2. 调用工具或模型拿结果。
3. 对结果做校验、归一化并返回控制面。

---

## 1. 渐进式推演：Executor

### V1：纯 LLM 执行器

```python
def execute(step):
    return llm.generate(step.task)
```

**问题**：

- 无实时信息。
- 无法可靠计算。
- 结果不可审计。

---

### V2：单工具执行器（搜索）

**需求升级**：至少让系统能查到外部事实。

```python
if step.type == "search":
    docs = search_api(step.query)
    return summarize(docs)
```

**问题**：

- 工具分支写死，扩展差。
- 异常处理混乱。

---

### V3：工具注册中心 + 统一 schema

**升级点**：把“工具能力”抽象成可注册单元。

```python
TOOL_REGISTRY = {
  "search": search_tool,
  "sql": sql_tool,
  "python": py_tool
}

result = TOOL_REGISTRY[name].invoke(args)
```

为每个工具定义：

- `input_schema`
- `timeout`
- `retry_policy`
- `output_schema`

**收益**：可插拔、可治理。

---

### V4：可靠执行器（容错+审计）

新增能力：

- 熔断（某工具失败率过高时临时禁用）
- 降级（主工具失败切备选）
- 审计（记录参数、耗时、结果摘要）

```python
try:
    result = invoke_with_timeout(tool, args)
except TimeoutError:
    result = fallback_tool(args)
finally:
    audit_log(task_id, step_id, tool, cost, latency)
```

---

## 2. 工具调用标准流水线（必须落地）

1. Tool Selection：为什么选这个工具。
2. Argument Building：参数是否齐全、类型是否正确。
3. Invocation：执行与超时控制。
4. Validation：结果是否满足 schema。
5. Normalization：统一字段返回上游。

---

## 3. 典型失败案例（学习重点）

**案例**：SQL 工具返回空结果。

- 错误做法：直接把“空”当事实继续生成。
- 正确做法：
  1) 判断是“确实无数据”还是“查询条件错”；
  2) 触发 query rewrite 重试；
  3) 仍失败则上报 `BLOCKED(reason=data_unavailable)`。
---

## 延伸阅读（本章子文档）

- [04-1-Tool 接口规范与注册中心设计.md](./04-1-Tool 接口规范与注册中心设计.md)
- [04-2-Executor：错误处理、降级与审计.md](./04-2-Executor：错误处理、降级与审计.md)
- [04-3-工具安全与权限：最小授权到审计闭环.md](./04-3-工具安全与权限：最小授权到审计闭环.md)
- [04-4-工具性能优化：批处理、缓存与并发控制.md](./04-4-工具性能优化：批处理、缓存与并发控制.md)
