---
name: table-patterns
description: 项目表格使用规范（TDesign Table、VXE Table、分页、排序、列配置）
---

# 表格使用规范

项目中表格是最常见的组件，主要使用 TDesign Table（`t-table`）和 VXE Table（`vxe-grid`）。

## TDesign Table 标准模式

### 基本结构

```vue
<script setup lang="ts">
import type { TableProps } from 'tdesign-vue-next'
import to from 'await-to-js'
import { getList } from '@/api/myModule'

// 分页（使用项目 composable）
const { paginationDetail, cntSelectKey } = usePagination()

// 列定义
const baseColumns: CustomColumn.TableCol[] = [
  { colKey: 'row-select', type: 'multiple', width: 50, fixed: 'left' },
  { colKey: 'name', title: t('common.name'), width: 200, ellipsis: true, fixed: 'left' },
  { colKey: 'status', title: t('common.status'), width: 120 },
  { colKey: 'clicks', title: t('common.clicks'), width: 100, sorter: true, align: 'right' },
  { colKey: 'cost', title: t('common.cost'), width: 120, sorter: true, align: 'right' },
  { colKey: 'operation', title: t('common.operation'), width: 150, fixed: 'right' },
]

// 可配置列（通过 custom-col-config 组件管理）
const columns = ref([...baseColumns])

// 数据
const loading = ref(false)
const tableData = ref<any[]>([])
const footData = ref<any[]>([])

// 排序
const sortInfo = ref({ sortBy: '', descending: true })

// 获取数据
const fetchData = async () => {
  loading.value = true
  const [err, res] = await to(getList({
    page: paginationDetail.current,
    page_size: paginationDetail.pageSize,
    order: sortInfo.value.sortBy,
    order_by: sortInfo.value.descending ? 'desc' : 'asc',
  }))
  loading.value = false
  if (err)
    return
  tableData.value = res.list
  paginationDetail.total = res.total
  footData.value = res.summary ? [res.summary] : []
}

// 排序变化
const handleSortChange: TableProps['onSortChange'] = (sort) => {
  sortInfo.value = { sortBy: sort.sortBy as string, descending: sort.descending }
  fetchData()
}

// 分页变化
const handlePageChange = () => {
  fetchData()
}

// 选择变化
const handleSelectChange: TableProps['onSelectChange'] = (selectedRowKeys) => {
  cntSelectKey.value = selectedRowKeys
}

onMounted(() => fetchData())
</script>

<template>
  <t-table
    row-key="id"
    resizable
    :data="tableData"
    :columns="columns"
    :loading="loading"
    :pagination="paginationDetail"
    :selected-row-keys="cntSelectKey"
    :foot-data="footData"
    :foot-affixed-bottom="true"
    :hide-sort-tips="true"
    cell-empty-content="/"
    @page-change="handlePageChange"
    @sort-change="handleSortChange"
    @select-change="handleSelectChange"
  >
    <!-- 自定义单元格插槽 -->
    <template #name="{ row }">
      <OverTooltip :content="row.name">
        {{ row.name }}
      </OverTooltip>
    </template>

    <template #status="{ row }">
      <t-tag :theme="row.status === 'active' ? 'success' : 'default'">
        {{ row.status }}
      </t-tag>
    </template>

    <template #operation="{ row }">
      <t-link theme="primary" @click="handleEdit(row)">
        {{ t('common.edit') }}
      </t-link>
      <t-link theme="danger" @click="handleDelete(row)">
        {{ t('common.delete') }}
      </t-link>
    </template>
  </t-table>
</template>
```

### 关键 Props 说明

| Prop | 说明 |
|------|------|
| `row-key` | 行唯一标识字段，通常为 `"id"` |
| `resizable` | 允许拖拽调整列宽 |
| `cell-empty-content` | 空单元格显示内容，项目统一用 `"/"` |
| `hide-sort-tips` | 隐藏排序提示 |
| `foot-data` | 表尾汇总行数据 |
| `foot-affixed-bottom` | 表尾固定在底部 |

## 列配置组件

项目封装了 `custom-col-config` 组件用于列的显示/隐藏控制：

