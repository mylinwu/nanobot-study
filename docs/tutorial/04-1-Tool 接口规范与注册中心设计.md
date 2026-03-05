# 04-1｜Tool 接口规范与注册中心设计

## 1. 目标

让工具从“脚本调用”升级为“标准化能力”。

## 2. 工具协议（建议）

```json
{
  "name": "search",
  "input_schema": {"query": "string", "topk": "number"},
  "output_schema": {"hits": "array"},
  "timeout_ms": 8000,
  "retry": 1
}
```

## 3. 注册中心

```python
class ToolRegistry:
    def register(self, spec, handler): ...
    def get(self, name): ...
```

收益：

- 插拔方便
- 可统一审计
- 可统一限流

## 4. 参数构建与校验

- 参数缺失：直接失败并返回缺失字段。
- 参数类型错：自动修复一次，仍失败则上报。

## 5. 常见坑

- 工具输出不标准，导致上游解析崩溃。
- 将业务逻辑写进工具层，难复用。

## 6. 练习

为 `sql` 工具写一份 input/output schema，并列出 3 条校验规则。
