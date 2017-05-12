# 用 原生Javascript 创建带动画的固顶导航菜单

> 原文链接: [Create an Animated Sticky Navigation Menu with Vanilla JavaScript](https://www.sitepoint.com/animated-sticky-navigation-menu-javascript/)
>
>作者: [Albert Senghor](https://www.sitepoint.com/author/asenghor/)
>
>译者: [codearvin](https://codearvin.github.io)
>
>转载请注明出处

当我们在网页中加入一个导航菜单的时候，需要考虑很多因素。如何确定它的位置？如何定义样式？还需要保证它具有良好的响应性。又或者你想为它添加一些炫酷的动画。这时你可能会对 jQuery 感兴趣，因为它会帮你做大部分的事情。但并不必这样！事实上只用几行代码就可以解决这件事情。

在这篇文章中，我会向你们展示如何用 原生Javascript，CSS 和 HTML 去创建一个带动画的固顶导航菜单。最后它会在页面向下滚动时上滑隐藏起来，页面向上滚动时下滑回到页面上(以一种时髦的透明效果)。许多流行的网站采用了这种技术，例如[Medium](https://medium.com/sitepoint/what-is-the-best-book-for-learning-javascript-973de4e2ef9e)和[Hacker Noon](https://hackernoon.com/how-to-build-a-todo-app-using-react-redux-and-webpack-1aa99dc2f45c)。

相信阅读完你就会在你的设计中尝试了，希望能有很好的效果。如果你等不及可以直接看文章末尾的demo

## 固顶导航菜单：基本HTML结构

下面是我们将要使用的 HTML 骨架，很普通的代码。

```html
<div class="container">
  <div class="banner-wrapper">
    <div class="banner">
      <div class="top">
        <!-- Navbar top row-->
      </div>
      <div class="nav">
        <!-- Links for navigation-->
      </div>
    </div>
  </div>

  <div class="content">
    <!-- Awesome content here -->
  </div>
</div>
```

## 添加一点样式

让我们为主要的元素添加一些样式。

### 主容器

我们需要移除浏览器的默认样式并设置主容器的 width 为100%。

```css
*{
  box-sizing:border-box;
  padding: 0;
  margin: 0;
}

.container{
  width: 100%;
}
```

### 横幅容器

它包裹在导航菜单的外面，位置是固定的，当页面垂直滚动时会上滑隐藏或下滑展现导航菜单。我们给它添加一个  `z-index` 属性来保证它会展现在页面的最顶层。

```css
.banner-wrapper {
  z-index: 4;
  transition: all 300ms ease-in-out;
  position: fixed;
  width: 100%;
}
```

### 横幅部分

它包含导航菜单。当页面上下滚动时，CSS 的 `transition` 属性会让位置和背景颜色的改变带有动画效果。

```css
.banner {
  height: 77px;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  background: rgba(162, 197, 35, 1);
  transition: all 300ms ease-in-out;
}
```

### 内容部分

这部分包含背景图片和文本。我们会在文章的后面部分为它添加视差效果。

```css
.content {
  background: url(https://unsplash.it/1400/1400?image=699) center no-repeat;
  background-size: cover;
  padding-top: 100%;
}
```

## 让菜单动起来

我们要做的第一件事是给滚动事件添加一个事件处理器，以便在用户滚动页面的时候相应的展示或隐藏菜单。我们把所有代码用即时执行函数[IIFE](https://www.sitepoint.com/demystifying-javascript-closures-callbacks-iifes/#immediately-invoked-function-expressions-iifes)封装起来以免与同一页面的其他代码发生冲突。

```javascript
(() => {
  'use strict';

  const handler = () => {
    //DOM manipulation code here
  };

  window.addEventListener('scroll', handler, false);
})();
```

### 设置一些初始变量

`refOffset` 表示用户向下滚动页面的距离。这个值会在页面加载时初始化为 `0`。`bannerHeight` 来储存菜单的高度。同样我们需要 `.banner-wrapper` 和 `.banner` 两个DOM元素的引用。

```javascript
let refOffset = 0;
let visible = true;
const bannerHeight = 77;

const bannerWrapper = document.querySelector('.banner-wrapper');
const banner = document.querySelector('.banner');
```

### 确定滚动方向

接下来我们需要确定滚动的方向以便我们能相应的展示和隐藏菜单。

我们以 `newOffset` 变量开始。在页面加载时它会被设置为 [window.scrollY](https://developer.mozilla.org/en-US/docs/Web/API/Window/scrollY) 的值 —— 当前页面垂直滚动的像素个数(初始为 `0`)。当用户滚动时，`newOffset 会相应的增加或减少。如果 `newOffset` 大于 `bannerHeight`，这时我们就知道菜单已经滚动到视图外了。

```javascript
const newOffset = window.scrollY;

if (newOffset > bannerHeight) {
  // Menu out of view
} else {
  // Menu in view
}
```

向下滚动会使 `newOffset` 大于 `refOffset` ，导航菜单应该上滑隐藏。向上滚动会使 `newOffset` 小于 `refOffset` ，导航菜单应该以透明效果滑回视图。所以我们需要不断通过 `newOffset` 来更新 `refOffset` ，来保持对用户滚动的距离的追踪。

```javascript
if (newOffset > bannerHeight) {
  // Menu out of view
  if(newOffset > refOffset) {
    // Hide the menu
  } else if (newOffset < refOffset) {
    // Slide menu back down
  }

  refOffset = newOffset;
} else {
  // Menu in view
}
```

## 菜单动画

最后，添加 `animateIn` 和 `animateOut` 两个方法，分别用来展示和隐藏菜单。我们也应该保证当菜单位于页面顶部的时候移除它的透明效果。

```javascript
if (newOffset > bannerHeight) {
  if (newOffset > refOffset) {
    animateOut();
  } else {
    animateIn();
  }

  refOffset = newOffset;
} else {
  banner.style.backgroundColor = 'rgba(162, 197, 35, 1)';
}
```

下面是动画效果的函数代码

```javascript
function animateOut() {
  bannerWrapper.style.msTransform = `translateY(-${bannerHeight}px)`;
  bannerWrapper.style.transform = `translateY(-${bannerHeight}px)`;
  bannerWrapper.style.webkitTransform = `translateY(-${bannerHeight}px)`;
  bannerWrapper.style.MozTransform = `translateY(-${bannerHeight}px)`;
  banner.style.background = 'rgba(162, 197, 35, 0.6)';
}

function animateIn() {
  bannerWrapper.style.transform = 'translateY(0px)';
  bannerWrapper.style.msTransform = 'translateY(0px)';
  bannerWrapper.style.webkitTransform = 'translateY(0px)';
  bannerWrapper.style.MozTransform = 'translateY(0px)';
  banner.style.background = 'rgba(162, 197, 35, 0.6)';
}
```

你可能考虑使用 CSS 的 class 来达到同样的效果，你可以修改相应的代码来实现。在本篇文章中，我选择了使用 JavaScript。

## 演示

这里是导航菜单的演示效果:

[Create an Animated Sticky Navigation Menu with Vanilla JavaScript](http://codepen.io/SitePoint/pen/ZKJVdw/)

![nav.png](https://ooo.0o0.ooo/2017/05/09/5911cd339cbe9.png)

## 总结

本文描述了如何仅仅用几行原生 JavaScript 代码(不需要 jQuery)来设计一个带动画的导航菜单。菜单会在页面向下滚动时上滑隐藏，页面向上滚动时下滑回视图并带有透明效果。通过监听垂直方向的滚动并改变 DOM 元素相应的 CSS 属性来实现功能。这样的定制化解决方案会给你更大的自由空间，让你可以根据自己的需求灵活的进行设计。
