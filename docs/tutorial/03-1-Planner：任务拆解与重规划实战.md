# 03-1｜Planner：任务拆解与重规划实战

## 1. 目标：把“需求”变成“可执行计划”

输入（用户需求）通常是自然语言，Planner 要输出结构化计划：

- 步骤列表（steps）
- 依赖关系（depends_on）
- 完成标准（done_when）
- 风险与兜底（fallback）

## 2. 数据结构建议

```json
{
  "goal": "完成技术选型报告",
  "constraints": {"deadline": "2d", "must_cite": true},
  "steps": [
    {"id": "s1", "task": "检索资料", "done_when": ">=10条可信来源"},
    {"id": "s2", "task": "提取对比维度", "depends_on": ["s1"]},
    {"id": "s3", "task": "输出结论与风险", "depends_on": ["s2"]}
  ]
}
```

## 3. 拆解算法（教学版）

1. 识别目标类型（调研/问答/执行/生成）。
2. 提取约束（时间、格式、证据要求）。
3. 拆成“可验证”的最小步骤。
4. 给每步写 done_when。
5. 检查是否存在循环依赖。

## 4. 重规划（Replan）触发条件

- 连续重试失败。
- 证据不足（引用为空或质量低）。
- 外部工具不可用。

伪代码：

```python
if step.failed_count >= 2 or evidence_score < 0.6:
    patch = planner.replan(task_ctx, failed_step=step)
    dag.merge(patch)
```

## 5. 常见错误

- 步骤太大：无法定位失败。
- done_when 太模糊：无法判定完成。
- 所有步骤都串行：时延高。

## 6. 练习

给需求“输出竞品分析 + 2 周计划”，写一个 5 步 DAG 计划，并明确每步 done_when。
