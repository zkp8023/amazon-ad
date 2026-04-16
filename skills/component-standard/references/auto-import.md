---
name: auto-import
description: 项目自动导入配置、手动导入边界，以及新增导出时的命名注意事项
---

# 自动导入（Auto Import）

项目同时使用了 `unplugin-auto-import` 和 `unplugin-vue-components`。写代码前先判断符号是否已经自动注入，避免重复 import。

## 自动导入来源

### 框架与工具库

- `vue`
- `vue-router`
- `vue-i18n`
- `pinia`
- `@vueuse/core`

### 项目目录导出

- `src/components/**/*`
- `src/store/**/*`
- `src/composables/**/*`

这意味着这些目录里导出的部分符号会直接进入全局类型声明，例如 `useStore`、`usePagination`、`RenderVariable`、`config`。

### TDesign

- 模板中的 TDesign 组件通过 `TDesignResolver` 自动注册
- `MessagePlugin` 当前也已自动导入，可直接使用

## 当前可直接用、通常不需要手动 import 的

```ts
// Vue / Vue Router / i18n / Pinia / VueUse
ref
reactive
computed
watch
onMounted
useRouter
useRoute
useI18n
defineStore
storeToRefs
useTemplateRef

// 项目导出
useStore
useGlobalStore
useUserStore
usePagination
useCustomColumns

// TDesign 命令式 API
MessagePlugin
```

## 仍然需要手动 import 的

```ts
// 类型
import type { TableProps, PaginationProps } from 'tdesign-vue-next'
import type { ECOption } from '@/components/echarts/config'

// 第三方库
import to from 'await-to-js'
import dayjs from 'dayjs'
import { cloneDeep } from 'lodash-es'

// TDesign 图标
import { ChevronDownIcon } from 'tdesign-icons-vue-next'

// API 函数
import { getTemplates } from '@/api/common'

// 本地显式文件依赖
import config from './config'
```

## 命名注意点

- 不要在 `src/components/**/*`、`src/store/**/*`、`src/composables/**/*` 中新增过于泛化的导出名，例如 `config`、`utils`、`data`
- 新增可复用导出前，先检查 [auto-imports.d.ts](/D:/LS/AmazonAD_Vue3/src/types/common/auto-imports.d.ts) 是否已有同名全局符号
- 遇到旧代码手动 import `MessagePlugin`，局部修改可保留，不必为此专门重构

## 配置位置

自动导入配置位于 [index.ts](/D:/LS/AmazonAD_Vue3/build/plugins/index.ts)。
