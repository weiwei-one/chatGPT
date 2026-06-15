# AppSelect Primitive Value Skill

## 适用场景

当项目中存在对 Element Plus `el-select` 的二次封装，并且业务中有以下需求时，使用本 Skill：

- 成组的 `el-select`。
- 每个 `el-select` 的 options 可能不同。
- 支持异步 options。
- 支持详情回显。
- 需要从选中的 option 中拿元数据。
- 不希望 `v-model` 绑定整个对象。
- 希望组件统一、稳定、易维护。

## 核心目标

重构后的选择器组件必须满足：

1. `v-model` 只绑定 primitive value。
2. 不支持对象作为 `v-model`。
3. 元数据从 options 中获取，不塞进 `v-model`。
4. 通过事件向外暴露完整 option。
5. 支持单选和多选。
6. 支持回显。
7. 支持每个 select 使用不同 options。
8. 组件内部不持有业务状态。
9. 业务层负责 optionsMap、回显注入、异步加载。

## 严格禁止

不要做以下事情：

- 不要让 `v-model` 绑定对象。
- 不要支持 `return-object`。
- 不要把完整 option 写入表单字段。
- 不要在组件内部请求业务接口。
- 不要把 options 写入 form row 中。
- 不要让组件自己决定 optionKey。
- 不要在组件内部处理业务联动。
- 不要默认深拷贝 options。
- 不要用 JSON.stringify 比较 option。
- 不要把 label 当作提交字段，除非业务层显式提取。

## 数据模型约定

### PrimitiveValue

```ts
type PrimitiveValue = string | number | boolean
```

### 单选 v-model

```ts
type SingleSelectValue = PrimitiveValue | null
```

### 多选 v-model

```ts
type MultipleSelectValue = PrimitiveValue[]
```

### 统一 modelValue

```ts
type SelectModelValue = SingleSelectValue | MultipleSelectValue
```

### Option

```ts
type SelectOption = Record<string, any>
```

option 可以携带任意元数据，例如：

```ts
const userOptions = [
  {
    id: 1001,
    name: '张三',
    deptId: 10,
    deptName: '研发部',
    disabled: false
  }
]
```

但 `v-model` 只能绑定：

```ts
form.userId = 1001
```

不能绑定：

```ts
form.user = {
  id: 1001,
  name: '张三'
}
```

## 组件 API 设计

组件名称建议：

```ts
AppSelect
```

### Props

```ts
type PrimitiveValue = string | number | boolean
type SelectModelValue = PrimitiveValue | PrimitiveValue[] | null
type SelectOption = Record<string, any>

interface AppSelectProps {
  modelValue: SelectModelValue
  options: SelectOption[]

  labelKey?: string
  valueKey?: string
  disabledKey?: string

  multiple?: boolean
  clearable?: boolean
  filterable?: boolean
  placeholder?: string
  disabled?: boolean
  loading?: boolean
}
```

### Emits

```ts
const emit = defineEmits<{
  'update:modelValue': [value: SelectModelValue]
  'change': [value: SelectModelValue]
  'change-option': [option: SelectOption | SelectOption[] | undefined]
  'clear': []
  'visible-change': [visible: boolean]
}>()
```

### 事件语义

| 事件 | 返回内容 | 用途 |
|---|---|---|
| `update:modelValue` | primitive value 或 primitive array | 表单绑定 |
| `change` | primitive value 或 primitive array | 常规监听 |
| `change-option` | 完整 option 或 option array | 获取元数据 |
| `clear` | 无 | 清空监听 |
| `visible-change` | boolean | 展开时加载 options |

## 标准组件实现

创建或重构组件：

