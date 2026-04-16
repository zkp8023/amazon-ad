---
name: component-templates
description: 项目组件模板（标准组件、对话框组件）
---

# 完整组件模板

## 标准 Vue 组件

```vue
<script setup lang="ts">
import { SomeIcon } from 'tdesign-icons-vue-next'

interface Props {
  /** 标题文本 */
  title: string
  /** 是否可见 */
  visible?: boolean
}

defineOptions({
  name: 'MyComponent',
})

const props = withDefaults(defineProps<Props>(), {
  visible: true,
})

const emits = defineEmits<{
  change: [value: string]
  close: []
}>()

const { t } = useI18n()
const { globalStore } = useStore()

// 响应式状态
const loading = ref(false)
const data = ref<any[]>([])

// 计算属性
const isEmpty = computed(() => data.value.length === 0)

// 方法
const handleChange = (val: string) => {
  emits('change', val)
}

// 生命周期
onMounted(() => {
  // 初始化逻辑
})
</script>

<template>
  <div v-if="visible" class="my-component">
    <div class="mb-10 fbc">
      <span class="font-title">
        {{ title }}
      </span>
      <SomeIcon class="cur-p" @click="emits('close')" />
    </div>
    <t-loading :loading="loading">
      <div v-if="isEmpty" class="h-200 fcc text-minor">
        {{ t('common.no_data') }}
      </div>
      <slot v-else></slot>
    </t-loading>
  </div>
</template>

<style scoped lang="less">
.my-component {
  padding: 16px;
  border-radius: @radius-medium;
  background: #fff;
}
</style>
```

## 对话框组件

```vue
<script setup lang="ts">
interface Props {
  title?: string
}

defineOptions({
  name: 'MyDialog',
})

const props = withDefaults(defineProps<Props>(), {
  title: '',
})

const emits = defineEmits<{ confirm: [data: any] }>()
const visible = defineModel<boolean>({ required: true })
const { t } = useI18n()

const handleConfirm = () => {
  emits('confirm', {})
  visible.value = false
}

const handleClose = () => {
  visible.value = false
}
</script>

<template>
  <t-dialog
    v-model:visible="visible"
    :header="title"
    width="600"
    @close="handleClose"
    @confirm="handleConfirm"
  >
    <slot></slot>
  </t-dialog>
</template>
```

## TSX 函数式组件（动态组件映射）

```tsx
import type { IDeliveryTableCellProps } from '@/components/deliveryTableCell/index.vue'
import DeliveryTableCell from '@/components/deliveryTableCell/index.vue'

interface Props extends Omit<IDeliveryTableCellProps, 'tabIndex'> {
  k: keyof typeof variableSlots
}

export const variableSlots = {
  clicks: (props: Props) => <DeliveryTableCell tabIndex="clicks" {...props} />,
  impressions: (props: Props) => <DeliveryTableCell tabIndex="impressions" {...props} />,
  // ...更多字段映射
}

export const RenderVariable = (props: Props) => (variableSlots[props.k])(props)
```

## ECharts 封装组件使用模式

项目将 ECharts 封装在 `src/components/echarts/` 中，包含：
- `config.ts` — 按需注册图表类型（Bar、Line、Pie、Radar、Gauge、Scatter）和组件（Tooltip、Grid、Legend、DataZoom 等），导出 `ECOption` 类型和配置好的 `echarts` 实例
- `index.vue` — 通用 ECharts 包装组件，处理初始化/销毁、响应式 resize、点击/图例事件

### 基本使用

```vue
<script setup lang="ts">
import type { ECOption } from '@/components/echarts/config'

const chartOption = computed<ECOption>(() => ({
  tooltip: { trigger: 'axis' },
  xAxis: {
    type: 'category',
    data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri'],
  },
  yAxis: { type: 'value' },
  series: [
    {
      type: 'line',
      data: [120, 200, 150, 80, 70],
      smooth: true,
    },
  ],
}))
</script>

<template>
  <Echarts :option="chartOption" height="300" />
</template>
```

### Echarts 组件 Props

| Prop | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `option` | `ECOption` | 必填 | ECharts 配置项 |
| `height` | `string` | `'100%'` | 图表高度 |
| `width` | `string` | `'100%'` | 图表宽度 |
| `theme` | `string` | — | ECharts 主题 |
| `renderer` | `'canvas' \| 'svg'` | `'canvas'` | 渲染器 |
| `resize` | `boolean` | `true` | 是否响应窗口 resize |

### 暴露的方法

```ts
// 通过 ref 访问 Echarts 组件实例
const chartRef = useTemplateRef('chartRef')

// 获取 ECharts 实例
chartRef.value?.getInstance()

// 手动触发 resize
chartRef.value?.resize()

// 重新绘制
chartRef.value?.draw(newOption)
```

### 动态数据更新

```vue
<script setup lang="ts">
import type { ECOption } from '@/components/echarts/config'
import to from 'await-to-js'
import { getChartData } from '@/api/myModule'

const chartData = ref<any[]>([])

const chartOption = computed<ECOption>(() => ({
  tooltip: { trigger: 'axis' },
  legend: { data: ['点击量', '花费'] },
  xAxis: { type: 'category', data: chartData.value.map(d => d.date) },
  yAxis: [
    { type: 'value', name: '点击量' },
    { type: 'value', name: '花费' },
  ],
  series: [
    { name: '点击量', type: 'bar', data: chartData.value.map(d => d.clicks) },
    { name: '花费', type: 'line', yAxisIndex: 1, data: chartData.value.map(d => d.cost) },
  ],
}))

const fetchChartData = async () => {
  const [err, res] = await to(getChartData(params))
  if (err)
    return
  chartData.value = res
}

onMounted(() => fetchChartData())
</script>

<template>
  <Echarts :option="chartOption" height="400" />
</template>
```

## SVG 图标使用

项目封装了 `SvgIcon` 组件（`src/components/svgIcon/`），用于渲染 `src/assets/icons/` 下的 SVG 文件：

```vue
<script setup lang="ts">
import { ChevronDownIcon } from 'tdesign-icons-vue-next'
</script>

<template>
  <!-- 使用 SVG 图标 -->
  <SvgIcon icon="common-icon_down" size="12px" />

  <!-- TDesign 图标（需手动 import） -->
  <ChevronDownIcon />
</template>
```
