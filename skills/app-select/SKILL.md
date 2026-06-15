# AppSelect 重构 Skill：Primitive Value Select

## 目标

用于重构基于 Element Plus el-select 的二次封装组件。本文只固定架构决策和执行边界，减少执行模型自行设计，降低 token 消耗。

## 适用场景

- 成组 select。
- 每个 select 的 options 可能不同。
- options 异步加载。
- 详情回显。
- 需要读取 option 元数据。
- form 字段可能是 primitive、primitive 数组，或业务对象数组。

## 核心决策

### 1. AppSelect 只绑定 primitive

AppSelect 是基础输入组件，不是对象状态管理组件。

允许：

- 单选：string | number | boolean | null。
- 多选：Array<string | number | boolean>。

禁止：

- v-model 绑定对象。
- v-model 绑定对象数组。
- return-object。
- 在组件内部做对象深比较。
- 在组件内部管理业务 optionsMap。
- 在组件内部请求业务接口。

## 组件契约

### Props

必须保留或补齐以下能力：

- modelValue：primitive、primitive array 或 null。
- options：option 数组，option 可以携带任意元数据。
- labelKey：默认 label。
- valueKey：默认 value。
- disabledKey：默认 disabled。
- multiple。
- clearable。
- filterable。
- loading。
- placeholder。

### Emits

至少支持：

- update:modelValue：只返回 primitive 或 primitive array。
- change：只返回 primitive 或 primitive array。
- change-option：返回当前选中的完整 option；多选返回 option 数组。
- clear。
- visible-change。

## 元数据规则

元数据放在 options 里，不放进 v-model。

需要元数据时使用两种方式之一：

1. 监听 change-option。
2. 在业务层通过 valueKey + value 反查 option。

不要为了拿 label、code、deptId 等字段而改成对象绑定。

## 成组 select 规则

成组 select 的 options 由业务层维护，不放进 AppSelect。

推荐业务层结构：

- rows：表单行数据，只保存 value 和必要业务字段。
- optionsMap：按 optionKey 缓存 options。
- loadingMap：按 optionKey 缓存 loading 状态。
- getOptionKey(row)：由业务决定当前行使用哪份 options。

optionKey 可以是：

- 类型 key，例如 user、department、city。
- 行 id，例如 row.id。
- 依赖组合 key，例如 city:provinceId。

## 回显规则

回显只有两个合法方案：

1. 初始化时加载对应 options。
2. 详情接口返回 value + label，业务层注入 echo option。

如果只返回 value，且 options 未加载，AppSelect 不负责猜 label。

## form 字段是对象数组时

AppSelect 仍然不能直接绑定对象数组。

使用业务适配层：

- 页面内少量使用：用 computed 在对象数组和 primitive 数组之间转换。
- 多页面复用：封装业务适配组件，例如 UserObjectArraySelect，内部使用 AppSelect。

适配规则：

- AppSelect 绑定 selectedIds。
- selectedIds 的 get 从 form.items 提取 id 数组。
- selectedIds 的 set 根据 ids 重建 form.items。
- 重建对象时优先使用当前 options 元数据；options 缺失时保留旧对象中的元数据。

不要把这个复杂度放进 AppSelect。

## 重构执行步骤

执行模型按顺序做：

1. 找到现有 select 封装组件。
2. 判断是否存在对象绑定、return-object、对象比较逻辑。
3. 删除或废弃对象绑定能力。
4. 统一 modelValue 为 primitive、primitive array 或 null。
5. 保留 labelKey、valueKey、disabledKey。
6. el-option 的 value 只能取 item[valueKey]。
7. 增加或修正 change-option。
8. 检查调用处，把对象 v-model 改成 id 或 ids。
9. 调用处需要元数据时，改用 change-option 或业务层反查。
10. 对象数组 form 字段使用 computed 或业务适配组件。
11. 回显问题在业务层通过 options 预加载或 echo option 解决。
12. 保持最小改动，不做无关重构。

## 验收清单

- 单选 primitive v-model 正常。
- 多选 primitive array v-model 正常。
- 对象 v-model 被禁止或报错。
- 对象数组不直接传给 AppSelect。
- change 返回 primitive。
- change-option 返回完整 option。
- 异步 options 加载后可回显。
- echo option 注入后可回显。
- 组件内部没有业务接口。
- 组件内部没有 optionsMap。
- 组件内部没有对象深比较。

## 最终原则

AppSelect 只负责选择 primitive value。

options 负责展示和元数据。

业务层负责 options 来源、回显、联动、元数据提取、对象数组适配和提交组装。