```vue
<script setup lang="ts">
import { computed } from 'vue'

type PrimitiveValue = string | number | boolean
type SelectModelValue = PrimitiveValue | PrimitiveValue[] | null
type SelectOption = Record<string, any>

const props = withDefaults(defineProps<{
  modelValue: SelectModelValue
  options: SelectOption[]
  labelKey?: string
  valueKey?: string
  disabledKey?: string
  multiple?: boolean
  clearable?: boolean
  filterable?: boolean
  placeholder?: string
  disabled?: boolean
  loading?: boolean
}>(), {
  labelKey: 'label',
  valueKey: 'value',
  disabledKey: 'disabled',
  multiple: false,
  clearable: true,
  filterable: false,
  placeholder: '请选择',
  disabled: false,
  loading: false
})

const emit = defineEmits<{
  'update:modelValue': [value: SelectModelValue]
  'change': [value: SelectModelValue]
  'change-option': [option: SelectOption | SelectOption[] | undefined]
  'clear': []
  'visible-change': [visible: boolean]
}>()

function isPrimitiveValue(value: unknown): value is PrimitiveValue {
  return ['string', 'number', 'boolean'].includes(typeof value)
}

function getOptionValue(option: SelectOption): PrimitiveValue {
  const value = option?.[props.valueKey]

  if (!isPrimitiveValue(value)) {
    throw new Error(`[AppSelect] option.${props.valueKey} must be string | number | boolean`)
  }

  return value
}

function getOptionLabel(option: SelectOption): string {
  const label = option?.[props.labelKey]
  return label == null ? '' : String(label)
}

function getOptionDisabled(option: SelectOption): boolean {
  return Boolean(option?.[props.disabledKey])
}

function findOptionByValue(value: PrimitiveValue): SelectOption | undefined {
  return props.options.find(option => getOptionValue(option) === value)
}

function findOptionsByValues(values: PrimitiveValue[]): SelectOption[] {
  const valueSet = new Set(values)
  return props.options.filter(option => valueSet.has(getOptionValue(option)))
}

function validateModelValue(value: SelectModelValue) {
  if (value == null) return

  if (Array.isArray(value)) {
    const invalid = value.some(item => !isPrimitiveValue(item))
    if (invalid) {
      throw new Error('[AppSelect] multiple modelValue only supports primitive array')
    }
    return
  }

  if (!isPrimitiveValue(value)) {
    throw new Error('[AppSelect] modelValue only supports string | number | boolean | null')
  }
}

function getSelectedOption(value: SelectModelValue): SelectOption | SelectOption[] | undefined {
  if (props.multiple) {
    return Array.isArray(value) ? findOptionsByValues(value) : []
  }

  if (value == null || Array.isArray(value)) return undefined
  return findOptionByValue(value)
}

const innerValue = computed<SelectModelValue>({
  get() {
    validateModelValue(props.modelValue)
    return props.modelValue
  },
  set(value) {
    validateModelValue(value)

    emit('update:modelValue', value)
    emit('change', value)
    emit('change-option', getSelectedOption(value))
  }
})

function handleClear() {
  emit('clear')
  emit('change-option', props.multiple ? [] : undefined)
}

function handleVisibleChange(visible: boolean) {
  emit('visible-change', visible)
}
</script>

<template>
  <el-select
    v-model="innerValue"
    :multiple="multiple"
    :clearable="clearable"
    :filterable="filterable"
    :placeholder="placeholder"
    :disabled="disabled"
    :loading="loading"
    @clear="handleClear"
    @visible-change="handleVisibleChange"
  >
    <el-option
      v-for="item in options"
      :key="getOptionValue(item)"
      :label="getOptionLabel(item)"
      :value="getOptionValue(item)"
      :disabled="getOptionDisabled(item)"
    />
  </el-select>
</template>
```

## 业务层 optionsMap 设计

组件本身不管理成组 select 的 options。业务层使用 `optionsMap` 管理。

```ts
type PrimitiveValue = string | number | boolean

type SelectRow = {
  id: string
  optionKey: string
  value: PrimitiveValue | null
  valueLabel?: string
  valueCode?: string
}

type SelectOption = Record<string, any>

const rows = ref<SelectRow[]>([])
const optionsMap = reactive<Record<string, SelectOption[]>>({})
const loadingMap = reactive<Record<string, boolean>>({})
```

### optionKey 规则

根据业务选择一种：

```ts
function getOptionKey(row: SelectRow) {
  return row.optionKey
}
```

如果 options 按类型复用：

```ts
row.optionKey = 'user'
row.optionKey = 'department'
row.optionKey = 'city'
```

如果每行 options 不同：

```ts
row.optionKey = row.id
```

如果 options 依赖其他字段：

```ts
row.optionKey = `city:${row.provinceId}`
```

## 业务层使用方式

```vue
<AppSelect
  v-model="row.value"
  :options="optionsMap[getOptionKey(row)] || []"
  :loading="loadingMap[getOptionKey(row)]"
  label-key="name"
  value-key="id"
  @visible-change="visible => visible && ensureOptions(row)"
  @change-option="option => handleChangeOption(row, option)"
/>
```

## 异步加载 options

```ts
async function ensureOptions(row: SelectRow) {
  const key = getOptionKey(row)

  if (optionsMap[key]?.length) return

  loadingMap[key] = true

  try {
    const list = await fetchOptionsByRow(row)
    optionsMap[key] = list
  } finally {
    loadingMap[key] = false
  }
}
```

## 回显策略

### 推荐后端详情返回 value + label

详情接口推荐返回：

```ts
{
  id: 'row_1',
  optionKey: 'user',
  value: 1001,
  valueLabel: '张三'
}
```

### 前端注入回显 option

