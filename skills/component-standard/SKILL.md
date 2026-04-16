---
name: component-standard
description: Use when working in the current project on Vue components, pages, dialogs, styles, API integration, stores, routing, tables, charts, or i18n, and you need repo-specific conventions that match the existing codebase while guiding new code toward preferred patterns.
---

# AmazonAD_Vue3 项目规范

> 这份 skill 分两层使用：先遵循当前仓库真实约定，再在新增代码中优先采用推荐模式。不要把“推荐模式”误当成“全仓已完全统一”。

## 1. 当前项目事实

| 类别     | 现状                                                                                                                                  |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| 框架     | Vue 3 + TypeScript + Vite                                                                                                             |
| UI       | `tdesign-vue-next` + `tdesign-icons-vue-next`                                                                                         |
| 样式     | UnoCSS + Less                                                                                                                         |
| 状态管理 | Pinia + `pinia-plugin-persistedstate`                                                                                                 |
| 国际化   | `vue-i18n`，但历史代码中仍存在未完全 i18n 化的文本                                                                                    |
| 表格     | TDesign Table 为主，部分复杂场景使用 `vxe-table`                                                                                      |
| 图表     | ECharts，封装在 `src/components/echarts/`                                                                                             |
| 自动导入 | `vue`、`vue-router`、`vue-i18n`、`pinia`、`@vueuse/core`，以及 `src/components/**/*`、`src/store/**/*`、`src/composables/**/*` 的导出 |
| 别名     | `@` -> `src`，`~` -> `src/types`                                                                                                      |

## 2. 新增代码推荐模式

### 2.1 组件

- 新增 Vue 组件优先使用 `<script setup lang="ts">`
- 新增组件优先写 `defineOptions({ name: 'ComponentName' })`
- `props`、`emits`、`v-model` 优先使用类型声明式写法
- 模板 ref 优先使用 `useTemplateRef`
- 修改旧组件时，优先保持局部风格一致，不为了“统一”做大范围重写

### 2.2 模板与样式

- TDesign 组件模板中使用 kebab-case，例如 `<t-dialog>`
- 自定义组件新代码优先使用 PascalCase，但旧代码保留现有风格即可
- 不要一上来写大段原生 CSS，布局和通用视觉优先直接使用 UnoCSS 原子类
- 如果同一个元素需要的原子类超过 4 个，或一组原子组合已经影响可读性，优先在 `<style scoped lang="less">` 中使用 `--uno`、`--at-apply` 这类指令方式收敛
- 只有 UnoCSS 原子类和指令方式都不适合表达时，再写原生 Less / CSS
- 穿透第三方组件样式用 `:deep()`
- 已存在 `--uno` / `--at-apply` 用法时可继续沿用

### 2.3 文本与 i18n

- 新增用户可见文本必须优先接入 i18n, `locals/common`文件夹下有公用的键值对,已有的键值对不要重复创建
- 修改旧代码时，如果当前改动已经触达硬编码文案，顺手迁移到 i18n
- 不要假设仓库现状已经 100% i18n 化

### 2.4 异步与提示

- 新增异步错误处理默认使用 `await-to-js`
- 历史代码如果已经使用 `@/utils/awaitto`，局部修改时可保持一致
- `MessagePlugin` 当前可通过自动导入直接使用，无需手动 import

### 2.5 Store 与路由

- Store 定义在仓库中是混合风格，新代码优先参考相邻模块
- 路由按业务模块拆在 `src/router/routes/`，新增路由优先遵循现有业务树结构

## 3. 高风险注意点

- `src/components/**/*`、`src/store/**/*`、`src/composables/**/*` 的导出会参与自动导入，新增导出时避免使用过于泛化的名字，例如 `config`、`utils`、`data`
- 不要把 skill 里的“推荐模式”当作对历史代码的强制清洗要求
- 涉及表格、图表、路由、store 时，优先先看相邻业务模块，而不是只套模板

## 4. 按任务查阅参考文档

| 主题               | 使用时机                                | 参考                                                     |
| ------------------ | --------------------------------------- | -------------------------------------------------------- |
| 文件结构           | 创建新组件、页面、局部 composable 前    | [file-structure](references/file-structure.md)           |
| 自动导入           | 不确定是否需要 import，或新增导出命名时 | [auto-import](references/auto-import.md)                 |
| 样式规范           | 写 UnoCSS、Less、主题变量时             | [styling](references/styling.md)                         |
| API / Store / i18n | 调 API、写 store、处理异步和翻译时      | [api-patterns](references/api-patterns.md)               |
| 组件模板           | 新建标准组件、对话框、图表组件时        | [component-templates](references/component-templates.md) |
| 表格模式           | 使用 TDesign Table / VXE Table 时       | [table-patterns](references/table-patterns.md)           |
| 路由规范           | 增加页面路由、读守卫逻辑时              | [router-patterns](references/router-patterns.md)         |
