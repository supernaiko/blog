---
title: 'Astro 页面切换慢的原因与优化'
pubDate: 2026-08-03
description: '解释 Astro 静态站点页面切换慢的原因,以及 View Transitions、prefetch 等优化方案'
author: '学习笔记'
tags: ["astro", "性能优化", "View Transitions", "prefetch"]
---

# Astro 页面切换慢的原因与优化

## 先搞清楚:1 秒算慢吗?

先给你一个参考标准:

| 切换感受 | 时间 | 评价 |
|---|---|---|
| 几乎无感 | < 100ms | 像 SPA 一样丝滑 |
| 还行 | 100-300ms | 用户能接受 |
| 稍慢但能用 | 300-1000ms | 你的项目目前在这 |
| 明显卡顿 | > 1s | 需要优化 |

你的"约 1 秒"属于"稍慢但能用"。不算病,但可以优化到"几乎无感"。

## 为什么慢?原因分析

### 原因 1:全页刷新(最主要原因)

Astro 默认是**多页应用(MPA)**,不是单页应用(SPA)。

什么意思?你点导航链接时,浏览器会:

1. 销毁当前页面
2. 向服务器发请求要新页面
3. 等服务器返回 HTML
4. 重新解析 HTML、CSS、JS
5. 重新渲染整个页面

整个流程下来,1 秒很正常。

**对比 SPA(单页应用)**:React/Vue 写的 SPA,点击链接时不会重新请求整个 HTML,而是用 JS 在前端切换内容,所以感觉"瞬间"完成。

生活比喻:
- **MPA(你现在)** = 每次去超市买东西,都要回家放下东西再去一次
- **SPA** = 在超市里逛,看到想要的直接拿,不用反复回家

### 原因 2:没有 View Transitions

页面切换时,浏览器先把旧页面"清空"(短暂白屏),再渲染新页面。这个"白屏瞬间"会让用户感觉"卡顿了一下"。

View Transitions API 是浏览器新特性,可以在切换时做平滑过渡动画(比如淡入淡出),消除白屏感。

### 原因 3:React 运行时被重复加载

你的 Navigation 是 React 组件。Astro 虽然是 SSR(服务端渲染成静态 HTML),但如果组件加了 `client:xxx` 指令,浏览器每次切换页面都要重新下载并执行 React 的 JS。

虽然你现在没加 client 指令,但 React 的 JS 包还是会包含在页面里,浏览器要解析它(即使不激活)。

### 原因 4:没有 prefetch(预加载)

浏览器只有当用户**点击**链接时,才开始请求新页面。如果能在用户**鼠标悬停**链接时就提前请求,点击时就能"瞬间"显示。

## 优化方案

### 方案 1:启用 View Transitions(最推荐,改动最小)

#### 是什么

View Transitions 是 Astro 内置的一个组件,启用后:

- 页面切换不再是"白屏 → 新页面"
- 而是平滑过渡(默认是淡入淡出)
- 视觉上感觉"瞬间"完成,即使实际加载时间没变

#### 怎么用

在你的 layout 文件(`BlogPost.astro`)和页面文件(`index.astro`、`blog.astro`、`about.astro`)的 `<head>` 里加一行:

```astro
---
import { ClientRouter } from 'astro:transitions';
---

<html lang="zh">
  <head>
    <meta charset="utf-8" />
    <title>{frontmatter.title}</title>
    <ClientRouter />
  </head>
  <body>
    ...
  </body>
</html>
```

#### 为什么有效

`ClientRouter`(老版本叫 `ViewTransitions`)做的事:

1. 拦截页面内的链接点击
2. 用 fetch 异步请求新页面
3. 新旧页面同时存在,做淡入淡出过渡
4. 看起来像 SPA 切换,无白屏

生活比喻:之前是"关灯 → 换布景 → 开灯",现在是"两个布景同时存在,慢慢一个淡出一个淡入"。

#### 浏览器兼容性

- Chrome、Edge、Safari、移动端都支持
- 老浏览器(IE 没有,旧版 Safari 可能不行)会自动降级成普通切换,不会报错

### 方案 2:启用 prefetch(预加载)

#### 是什么

Astro 内置 prefetch 功能,启用后:

- 鼠标悬停链接时,提前下载目标页面
- 用户真正点击时,页面已经在缓存里,瞬间显示

#### 怎么用

在 `astro.config.mjs` 加配置:

```js
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import mdx from '@astrojs/mdx';

export default defineConfig({
  integrations: [react(), mdx()],
  prefetch: {
    prefetchAll: true,      // 预加载所有页面链接
    defaultStrategy: 'hover' // 鼠标悬停时预加载
  }
});
```

#### 配置项解释

| 选项 | 作用 |
|---|---|
| `prefetchAll: true` | 自动给所有 `<a>` 链接加 prefetch |
| `defaultStrategy: 'hover'` | 鼠标悬停时触发预加载(默认) |
| `defaultStrategy: 'viewport'` | 链接进入视口时触发 |
| `defaultStrategy: 'tap'` | 点击瞬间触发(比 hover 晚一点) |
| `defaultStrategy: 'load'` | 页面加载完就预加载所有链接 |

