---
title: Web Performance Metrics
date: 2022-04-07 11:11:34
tags: [JavaScript, Browser]
---

> 总结下前端性能监控常用指标和计算方式

# FP 和 FCP

## FP

首次渲染的时间点。从用户开始访问 Web 页面的时间点到 FP 的时间点这段时间可以被视为 **白屏时间** 。反映当前 Web 页面的网络加载性能情况。

## FCP

FCP 指标测量页面从开始加载到页面内容的任何部分在屏幕上完成渲染的时间。“内容”指文本、图像、背景图、 `<svg>` 或非白色的 `<canvas>` 元素。

首次有内容渲染的时间点。从用户访问 Web 页面的时间点到 FCP 的时间点这段时间可以被视为 **无内容时间** 。反映当前 Web 页面的网络加载性能情况、页面 DOM 结构复杂度情况、inline script 的执行效率的情况。

FCP 在 1.8 秒以内（ P75 ）。

> 参考 https://web.dev/defining-core-web-vitals-thresholds/

## 如何计算

通过如下 API ，可获取 FP 和 FCP ，其中 startTime 为对应时间点。

```javascript
performance.getEntriesByType('paint');
```

{% asset_img fp_fcp.png fp_fcp %}

## 如何优化

### 消除阻塞资源

Lighthouse 中阻塞 FP 的 URL 列表。

-   head 中的 script ，无 defer 或 async
-   `<link rel="stylesheet">` ，无 disabled 属性，`media="all"`阻塞渲染。

Chrome DevTools 中 Coverage Tab 识别非关键 JS 和 CSS

### 缩小 CSS

minify css

### 移除未使用的 CSS

Lighthouse 中未使用的 CSS 列表。

### 预连接到所需的来源

`<link rel="preconnect">`

`<link rel="preload">`

`<link rel="dns-prefetch">`

为第三方源提早建立连接。

### 减少服务端响应时间 TTFB

Lighthouse 中 TTFB 列表。

# LCP

最大绘制内容指标会根据页面首次开始加载的时间点来报告可视区域内可见的最大图像或文本块完成渲染的相对时间。

LCP 是测量感知加载速度的一个以用户为中心的重要指标。

-   img 元素
-   svg 中的 image 元素
-   video 元素（使用封面图像）
-   通过 url()加载背景图像的元素
-   包含文本节点或其他行内级文本元素子元素的块级元素

## 如何计算

```javascript
new PerformanceObserver((entry_list) => {
    for (const entry of entry_list.getEntries()) {
        // entry.startTime
    }
}).observe({ type: 'largest-contentful-paint', buffered: true });
```

## 如何优化

-   服务器响应速度
-   JS 和 CSS 阻塞渲染
-   资源加载时间
-   客户端渲染

> 参考 https://web.dev/optimize-lcp/

# FMP

首次绘制有意义内容的时间。整体布局和文字内容全部渲染完成后，即认为完成首次绘制有意义内容。

FMP 衡量了用户看到网页的主要内容时间。

认定页面在加载和渲染过程中最大布局比变动之后的那个绘制时间为 FMP 。

## 如何计算

DOM 结构变化最剧烈的时间点

DOM 结构变化的时间点 通过 MutationObserver API 获得。

MutationObserver 监听每次页面整体的 DOM 变化，触发 MutationObserver 的回调，回调中计算当前 DOM 树的分数，分数变化最剧烈的时刻为 FMP。

FMP 无法绝对精确。

## 如何优化

缩短页面关键路径的渲染时间。

# TTI

从页面加载开始到页面处于完全可交互状态所花费的时间。

-   页面显示有用内容
-   可见元素关联的响应事件已注册
-   响应事件函数可在事件发生后 50ms 内开始执行

TTI 越小，用户可以更早操作页面。

## 如何计算

1. 从起始点开始（FCP 或 FMP），向前搜索不小于 5s 的静默窗口期（无 Longtask，且网络请求数不超过 2）
2. 找到后，从静默窗口期向后搜索到最近的 longtask，longtask 结束时间为 TTI
3. 未找到 Long task 起始时间为 TTI
4. 2、3 得到的 `TTI < DOMContentLoadedEventEnd` ，以 DOMContentLoadedEventEnd 为 TTI

{% asset_img tti.svg tti %}

> 参考 https://web.dev/i18n/zh/tti/
