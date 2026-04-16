---
name: styling
description: 项目样式规范（Less变量、UnoCSS、CSS变量、深度选择器）
---

# 样式规范

## 样式书写优先级

Agent 在这个项目里写组件样式时，按下面顺序执行，不要跳步：

1. 先判断是否可以直接用 UnoCSS 原子类解决，尤其是布局、间距、对齐、文字颜色、尺寸、圆角、显隐、溢出等通用样式
2. 如果同一个元素上的原子类超过 4 个，或者同一组原子类已经明显影响模板可读性，就不要继续在 `class` 里堆原子，改为在 `<style scoped lang="less">` 中使用 `--uno`、`--at-apply` 等指令方式收敛
3. 只有当 UnoCSS 原子类和指令方式都不适合表达，或者需要写复杂选择器、状态覆盖、第三方组件覆盖、动态样式时，才写原生 Less / CSS

### 执行要求

- 不要一上来写大量原生 CSS
- 能用原子类解决的，不要改写成普通 CSS
- 能用 `--uno` / `--at-apply` 收敛的，不要继续在模板里堆很长的类名
- 原生 CSS 保留给 UnoCSS 不擅长的场景，而不是作为默认方案

## 使用 scoped Less

```vue
<style scoped lang="less">
.my-component {
  color: @color-primary;
  font-size: @font-size-medium;
  border-radius: @radius-medium;
}
</style>
```

## 全局 Less 变量

以下变量可直接在任何组件的 `<style>` 中使用（通过 Vite `additionalData` 自动注入自 `src/style/style.less`）：

```less
// 颜色
@color-success: #14ca64;
@color-error: #ff3600;
@color-warning: #fab007;
@color-primary: #237ffa;
@color-info: #111111;
@color-info2: #929aa6;

// 背景色
@back-success / @back-error / @back-warning / @back-primary / @back-info / @back-info2 / @back-info3

// 字体
@font-size-small: 12px;
@font-size-medium: 14px;
@font-size-large: 16px;
@font-size-title: 18px;

// 圆角
@radius-small: 2px;
@radius-medium: 4px;
@radius-large: 8px;
```

## UnoCSS 原子类

### 项目自定义 shortcuts

```text
fcc    → flex justify-center items-center（居中布局）
fbc    → flex justify-between items-center（两端对齐）
fc     → flex items-center（垂直居中）
f-col-c → flex-col justify-center items-center（纵向居中）
cur-p  → cursor-pointer（鼠标指针）
o-auto → overflow-auto
o-hidden → overflow-hidden
pre_wrap → whitespace-pre-wrap
preserve-wrap → pre_wrap break-all
```

### 项目自定义 rules

```text
ell-N  → N行省略（如 ell-2 两行省略）
a-d-N  → animation-delay: Nms
flex-N → flex: N
```

### rem 转 px 配置

> **重要**：UnoCSS 配置了 `presetRemToPx({ baseFontSize: 4 })`，即 `1rem = 4px`。
> 写 `w-100` 等于 `width: 100px`（UnoCSS 默认 `w-100` = `25rem`，乘以 4 = `100px`）。

### 使用示例

```vue
<template>
  <div class="mr-20 h-100 w-full flex gap-10">
    内容
  </div>
  <div class="ell-2">
    两行省略
  </div>
  <span class="text-primary_b">
    主色文字
  </span>
  <span class="text-error">
    错误色文字
  </span>
</template>
```

## 指令方式示例

当原子类开始变长时，优先收敛到样式块：

```vue
<template>
  <div class="toolbar">
    内容
  </div>
</template>

<style scoped lang="less">
.toolbar {
  --at-apply: mb-16 fc flex-wrap gap-12 px-16 py-12 bg-white rd-8;
}
</style>
```

## 什么时候再写原生 CSS

以下场景再考虑写原生 Less / CSS：

- 复杂选择器或层级关系
- `:deep()` 覆盖第三方组件样式
- `v-bind()` 动态样式
- UnoCSS 不好表达的细节视觉规则
- 需要精细控制伪类、伪元素、动画细节

## CSS 变量（主题色）

UnoCSS 主题色映射到 CSS 变量，可在模板中使用 `text-primary`、`text-error` 等：

```text
--primary, --primary_b, --success, --error, --warning, --info
--main, --minor, --little, --border
```

## 深度选择器

穿透 TDesign 组件样式时使用 `:deep()`：

```less
:deep(.t-icon) {
  transition: all 0.3s ease;
}
```

## CSS in JS（v-bind）

可使用 `v-bind` 在样式中绑定 props：

```less
.svg-icon {
  width: v-bind('props.size');
  height: v-bind('props.size');
}
```
