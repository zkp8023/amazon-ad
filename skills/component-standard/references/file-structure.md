---
name: file-structure
description: 项目组件、页面、辅助文件和目录命名的真实结构，以及新增代码应如何贴近相邻模块
---

# 文件结构

这套仓库不是单一模板结构，而是“业务模块优先、相邻文件优先”。新增文件时，先看同目录最近的 2 到 3 个实现，再决定落点。

## 当前常见模式

### 通用组件目录模式

```text
src/components/myComponent/
  ├── index.vue
  ├── config.ts
  ├── useMyComponent.ts
  └── utils.ts
```

适用于可复用组件、图表封装、表格辅助组件。

### 平铺组件文件模式

```text
src/components/dialog/timeRuleDialog.vue
src/components/drawer/costChartsMessage.vue
```

适用于 `dialog`、`drawer` 这类历史上已经采用平铺命名的目录。新增同类文件时，优先延续该模式。

### 页面目录模式

```text
src/pages/myModule/index.vue
src/pages/myModule/components/MyDialog.vue
src/pages/myModule/composables/useMyModule.ts
```

适用于业务页面和页面私有组件。

## 新增代码怎么选

- 如果目标目录已经明显使用平铺命名，就继续平铺
- 如果目标目录以 `index.vue` 为入口并配套局部文件，就继续目录模式
- 页面私有组件、页面私有 composable，优先贴近页面目录，不要默认塞进全局 `src/components`
- 只有在多个页面复用、且语义稳定时，才提升到全局组件或全局 composable

## 命名建议

- 目录名优先保持与相邻目录一致，仓库中同时存在 `camelCase` 和 `snake_case` 页面目录
- 组件名新代码优先语义化命名，不使用过度抽象的 `index2`、`tempDialog`
- composable 使用 `useXxx`
- 配置和工具文件优先使用具体名字，避免 `config.ts`、`utils.ts` 泛滥到自动导入冲突

## 项目主目录参考

```text
src/
├── api/
├── assets/
├── components/
├── composables/
├── config/
├── directives/
├── enums/
├── layouts/
├── locales/
├── pages/
├── plugins/
├── router/
├── store/
├── style/
├── types/
└── utils/
```