```ts
function injectEchoOption(row: SelectRow) {
  if (row.value == null || !row.valueLabel) return

  const key = getOptionKey(row)

  if (!optionsMap[key]) {
    optionsMap[key] = []
  }

  const exists = optionsMap[key].some(option => option.id === row.value)

  if (!exists) {
    optionsMap[key].unshift({
      id: row.value,
      name: row.valueLabel,
      code: row.valueCode
    })
  }
}
```

初始化：

```ts
async function init() {
  const detail = await fetchDetail()

  rows.value = detail.rows.map((item: any) => ({
    id: item.id,
    optionKey: item.optionKey,
    value: item.value,
    valueLabel: item.valueLabel,
    valueCode: item.valueCode
  }))

  rows.value.forEach(injectEchoOption)
}
```

## 元数据处理

不要把完整 option 写入 `row.value`。

如需元数据，用 `change-option`：

```ts
function handleChangeOption(row: SelectRow, option: any) {
  if (!option || Array.isArray(option)) return

  row.valueLabel = option.name
  row.valueCode = option.code
}
```

如果提交时需要元数据，在提交时组装：

```ts
function submit() {
  return rows.value.map(row => ({
    id: row.id,
    value: row.value,
    label: row.valueLabel,
    code: row.valueCode
  }))
}
```

## 多选规则

多选时：

```ts
form.roleIds = [1, 2, 3]
```

不要：

```ts
form.roles = [
  { id: 1, name: '管理员' },
  { id: 2, name: '编辑' }
]
```

使用：

```vue
<AppSelect
  v-model="form.roleIds"
  multiple
  :options="roleOptions"
  label-key="name"
  value-key="id"
  @change-option="handleRoleOptions"
/>
```

```ts
function handleRoleOptions(options: any[]) {
  form.roleNames = options.map(item => item.name)
}
```

## 重构步骤

按以下顺序执行，不要跳步：

1. 找到当前封装的 select 组件。
2. 检查是否存在对象绑定逻辑。
3. 删除 `returnObject`、对象比较、对象回显相关代码。
4. 统一 `modelValue` 类型为 primitive 或 primitive array。
5. 增加 `labelKey`、`valueKey`、`disabledKey`。
6. 统一 `el-option` 的 `value` 为 `item[valueKey]`。
7. 增加 `change-option` 事件。
8. 增加 modelValue 校验，禁止 object。
9. 检查所有调用处，把对象 v-model 改成 id v-model。
10. 如果调用处需要元数据，改用 `@change-option`。
11. 如果需要回显，业务层通过 options 预加载或注入回显 option。
12. 删除 form 中冗余的完整对象字段。
13. 提交逻辑只提交 primitive value，必要元数据在 submit 时组装。

## 验收清单

完成后必须满足：

- [ ] `v-model="form.userId"` 正常。
- [ ] `v-model="form.user"` 如果是对象，会抛错或 TypeScript 报错。
- [ ] 单选可以回显。
- [ ] 多选可以回显。
- [ ] options 异步加载后可以回显。
- [ ] options 为空但注入回显 option 后可以回显。
- [ ] `change` 返回 primitive value。
- [ ] `change-option` 返回完整 option。
- [ ] clear 后 modelValue 变成 `null` 或 `[]`。
- [ ] 组件内部没有业务接口请求。
- [ ] 组件内部没有 optionsMap。
- [ ] form 中没有完整 option 对象。

## 常见错误修复

### 错误：回显只显示 id，不显示 label

原因：options 中没有对应 value 的 option。

修复：

- 初始化时加载 options；或
- 后端返回 valueLabel，前端注入回显 option。

### 错误：change-option 返回 undefined

原因：当前 value 在 options 中找不到。

修复：

- 确认 `valueKey` 是否正确。
- 确认 value 类型是否一致，例如 `1` 和 `'1'` 不相等。
- 确认 options 是否已加载。

### 错误：多选回显失败

原因：modelValue 不是 primitive array。

修复：

```ts
form.roleIds = [1, 2, 3]
```

不要：

```ts
form.roleIds = [{ id: 1 }, { id: 2 }]
```

### 错误：接口需要 label/code

不要改成对象绑定。

在提交时组装：

```ts
const payload = {
  userId: form.userId,
  userName: selectedUser?.name,
  userCode: selectedUser?.code
}
```

## 最终原则

选择器组件只负责输入 primitive value。

options 负责展示和元数据。

业务层负责：

- options 来源；
- 回显注入；
- 异步加载；
- 联动逻辑；
- 元数据提取；
- 提交组装。

组件层禁止：

- 绑定对象；
- 管理业务 options；
- 处理业务联动；
- 保存业务元数据。
