---
name: api-patterns
description: 项目 API 调用、异步错误处理、Store、Composable、国际化和类型定义的当前模式与新增代码建议
---

# API / Store / i18n 规范

## API 调用

API 文件通常放在 `src/api/`，按业务模块拆分。

```ts
import type { MyResponse } from '@/types/myModule'
import { request } from '@/utils/request'

enum URL {
  GET_LIST = '/my-module/list',
  CREATE = '/my-module/create',
}

export const getList = (data: ListParams) => {
  return request.post<MyResponse>({
    url: URL.GET_LIST,
    data,
  })
}

export const create = (data: CreateParams) => {
  return request.post<BaseResponse>({
    url: URL.CREATE,
    data,
  }, { isTransformResponse: false })
}
```

### 常见 request 选项

- `isTransformResponse: false`：返回完整响应体
- `ignoreError: true`：跳过统一错误提示
- `isReturnNativeResponse: true`：返回原生响应
- `loading: true`：显示全局 loading

## 异步错误处理

### 新增代码默认

新增代码默认使用 `await-to-js`：

```ts
import to from 'await-to-js'

const [err, res] = await to(getList(params))
if (err)
  return
tableData.value = res.list
```

### 历史兼容

仓库里也存在本地 helper [awaitto.ts](/D:/LS/AmazonAD_Vue3/src/utils/awaitto.ts)。如果当前修改的文件已经在使用：

```ts
import awaitTo, { filterNoneParams } from '@/utils/awaitto'
```

则局部修改时可以保持一致，不必为了统一而大范围改旧代码。

### 常见使用场景

```ts
import to from 'await-to-js'

const fetchData = async () => {
  loading.value = true
  const [err, res] = await to(getList({
    ...queryParams,
    page: paginationDetail.current,
    page_size: paginationDetail.pageSize,
  }))
  loading.value = false
  if (err)
    return
  tableData.value = res.list
  paginationDetail.total = res.total
}

const handleSubmit = async () => {
  const [err] = await to(create(formData))
  if (err)
    return
  MessagePlugin.success(t('common.success'))
  visible.value = false
  fetchData()
}
```

## 参数清洗

如果需要过滤空参数，项目已有：

```ts
import { filterFalsy } from '@/utils/webapp'
```

适合在拼接查询条件时使用。

## Store

### 当前仓库现状

- Store 定义风格并不完全统一
- 大多数业务 store 在 `src/store/modules/`
- 聚合访问可通过 `useStore()`

```ts
const { globalStore, deliveryStore } = useStore()
const userStore = useUserStore()
const { collapsed } = storeToRefs(globalStore)
```

### 新增代码建议

- 新增 store 优先参考相邻模块，而不是强行套单一模板
- 如果新建的是常规业务 store，优先沿用当前主流的 `defineStore('name', { ... })` 风格
- 如果修改旧 store，保持该文件原有模式

## Composable

- 全局 composable 放在 `src/composables/`
- 组件私有逻辑可放在组件目录内
- 需要返回 JSX / 渲染函数时使用 `.tsx`

```ts
export const useMyFeature = () => {
  const { t } = useI18n()
  const loading = ref(false)

  const refresh = () => {
    loading.value = true
  }

  return { loading, refresh, t }
}
```

## 国际化

### 新增代码要求

- 新增用户可见文本优先使用 i18n
- 优先复用现有 key，必要时再新增 key

```vue
<script setup lang="ts">
const { t } = useI18n()
</script>

<template>
  <t-button>{{ t('common.confirm') }}</t-button>
</template>
```

### 当前仓库现状

- `src/plugins/i18n.ts` 中手动聚合各模块翻译
- 历史代码中仍有部分硬编码文本，修改旧文件时按触达范围渐进迁移

## 类型定义

- 全局类型主要放在 `src/types/`
- 组件局部类型可直接写在 `<script setup lang="ts">`
- 新增业务类型优先靠近使用方，复用类型再提升到 `src/types/`
