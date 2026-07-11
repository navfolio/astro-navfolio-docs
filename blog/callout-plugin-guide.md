---
title: '使用 Callout 插件渲染 Obsidian 风格提示框'
description: '了解 @navfolio/plugin-callout 如何在 Navfolio 中解析 Obsidian Callout 语法，并渲染 note、tip、warning、danger、quote 等提示框。'
date: '2026-07-12T03:00:00+08:00'
draft: false
showHeroImage: false
tags:
  - Markdown
  - Callout
  - Obsidian
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

`@navfolio/plugin-callout` 是 Navfolio 的 Obsidian 风格 Callout 插件。它会把 Markdown 引用块中的 `[!type]` 标记解析成更适合阅读的提示框，用来承载说明、提醒、警告、FAQ、引用和示例。

这个插件适合从 Obsidian 迁移内容的写作者：原本写在笔记里的 Callout 语法可以直接放进 Navfolio 的文章和文档里，不需要改成额外的 Astro 组件。

这篇文章同时覆盖普通、折叠、展开和嵌套 Callout，可作为本地构建时的渲染检查页面。

## 安装与启用

在 Navfolio 项目中安装插件：

```sh
bun add @navfolio/plugin-callout@github:navfolio/plugin-callout
```

然后在 `navfolio.config.ts` 中启用：

```ts
import { calloutPlugin } from '@navfolio/plugin-callout';
import { markdownPlugin } from '@navfolio/plugin-markdown';

import { defineNavfolioConfig } from './src/plugins/config';

export default defineNavfolioConfig({
  plugins: [
    calloutPlugin(),
    markdownPlugin({
      expressiveCode: true,
      math: {
        enabled: true,
      },
      mermaid: true,
      responsiveTables: true,
    }),
  ],
});
```

最后在全局样式中引入默认样式：

```css
@import '@navfolio/plugin-callout/styles.css';
```

## 基础语法

Callout 使用标准的 Obsidian 写法：

```md
> [!tip] 写作提示
> 把重要信息放进 Callout，可以让读者更快抓住上下文。
```

渲染效果如下：

> [!tip] 写作提示
> 把重要信息放进 Callout，可以让读者更快抓住上下文。

如果不写标题，插件会使用类型的默认标题：

```md
> [!note]
> 这是一条默认标题的 note。
```

> [!note]
> 这是一条默认标题的 note。

## 常用类型

插件内置了 Obsidian 常见类型和别名，包括 `note`、`info`、`tip`、`success`、`question`、`warning`、`danger`、`bug`、`example`、`quote` 等。

> [!info] 信息
> `info` 适合补充背景、版本说明和非阻断性的上下文。

> [!warning] 注意
> `warning` 适合提醒读者某个操作可能带来风险。

> [!danger] 危险操作
> `danger` 适合放置会破坏数据、影响部署或需要额外确认的步骤。

> [!quote] 引用
> `quote` 适合保留一句原文、观点或设计原则。

## 可折叠 Callout

在类型标记后加 `-`，会生成默认折叠的 `<details>`：

```md
> [!faq]- 为什么选择独立插件？
> 因为 Callout 属于 Markdown 渲染能力，单独维护可以复用到文档站、博客和后续 Navfolio 插件生态。
```

> [!faq]- 为什么选择独立插件？
> 因为 Callout 属于 Markdown 渲染能力，单独维护可以复用到文档站、博客和后续 Navfolio 插件生态。

在类型标记后加 `+`，会生成默认展开的 `<details>`：

```md
> [!example]+ 默认展开示例
> 这个区块会在页面加载时保持展开状态。
```

> [!example]+ 默认展开示例
> 这个区块会在页面加载时保持展开状态。

## 嵌套写法

Callout 本质上仍然来自 Markdown blockquote，所以可以继续嵌套：

```md
> [!question] 外层问题
> 这里是问题背景。
>
> > [!tip] 内层提示
> > 嵌套 Callout 可以用于补充说明。
```

> [!question] 外层问题
> 这里是问题背景。
>
> > [!tip] 内层提示
> > 嵌套 Callout 可以用于补充说明。

## 适合放在哪里

Callout 适合用于：

1. 操作步骤中的注意事项。
2. 文档里的 FAQ 和背景补充。
3. 迁移自 Obsidian 的知识库内容。
4. 博客文章中的重点、警告、引用和示例。

它不适合替代正文结构。如果一段内容本身应该成为章节，优先使用标题；如果它只是帮助读者理解正文的辅助信息，再使用 Callout。
