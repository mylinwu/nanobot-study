# 06-1｜Prompt 模板工程与版本管理

## 1. 为什么要模板化

避免“改一处，坏一片”。

## 2. 推荐模板结构

```text
[System Rules]
[Task Goal]
[Step Instruction]
[Evidence]
[Output Schema]
```

## 3. 版本管理

- `prompt_id`
- `version`
- `change_note`
- `rollback_to`

## 4. 实践建议

- 小步修改，每次只改一个变量。
- 与评测集绑定，改完立刻回归。

## 5. 练习

把“输出 Markdown 表格”要求改成 schema 约束，比较稳定性差异。
