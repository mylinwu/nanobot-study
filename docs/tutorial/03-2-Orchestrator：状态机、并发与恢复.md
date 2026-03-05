# 03-2｜Orchestrator：状态机、并发与恢复

## 1. 状态机为什么是核心

没有状态机，任务失败后很难恢复；有状态机才能实现：

- 精确重试
- 断点续跑
- 失败审计

建议状态：`PENDING/RUNNING/SUCCESS/FAILED/RETRYING/BLOCKED/CANCELLED`。

## 2. 调度策略

### 串行
简单稳定，适合强依赖链路。

### 并行
依赖满足后即可并行，适合资料采集、批量分析。

### 混合
主链路串行，支线并行。

## 3. 幂等与去重

每个 step 执行要带 `idempotency_key`，防止重复写入。

```python
key = f"{task_id}:{step_id}:{attempt_group}"
if not store.exists(key):
    out = run_step()
    store.save(key, out)
```

## 4. 断点恢复

最小实现：

- 每个步骤完成后写 checkpoint。
- 重启后读取最后成功 step 继续执行。

## 5. 故障案例

案例：worker 重启导致同一步重复执行。

解决：

1. 使用幂等键。
2. 输出写入前先查重。
3. audit log 标记重复请求来源。

## 6. 练习

设计一个“最多重试 2 次 + 指数退避 + 可取消”的调度伪代码。
