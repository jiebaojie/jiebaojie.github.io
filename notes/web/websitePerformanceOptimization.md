---
layout: post
notes: true
subtitle: "大型网站性能优化实践（从前端、网络、CDN到后端、大促的全链路性能优化详解）"
comments: false
author: "周涛明 张荣华 张新兵"
date: 2019-3-9 12:00:00

---

![](/img/notes/web/websitePerformanceOptimization/website_performance_optimization.jpg)

*   目录
{:toc }

# 第1章 基于用户体验的性能优化要素

## 1.1 页面用户体验的要素介绍

几个优秀Market的要素：

*	商品丰富、质量由保证等
*	信誉好、品牌影响力大等
*	价廉物美等其他因素

衡量网站性能方面的用户体验要素：

*	白屏
*	首屏
*	页面整体加载
*	页面可交互
*	功能交互响应

## 1.2 白屏实践

### 1.2.1 白屏时间的重要性

简单理解，即用户打开浏览器输入网站的URL后，从屏幕空白到第一个画面出来的时间，这个时间的长短将直接决定网站页面给用户的第一印象。

*	StartRender
*	FirstPaint

重要性：页面渲染的时间越短，用户等待的时间就越短，用户感知到的页面速度就越快，这样可以大大提高用户体验，减少新用户的跳出，提高留存率。反之，过长的等待时间，会让用户变得烦躁，更轻易跳出或者关闭这个网站。

### 1.2.2 白屏过程详解

![](/img/notes/web/websitePerformanceOptimization/white_screen.png)

每个步骤：

#### 1. DNS Lookup

DNS Lookup即浏览器从DNS服务器中进行域名查询。浏览器解析域名拿到对应的IP地址后，才能和服务器进行通信。通常浏览器在加载页面的过程中，会进行很多次DNS Lookup操作，其中包括页面本身的域名查询，以及浏览器在解析页面HTML代码过程中，需要加载JS、CSS、Image等资源产生的解析。

通常在页面加载过程中会产生下列域名解析：

*	www.example.com，页面URL本身。
*	style.example.com，JS、CSS等静态资源请求。
*	img.example.com，静态图片请求。
*	ajax.example.com，各种站内的Ajax请求。
*	*.google.com，其他比如Google的站外请求。

DNS解析速度的优化策略有很多：

*	DNS缓存优化
*	DNS预加载策略
*	页面中资源的域名的合理分配
*	稳定可靠的DNS服务器

#### 2. 建立TCP请求连接

浏览器针对各个域名进行DNS Lookup得到IP地址后，就开始建立TCP请求连接的过程。

两个要素理解TCP连接及传输的过程：

*	网络传输链路建立
*	数据传输确认

优化思路：

*	基于不同的网络环境，优化数据包的大小，以减少数据因传输丢失或者被破坏产生的重传，从而提高传输效率
*	网络传输链路的优化，对于大部分企业来说，是需要很大投入的。



