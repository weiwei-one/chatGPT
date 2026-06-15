# AppSelect 重构 Skill：Primitive Value Select

## 目标

用于重构基于 Element Plus `el-select` 的二次封装组件。

本 Skill 只规定架构边界、组件契约和执行策略，不提供大段模板代码。执行模型应先阅读现有代码，再按本规则做最小改动。

---

## 适用场景

- 成组 select。
- 每个 select 的 options 可能不同。
- options 可能异步加载。
- 需要详情回显。
- 需要读取 option 元数据。
- 表单字段可能是 primitive、primitive 数组，或业务对象数组。

---

## 核心决策

### 1. 基础组件只处理 primitive value

`AppSelect` 是基础输入组件，不是对象状态管理组件。

允许的 `v-model`：

```ts
t