```vue
<template>
  <custom-col-config v-model="columns" :base-columns="baseColumns" storage-key="myPage" />
  <t-table :columns="columns" ... />
</template>
```

## 自定义单元格组件

项目封装了多种表格单元格组件（位于 `src/components/table/` 和 `src/components/deliveryTableCell/`）：

- **OverTooltip** — 文本溢出时显示 tooltip
- **DeliveryTableCell** — 投放数据单元格（含变化趋势）
- **ApplyState** — 应用状态显示
- **ApplyAction** — 应用/取消应用操作按钮
- **BatchOperation** — 批量操作弹窗

## 分页 Composable

```ts
// usePagination 返回分页配置和选中行 key
const { paginationDetail, cntSelectKey } = usePagination()

// paginationDetail 包含：
// { current, pageSize, total, pageSizeOptions, showJumper, onChange, onPageSizeChange }
```

## VXE Table 使用模式

当需要更复杂的表格功能（虚拟滚动、单元格编辑等）时使用 VXE Table：

```vue
<script setup lang="ts">
import type { VxeGridProps } from 'vxe-table'

const gridOptions = reactive<VxeGridProps>({
  border: true,
  showOverflow: true,
  height: 'auto',
  columnConfig: { resizable: true },
  columns: [
    { field: 'name', title: t('common.name'), width: 200 },
    { field: 'value', title: t('common.value'), width: 150 },
  ],
  data: [],
})
</script>

<template>
  <vxe-grid v-bind="gridOptions" />
</template>
```

## 表格页面完整模板

一个典型的列表页面包含：搜索区域 + 操作栏 + 表格 + 分页

```vue
<script setup lang="ts">
import to from 'await-to-js'
import { getList } from '@/api/myModule'

defineOptions({ name: 'MyListPage' })

const { t } = useI18n()
const { paginationDetail, cntSelectKey } = usePagination()

// 搜索条件
const searchParams = reactive({
  keyword: '',
  status: '',
})

const loading = ref(false)
const tableData = ref<any[]>([])
const columns = ref<CustomColumn.TableCol[]>([
  { colKey: 'row-select', type: 'multiple', width: 50 },
  { colKey: 'name', title: t('common.name'), width: 200, ellipsis: true },
  { colKey: 'status', title: t('common.status'), width: 120 },
  { colKey: 'operation', title: t('common.operation'), width: 150 },
])

const fetchData = async () => {
  loading.value = true
  const [err, res] = await to(getList({
    ...searchParams,
    page: paginationDetail.current,
    page_size: paginationDetail.pageSize,
  }))
  loading.value = false
  if (err)
    return
  tableData.value = res.list
  paginationDetail.total = res.total
}

const handleSearch = () => {
  paginationDetail.current = 1
  fetchData()
}

const handleReset = () => {
  Object.assign(searchParams, { keyword: '', status: '' })
  handleSearch()
}

onMounted(() => fetchData())
</script>

<template>
  <div class="p-20">
    <!-- 搜索区域 -->
    <div class="mb-16 fc flex-wrap gap-12">
      <t-input
        v-model="searchParams.keyword"
        :placeholder="t('common.please_input')"
        clearable
        class="w-200"
        @enter="handleSearch"
      />
      <t-select
        v-model="searchParams.status"
        :placeholder="t('common.please_select')"
        clearable
        class="w-150"
      >
        <t-option value="active" :label="t('common.active')" />
        <t-option value="paused" :label="t('common.paused')" />
      </t-select>
      <t-button theme="primary" @click="handleSearch">
        {{ t('common.search') }}
      </t-button>
      <t-button variant="outline" @click="handleReset">
        {{ t('common.reset') }}
      </t-button>
    </div>

    <!-- 表格 -->
    <t-table
      row-key="id"
      resizable
      :data="tableData"
      :columns="columns"
      :loading="loading"
      :pagination="paginationDetail"
      :selected-row-keys="cntSelectKey"
      cell-empty-content="/"
      @page-change="fetchData"
      @select-change="(keys) => cntSelectKey = keys"
    >
      <template #operation="{ row }">
        <t-link theme="primary" @click="handleEdit(row)">
          {{ t('common.edit') }}
        </t-link>
      </template>
    </t-table>
  </div>
</template>
