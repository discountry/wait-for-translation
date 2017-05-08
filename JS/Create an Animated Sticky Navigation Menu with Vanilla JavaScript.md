# 用 原生Javascript 创建带动画的固顶导航菜单

> 原文链接: [Create an Animated Sticky Navigation Menu with Vanilla JavaScript](https://www.sitepoint.com/animated-sticky-navigation-menu-javascript/)
作者: [Albert Senghor](https://www.sitepoint.com/author/asenghor/)
译者: [codearvin](https://codearvin.github.io)
转载请注明出处

当我们在网页中加入一个导航菜单的时候，需要考虑很多因素。例如：位置、样式、响应式。又或者你可能想为它添加一些动画(当然这会更炫酷)。这时你可能有兴趣去使用jQuery这个插件，它会帮你做大部分的事情。但并不必这样！事实上你可以只用几行代码来创建出你自己的解决方案。

在这篇文章中，我会向你们展示如何用 原生Javascript，CSS 和 HTML 去创建一个带动画的固顶导航菜单。最后的成品会在页面向下滚动时上滑隐藏，页面向上滚动时下滑回到视图(以一种时髦的透明效果)。这种技术被许多流行的网站采用，例如[Medium](https://medium.com/sitepoint/what-is-the-best-book-for-learning-javascript-973de4e2ef9e)和[Hacker Noon](https://hackernoon.com/how-to-build-a-todo-app-using-react-redux-and-webpack-1aa99dc2f45c)。

阅读完这篇文章后你应该就会准备好在你的设计中应用这项技术了，希望能有很好的效果。如果你等不及可以直接看[文章末尾的demo](#show)

## 固顶导航菜单：基本HTML结构

下面是我们将要使用的HTML骨架代码。这里并没有什么激动人心的地方。
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

让我们为一些主要的元素添加一些样式。

### 主容器

我们需要移除任何固有的浏览器样式并且设置我们主容器的width为100%。
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

这是一个包裹在导航菜单外侧的包装容器。它的位置是固定的，当页面垂直滚动的时候会上滑隐藏或下滑展现导航菜单。我们会给它一个`z-index`属性来保证它会展现在页面的最顶层。

```css
.banner-wrapper {
  z-index: 4;
  transition: all 300ms ease-in-out;
  position: fixed;
  width: 100%;
}
```

### 横幅部分

它包含导航菜单。当页面上下滚动时，位置和背景颜色的变化通过CSS的`transtion`属性指定。

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

这部分包含背景图片和文本。我们会在接下来的文章中为这部分页面添加视差效果。
```css
.content {
  background: url(https://unsplash.it/1400/1400?image=699) center no-repeat;
  background-size: cover;
  padding-top: 100%;
}
```

## 让菜单动起来

我们要做的第一件事是给滚动事件添加一个事件处理器，以便我们在用户滚动页面的时候展示或隐藏菜单。我们也会把所有代码用[IIFE](https://www.sitepoint.com/demystifying-javascript-closures-callbacks-iifes/#immediately-invoked-function-expressions-iifes)封装起来以免与同一页面的其他代码发生冲突。
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

我们用`refoffset`变量表示用户向下滚动页面的距离。这个值会在页面加载时初始化为`0`。我们用`bannerHeight`变量来储存菜单的高度。同样我们需要`.banner-wrapper`和`.banner`两个DOM元素的引用。
```javascript
let refOffset = 0;
let visible = true;
const bannerHeight = 77;

const bannerWrapper = document.querySelector('.banner-wrapper');
const banner = document.querySelector('.banner');
```

### 确定滚动方向

接下来我们需要确定滚动的方向以便我们能相应的展示和隐藏菜单。

我们以`newOffset`变量开始。在页面加载的时候这个变量会被设置为[window.scrollY](https://developer.mozilla.org/en-US/docs/Web/API/Window/scrollY)的值 —— 当前页面垂直滚动的像素个数(初始为`0`)。当用户滚动时，`newOffset`的值会相应的增加或减少。如果`newOffset`的值大于`bannerHeight`的值，这时我们就知道菜单已经滚动到视图外了。
```javascript
const newOffset = window.scrollY;

if (newOffset > bannerHeight) {
  // Menu out of view
} else {
  // Menu in view
}
```

向下滚动会使`newOffset`的值大于`refOffset`的值，这时导航菜单应该上滑隐藏。向上滚动会使`newOffset`的值小于`refOffset`的值，这时导航菜单应该以透明效果滑回视图。通过这样的分析，我们需要不断通过`newOffset`来更新`refOffset`的值，来保持对用户滚动的距离的追踪。
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

最后，让我们添加`animateIn`和`animateOut`两个方法来展示和隐藏菜单。我们也应该保证当菜单位于页面顶部的时候移除透明效果。
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

你可能考虑使用CSS的class，你可以相应的修改代码。然而在这种情况下，我选择了使用JavaScript。

## 演示

<span id="show">这里是导航菜单的演示效果</span>
<p data-height="265" data-theme-id="0" data-slug-hash="ZKJVdw" data-default-tab="css,result" data-user="SitePoint" data-embed-version="2" data-pen-title="Create an Animated Sticky Navigation Menu with Vanilla JavaScript" class="codepen">See the Pen <a href="http://codepen.io/SitePoint/pen/ZKJVdw/">Create an Animated Sticky Navigation Menu with Vanilla JavaScript</a> by SitePoint (<a href="http://codepen.io/SitePoint">@SitePoint</a>) on <a href="http://codepen.io">CodePen</a>.</p>

## 总结

这篇文章描述了如何仅仅用几行JavaScript代码(不需要jQuery)设计一个带动画的导航菜单。菜单会在页面向下滚动时上滑隐藏，页面向上滚动时下滑回视图并带有透明效果。这是通过监听垂直方向的滚动并在需要的时候把 CSS transformatinos 应用到DOM元素上来完成的。这样的定制化解决方案会给你更大的自由空间，让你可以根据自己的需求和规定灵活的进行设计。

这篇文章由[Vildan Softic](https://www.sitepoint.com/author/vildansoftic)审阅。感谢所有致力于让SitePoint内容完美的审稿人!
