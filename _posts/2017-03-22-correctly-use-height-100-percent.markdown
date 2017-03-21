---
layout: post
title:  "正确设置 CSS 的 height: 100%"
date:   2017-03-22 01:42:44 +0800
categories: CSS
tags: CSS 布局
---

我想要这样一种布局：顶部和底部固定高度，中间部分随窗口大小自适应。效果大概是这样：![想要实现的效果](/assets/correctly-use-height-100-percent/target.png)

我的思路很简单，将顶部和底部的 `<header>` 和 `<footer>` 设置为固定高度，将中间的 `<main>` 的高度设为 `100%`。`<main>` 使用百分比高度时，参照了父元素容器的高度，所以我们要将父元素 `<body>` 的高度也设置为 `100%`。具体代码是这样的：

{% highlight html %}
<body>
  <header></header>
  <main></main>
  <footer></footer>
</body>
{% endhighlight %}

{% highlight css %}
html, body { height: 100%; }

header, footer {
  height: 60px;
  width: 100%;
}

main {
  height: 100%;
  width: 100%;
}
{% endhighlight %}

但是我实际得到的页面是这样的：![实际的效果](/assets/correctly-use-height-100-percent/actual.png)

其实这个页面显示得不完全，向下滚动还有内容。

如果用阴影标注被隐藏的部分，页面是这个样子的：![整个页面被拉长了](/assets/correctly-use-height-100-percent/analyse.png)

我们来看看究竟发生了什么。实际上，最终页面上 `<main>` 部分（红色区域）的高度，刚好是窗口的高度。它和窗口等高，头尾再加上两段固定的高度，整个页面的高度自然就超过窗口的高度了。

## 使用绝对定位解决

将顶部和底部元素设置成绝对定位可以解决问题。有几个注意点：

- 要把它们的父元素设为 `position: relative`
- 为了给自适应部分腾出空间，给父元素使用 padding
- 父元素的 `height` 指的是 content 的高度，不包括 padding。要设置 `box-sizing: border-box`，让 content 和 padding 加起来的高度是 `height: 100%`
- 顶部和底部的元素需要 `position: absolute`，并分别使用 `top` 和 `bottom`

修改后的代码是这样的：

{% highlight css %}
html, body { height: 100%; }

body {
  position: relative;
  padding: 60px 0 60px 0;
  box-sizing: border-box;
}

header, footer {
  position: absolute;
  height: 60px;
  width: 100%;
}

header { top: 0; }
footer { bottom: 0; }

main {
  height: 100%;
  width: 100%;
}
{% endhighlight %}

## 使用 `calc()` 解决

还有一种更加方便的解决方法。之前的问题出在，错误地将 `<main>` 的高度设置成了窗口的高度，而实际需要的高度，其实是（窗口的高度 - 顶部的高度 - 底部的高度）。

我想到了 CSS 的 `calc()` 函数，用它可以计算所需要的尺寸。注意，`calc()` 函数的单位可以混合，我们就可以用类似 `calc(100% - 120px)` 解决问题。用这个方法，只需修改一行代码：

{% highlight css %}
/* ... */
main {
  height: calc(100% - 120px);
  width: 100%;
}
{% endhighlight %}