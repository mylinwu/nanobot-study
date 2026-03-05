# 04-2｜Executor：错误处理、降级与审计

## 1. 错误分级

- 可重试：超时、临时网络失败。
- 不可重试：参数非法、权限不足。
- 需人工介入：数据源缺失、账号封禁。

## 2. 降级策略

主工具失败 -> 备选工具 -> 纯模型兜底（并标记置信度下降）。

## 3. 执行框架伪代码

```python
try:
    out = invoke(primary_tool, args)
except RetryableError:
    out = retry_with_backoff(primary_tool, args)
except Exception:
    out = invoke(fallback_tool, args)
finally:
    audit(task_id, step_id, tool, latency, cost, status)
```

## 4. 审计日志关键字段

- task_id / step_id
- tool_name / args_digest
- latency_ms / token_cost
- status / error_type

## 5. 故障复盘样例

“SQL 空结果”不应直接生成结论，应进入 query rewrite 或返回 `BLOCKED(data_unavailable)`。

## 6. 练习

设计一个错误码表（10 个以内），覆盖重试、降级、阻塞三类场景。
