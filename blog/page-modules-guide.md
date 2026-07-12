---
title: 'Page Module 页面能力配置指南'
description: '了解 navfolio 如何把 Projects、Vibe 等页面能力配置为可开启、可关闭、可改路由的 page module。'
date: '2026-07-12T16:30:00+08:00'
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

page module 有两层配置入口：

- `src/config/site.toml`：快速内容配置，适合改导航文案、导航顺序、页面标题、说明文字和首页卡片。
- `navfolio.config.ts`：高级构建配置，适合改页面是否启用、真实路由、内容目录、内容脚手架和后续插件注册。

这个分工可以避免路由同时散落在多个地方。真实 route 由 `navfolio.config.ts` 决定，`site.toml` 里的导航可以引用模块 id。

## 默认开启

模板默认启用内置页面模块：

```ts
import { markdownPlugin } from '@navfolio/plugin-markdown';

import { projectsModule, vibeModule } from './src/modules';
import { defineNavfolioConfig } from './src/plugins/config';

export default defineNavfolioConfig({
  modules: [projectsModule(), vibeModule()],
  plugins: [markdownPlugin()],
});
```

默认路由和内容目录是：

| 模块               | 默认路由    | 内容目录                |
| ------------------ | ----------- | ----------------------- |
| `projectsModule()` | `/projects` | `src/content/projects/` |
| `vibeModule()`     | `/vibe`     | `src/content/vibe/`     |

默认开启是为了让模板站点能完整展示作品集页面、碎片记录页面、博客文章和其他内置体验。你在真正落地自己的站点时，可以按需要删减。

## 快速配置导航

推荐在 `src/config/site.toml` 里用 `module` 引用内置页面模块：

```toml
[[config.topNav.links]]
label = "Projects"
module = "projects"

[[config.topNav.links]]
label = "Vibe"
module = "vibe"
```

这样 `site.toml` 只负责导航的展示顺序和文字，真实路径仍由 `navfolio.config.ts` 里的模块配置决定。如果你之后把 `vibe` 改成 `/space`，导航会自动指向 `/space`。

普通页面和外链继续使用 `href`：

```toml
[[config.topNav.links]]
label = "About"
href = "/about"

[[config.topNav.links]]
label = "GitHub"
href = "https://github.com/dodolalorc/astro-navfolio"
```

旧写法 `href = "/projects"`、`href = "/vibe"` 仍然兼容。navfolio 会识别这些默认模块路由，并在你自定义 route 时自动改写为真实路径。不过新站点更推荐使用 `module`，因为它不会把构建级 route 重复写进 TOML。

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
- 使用 `module = "..."` 引用该模块的导航项会被过滤。
- 对应内容脚手架命令会给出禁用提示。

也就是说，禁用 `vibeModule()` 后，`/vibe` 不会存在，`src/content/vibe/` 也不会作为 Astro content collection 参与构建。

## 自定义路由

你可以保留页面能力，但改成自己的路径。例如把 Vibe 页面作为 `/space` 使用：

```ts
export default defineNavfolioConfig({
  modules: [
    projectsModule(),
    vibeModule({
      route: '/space',
    }),
  ],
  plugins: [markdownPlugin()],
});
```

构建结果会生成 `/space`，不会再生成 `/vibe`。站点顶部导航、博客页顶部导航、搜索结果类型识别等位置会读取 resolved route，所以它们会跟随新的路径。

`route` 可以写成 `space` 或 `/space`，最终都会规范化为 `/space`。不同模块不能占用同一个路由，否则构建配置会直接报错，避免出现两个页面抢同一个地址的问题。

## 内容和命令

页面模块开启后，内容仍然按模块配置里的目录管理：

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

也可以使用通用入口：

```bash
bun run content:new -- blog my-post
bun run content:new -- project my-project
bun run content:new -- vibe today-note
```

`blog` 是核心文章脚手架；`project` 和 `vibe` 来自启用的 page module。如果 `projectsModule` 或 `vibeModule` 被禁用，对应命令会停止创建文件并提示先启用模块。

模块可以在 `navfolio.config.ts` 里声明自己的脚手架信息：

```ts
customModule({
  route: '/space',
  scaffold: {
    command: 'space',
    collection: 'space',
    directory: 'src/content/space',
    defaultExtension: 'md',
    template: 'article',
  },
});
```

之后就可以使用：

```bash
bun run content:new -- space hello
```

内置模板支持 `article`、`project` 和 `vibe`。如果自定义模块需要完全不同的 frontmatter 或正文，也可以在模块的 scaffold 配置中提供自定义模板函数。

## 适合怎么取舍

建议把 page module 当成“页面级能力包”，而不是单纯的导航开关：

- 只是想改页面标题、说明文案、导航文本：改 `src/config/site.toml`。
- 想换页面路径、内容目录、脚手架命令：改 `navfolio.config.ts`。
- 想完全移除某个页面能力：把模块设为 `enabled: false`。
- 想保留页面但暂时没有内容：保持模块开启，让页面展示空状态或默认文案。

这个分工可以让内容配置保持易读，也让构建结果和你实际启用的功能保持一致。
