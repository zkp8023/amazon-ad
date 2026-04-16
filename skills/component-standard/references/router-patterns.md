---
name: router-patterns
description: 项目路由模块拆分、守卫逻辑和新增路由时应遵循的当前结构
---

# 路由规范

## 当前仓库结构

- 路由入口在 `src/router/index.ts`
- 业务模块路由拆在 `src/router/routes/`
- `routes/index.ts` 聚合的并不都是数组，很多是单个业务树对象

## 真实业务路由模式

当前常见写法是一个模块导出一个 `RouteRecordRaw` 对象：

```ts
import type { RouteRecordRaw } from 'vue-router'

export const myModule: RouteRecordRaw = {
  path: '/my-module',
  name: 'myModule',
  redirect: '/my-module/list',
  meta: {
    title: 'menu.my_module',
    icon: 'menu_my_module',
  },
  children: [
    {
      path: '/my-module/list',
      name: 'MyModuleList',
      component: () => import('@/pages/myModule/index.vue'),
      meta: {
        title: 'menu.my_module',
      },
    },
  ],
}
```

然后在 `src/router/routes/index.ts` 中聚合：

```ts
export const routes: RouteRecordRaw[] = [
  myModule,
  otherModule,
]
```

## 新增路由建议

- 先看相邻业务模块是导出对象还是数组，优先保持一致
- 页面入口通常直接指向 `src/pages/<module>/index.vue`
- `meta.title` 优先使用 i18n key，而不是硬编码标题
- 如果该业务模块已有 `redirect`、`icon`、`roles` 约定，新增子路由时延续它

## 守卫关注点

`src/router/index.ts` 当前守卫会处理：

- `NProgress`
- 登录校验
- 白名单放行
- 基于 `meta.roles` 的权限判断
- 页面标题写入
- 全局 store 中部分状态同步

因此新增路由时要注意：

- 是否需要加入白名单
- 是否需要配置 `meta.roles`
- `meta.title` 是否可被 i18n 正确解析

## 命名建议

- `path` 沿用当前业务模块已有风格
- `name` 保持和相邻路由一致，不强行统一成另一种大小写
- 同一业务树内，新增页面优先贴近现有路径结构