#### hover vs viewport

- `hover`:用户鼠标移到链接上才预加载,比较省流量
- `viewport`:链接一出现在屏幕上就预加载,更快但费流量

个人博客推荐 `hover`,平衡速度和流量。

### 方案 3:减少 React 使用(可选,改动大)

你现在唯一的 React 组件是 `Navigation.jsx`,它只是几个 `<a>` 标签,没有任何交互逻辑(state、事件)。

#### 改成 .astro 组件

新建 `src/components/Navigation.astro`:

```astro
---
---

<div>
  <a href="/">首页</a>
  <a href="/about/">关于</a>
  <a href="/blog/">博客</a>
</div>
```

然后把所有 `import Navigation from './Navigation.jsx'` 改成 `import Navigation from './Navigation.astro'`。

#### 好处

- 去掉 React 依赖,页面更轻量
- 不用再下载 React 的 JS
- 构建/部署更快

#### 坏处

- 你失去了"用 React 写组件"的学习机会
- 以后真要用 React 的交互组件时,React 还是会回来

### 方案 4:检查图片体积(可能有外部图片慢)

你的文章 frontmatter 里有这种图片:

```yaml
image:
  url: 'https://docs.astro.build/assets/rose.webp'
```

这是外链图片,每次页面加载都要请求 docs.astro.build 这个外部服务器。如果那个服务器慢,你的页面就慢。

#### 怎么排查

打开浏览器的开发者工具(F12) → Network 标签 → 刷新页面 → 看"瀑布图",找出哪个请求最慢。

#### 怎么优化

- 把外链图片下载到本地 `public/` 目录,用本地路径
- 或者在 layout 里用 `loading="lazy"` 让图片懒加载
- 或者干脆不用图片

### 方案 5:检查部署平台

你部署在哪里?Vercel?Netlify?GitHub Pages?

不同平台的 CDN(内容分发网络)速度不一样:

| 平台 | 速度 | 免费层 |
|---|---|---|
| Vercel | 快(全球 CDN) | 有 |
| Netlify | 快 | 有 |
| GitHub Pages | 较慢(无国内 CDN) | 免费 |
| Cloudflare Pages | 快 | 免费 |

如果你在国内访问 GitHub Pages 部署的站,1 秒算快的。换成 Vercel 或 Cloudflare 可能更快。

## 推荐的优化顺序

按"性价比"排序(改动小收益大):

1. **方案 1(View Transitions)**:加一行代码,立刻见效
2. **方案 2(prefetch)**:改 config,鼠标悬停就预加载
3. **方案 4(检查图片)**:把外链图片改本地
4. **方案 3(去 React)**:如果你不打算继续用 React,可以改
5. **方案 5(换平台)**:如果是 GitHub Pages 慢,换 Vercel

## 实战:给你的项目加 View Transitions

### 第一步:改 BlogPost.astro

```astro
---
import Navigation from '../pages/components/Navigation.jsx';
import { ClientRouter } from 'astro:transitions';

const { frontmatter } = Astro.props;
---

<html lang="zh">
  <head>
    <meta charset="utf-8" />
    <title>{frontmatter.title}</title>
    <ClientRouter />
  </head>
  <body>
    <Navigation />
    <h1>{frontmatter.title}</h1>
    <p>作者:{frontmatter.author}</p>
    <p>发布日期:{new Date(frontmatter.pubDate).toLocaleDateString('zh-CN')}</p>
    <hr />
    <article>
      <slot />
    </article>
  </body>
</html>
```

### 第二步:改 index.astro、blog.astro、about.astro

这几个文件的 `<head>` 里也都要加 `<ClientRouter />`。

### 第三步:加 prefetch(可选)

在 `astro.config.mjs` 加 prefetch 配置。

## 替代方案:用 SPA 模式

如果你想要"完全像 React SPA 一样"的体验,可以:

- 用 Next.js(React 全栈框架,默认 SPA)
- 用 Gatsby(React 静态站点生成器,带 Link 组件预加载)
- 用 SvelteKit(Svelte 版 SPA)

但这些都是"换框架",代价大。Astro 的 View Transitions + prefetch 已经能让多页应用感觉像 SPA 了,不需要换。

## 怎么验证优化效果

优化后,用浏览器 DevTools 测一下:

1. F12 → Network 标签
2. 勾选 "Disable cache"(禁用缓存,模拟首次访问)
3. 点击切换页面
4. 看请求时间和 DOMContentLoaded 时间

对比优化前后的数字,看提升了多少。

## 总结

- 1 秒不算病,但可以优化到"几乎无感"
- 主要原因是全页刷新(无 View Transitions)
- 最有效的优化是启用 View Transitions(加一行代码)
- 其次是 prefetch(鼠标悬停预加载)
- 检查外链图片是不是拖慢了页面
- 如果是部署平台问题,考虑换 Vercel/Cloudflare
