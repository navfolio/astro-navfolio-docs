---
title: "Page Module 页面能力配置指南"
description: "了解 navfolio 如何把 Projects、Vibe 等页面能力配置为可开启、可关闭、可改路由的 page module。"
date: "2026-07-12T16:30:00+08:00"
draft: false
showHeroImage: false
tags:
  - Astro
  - Configuration
  - Navfolio
categories:
  - 站点建设
series:
  - Navfolio 模块手册
comments: true
sidebar:
  enable: true
  toc: true
  relatedPosts: true
---

navfolio 默认展示完整的模板能力，所以 `Projects` 和 `Vibe` 页面都是开启的。对于真实个人站点，你可以把这些页面当成 page module：需要就保留，不需要就关闭，想换路径也可以直接在构建配置里调整。

这类配置放在：

```bash
navfolio.config.ts
```

`src/config/site.toml` 继续负责页面文案、导航标题、个人资料等站点内容；`navfolio.config.ts` 负责会影响构建结果的能力开关，例如某个页面是否生成路由、是否注册内容集合。

## 默认开启

模板默认启用内置页面模块：

```ts
import { markdownPlugin } from "@navfolio/plugin-markdown";

import { projectsModule, vibeModule } from "./src/modules";
import { defineNavfolioConfig } from "./src/plugins/config";

export default defineNavfolioConfig({
  modules: [projectsModule(), vibeModule()],
  plugins: [markdownPlugin()],
});
```

默认路由是：

| 模块               | 默认路由    | 内容目录                |
| ------------------ | ----------- | ----------------------- |
| `projectsModule()` | `/projects` | `src/content/projects/` |
| `vibeModule()`     | `/vibe`     | `src/content/vibe/`     |

默认开启是为了让模板站点能完整展示作品集页面、碎片记录页面、博客文章和其他内置体验。你在真正落地自己的站点时，可以按需要删减。

## 禁用页面

如果你暂时不需要某个页面，可以把对应模块设为 `enabled: false`：

```ts
export default defineNavfolioConfig({
  modules: [projectsModule({ enabled: false }), vibeModule({ enabled: false })],
  plugins: [markdownPlugin()],
});
```

禁用后不只是隐藏导航，而是从构建层面移除这项能力：

- 不注入对应页面路由。
- 不注册对应内容集合。
- 不生成对应 `dist` 输出。
- 顶部导航中匹配默认模块路由的入口会被过滤。
- 对应内容脚手架命令会给出禁用提示。

也就是说，禁用 `vibeModule()` 后，`/vibe` 不会存在，`src/content/vibe/` 也不会作为 Astro content collection 参与构建。

## 自定义路由

你也可以保留页面能力，但改成自己的路径。例如把 Vibe 页面作为 `/space` 使用：

```ts
export default defineNavfolioConfig({
  modules: [
    projectsModule(),
    vibeModule({
      route: "/space",
    }),
  ],
  plugins: [markdownPlugin()],
});
```

构建结果会生成 `/space`，不会再生成 `/vibe`。站点顶部导航、博客页顶部导航、搜索结果类型识别等位置会读取 resolved route，所以它们会跟随新的路径。

`route` 可以写成 `space` 或 `/space`，最终都会规范化为 `/space`。不同模块不能占用同一个路由，否则构建配置会直接报错，避免出现两个页面抢同一个地址的问题。

## 内容和命令

页面模块开启后，内容仍然按原来的目录管理：

```bash
src/content/projects/
src/content/vibe/
```

常用脚手架命令：

```bash
bun run post:new -- my-post
bun run project:new -- my-project
bun run vibe:new -- 2026-07-12-small-note
```

`post:new` 用于博客文章，`project:new` 用于项目页内容，`vibe:new` 用于碎片记录。如果 `projectsModule` 或 `vibeModule` 被禁用，对应命令会停止创建文件并提示先启用模块。

## 适合怎么取舍

建议把 page module 当成“页面级能力包”，而不是单纯的导航开关：

- 只是想改页面标题、说明文案、导航文本：改 `src/config/site.toml`。
- 想换页面路径：改 `navfolio.config.ts` 里的 `route`。
- 想完全移除某个页面能力：把模块设为 `enabled: false`。
- 想保留页面但暂时没有内容：保持模块开启，让页面展示空状态或默认文案。

这个分工可以让内容配置保持易读，也让构建结果和你实际启用的功能保持一致。
