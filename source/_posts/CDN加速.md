---
title: CDN加速
tags:
- Web
- cdn加速
- katex
- Blog
categories:
- Blog
date: 2019-07-14 19:19:04
top:
mathjax: true
---



{% cq %}

内容分发网络（英语：Content Delivery Network或Content Distribution Network，缩写：CDN）是指一种透过互联网互相连接的计算机网络系统，利用最靠近每位用户的服务器，更快、更可靠地将音乐、图片、影片、应用程序及其他文件发送给用户，来提供高性能、可扩展性及低成本的网络内容传递给用户。

{% endcq %}



<!-- more -->

<br>

# **优点**

------

- 内容分发网络的总承载量可以比单一骨干最大的带宽还要大。这使得内容分发网络可以承载的用户数量比起传统单一服务器多。也就是说，若把有100Gbps处理能力的服务器放在只有10Gbps带宽的数据中心，则亦只能发挥出10Gbps的承载量。但如果放到十个有10Gbps的地点，整个系统的承载量就可以到10×10Gbps。同时，**将服务器放到不同地点，可以减少互连的流量，进而降低带宽成本。**
- 对于TCP传输而言，TCP的速度（throughput）会受到延迟时间（latency）与数据包漏失率（packet loss）影响。为了改善这些负面因素，**内容分发网络通常会指派较近、较顺畅的服务器节点将数据传输给用户**。虽然距离并不是绝对因素，但这么做可以尽可能提高性能，用户将会觉得比较顺畅。这使得一些比较高带宽的应用（传输高清画质的影片）更容易推动。
- 内容分发网络另外一个好处在于有**异地备援**。当某个服务器故障时，系统将会调用其他邻近地区的服务器服务，进而提供接近100%的可靠度。
- 除此之外，内容分发网络**提供给服务提供者更多的控制权**。提供服务的人可以针对客户、地区，或是其他因子调整。

## **公用开源cdn哪家强?**

我们所有的主题渲染highlight.js以及数学公式katex.js都是都可以在[**jsdelivr**](https://www.jsdelivr.com/)上找到,而且也非常推荐使用此公共cdn,对中国有加速，支持npm,github ,WordPress包，完全免费无限制，全球都很快。

<br>

# **代码主题渲染**

------

> 访问其官方网站[**highlight.js**](https://highlightjs.org/)进行探索  


## **操作步骤:**

1. 只需要把以下代码放在你的博客源码的head中,他会自动寻找<\pre><\code>中的代码并且去渲染  

```js
<link rel="stylesheet"
      href="//cdn.jsdelivr.net/gh/highlightjs/cdn-release@9.15.8/build/styles/default.min.css">
<script src="//cdn.jsdelivr.net/gh/highlightjs/cdn-release@9.15.8/build/highlight.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```

2. 如果你不喜欢默认的主题,你可以把代码中的**default**关键字替换成你喜欢的主题的名字,所有主题你都可以在官网上查看渲染不同语音的不同主题效果,如图所示:  

![](file://C:/Users/Administrator/Desktop/googlesync/Gridea/post-images/1561820901909.png)

## **此处写一下我在操作遇到的坑:**  

- 直接在官网上找链接,不要使用jsdelivr网站上的npm方式的链接方式(很气愤😤,不知道为什么渲染不成功,搞了好长时间)  
- 一定要看自己的博客有么有<\pre>,如果没有需要自己设置  

## **渲染效果**

```java
/**
 * @author John Smith <john.smith@example.com>
*/
package l2f.gameserver.model;

public abstract class L2Char extends L2Object {
  public static final Short ERROR = 0x0001;

  public void moveTo(int x, int y, int z) {
    _ai = null;
    log("Should not be called");
    if (1 > 5) { // wtf!?
      return;
    }
  }
}
```

<br>

# 数学公式

------

## **操作步骤:**

1. 访问其官网[**katex**](https://katex.org/)可自行探索  ,数学公式的渲染如同代码主题的渲染操作,复制以下代码添加到head中:  

```js
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/katex.min.css" integrity="sha384-yFRtMMDnQtDRO8rLpMIKrtPCD5jdktao2TV19YiZYWMDkUR5GQZR/NOVTdquEx1j" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/katex.min.js" integrity="sha384-9Nhn55MVVN0/4OFx7EE5kpFBPsEMZxKTCnA+4fqDmg12eCTqGi6+BB2LjY8brQxJ" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/contrib/auto-render.min.js" integrity="sha384-kWPLUVMOks5AQFrykwIup5lo0m3iMkkHrD0uJ4H5cjeGihAutqP0yW0J6dpFiVkI" crossorigin="anonymous"
    onload="renderMathInElement(document.body);"></script>
```

> 此代码不会在页面加载时就开始渲染,而是出现关键符号是才会加载,所以不会影响页面加载速度

2. 在写公式的时候按照markdown格式处理你要显示的数学公式即可.  

## **转换效果**

*转换前:*  

```
f{x} = \int_{-\infty}^\infty
\hat f(\xi),e^{2 \pi i \xi x}
\,d\xi
```

*转换后:*  
$$
f{x} = \int_{-\infty}^\infty
    \hat f(\xi),e^{2 \pi i \xi x}
    \,d\xi
$$

------

还是要学好基础啊,走了许多弯路