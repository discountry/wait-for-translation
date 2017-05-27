> 原文：[JavaScript-A First-Class Language At Last](https://hackernoon.com/javascript-a-first-class-language-at-last-209376f69731)
>
> 作者：[Tom Goldenberg](https://hackernoon.com/@tomgoldenberg)
>
> 译者：[LeviDing](http://www.dingxuewen.com/)
>
> 声明：转载请联系本人，并注明出处。

2003年，保罗·格雷厄姆（Paul Graham）在他的一篇文章中提到，他的公司决定使用 Lisp （一门编程语言）。在文章中他将 Lisp 描绘成计算机语言界的法语，它独特、深邃，能够表达出难以描述的事物（就像法语中 je ne sais quoi 所指的）。他指出他的公司与竞争对手相比，优势就在于 Lisp 。

如果 Lisp 像法语，那么现如今的 JavaScript 就像英语一般。尽管二者的语法不太一致，但英语是世界上使用最广泛的语言，JavaScript 是应用最广泛的计算机语言。

然而，JavaScript 仍未得到与其他语言同等的尊重。尽管它在创业公司和大型公司中的使用率持续增长，但 JavaScript 仍被认为是一门没那么重要的语言。大公司的高级工程师们声称它不是一门“真正的”编程语言，许多人并不知道除了操作像素，它还能被用于何处。

作为一名 JavaScript 工程师，我希望更深入地了解公众对这门语言的看法，并观察这些观点在现实当中是有多牢不可破。我发现，有一部分的批评比较有水准，但大多数的批评则是没有意义的。

## 不断增长的生态系统

除了样式效果外，JavaScript 也被越来越多地用于软件开发方面。例如后端任务、Web 服务器以及数据处理。Zeit 首席执行官 Guillermo Rauch 指出，JavaScript 不是人为设计出来的，它是在进化过程中得到的结果。它成型很快，起初只关注一个很小的方面，其余都是市场的力量对这门语言进行的改造。

Rauch 的公司提供一个仅在浏览器和服务器中使用 JavaScript 的开源 Web 框架，事实证明，许多公司都在做同样的事。根据展示公司技术栈信息的网站 [StackShare.io](https://StackShare.io) 上的数据，在后端语言的选取上，相比 Python（4000）或 Java（3900），更多公司选择使用 JavaScript（6000）。这个网站面向的更多的是创业型公司，但它从侧面反映出了关于 JavaScript 的一个不断增长的生态系统。以下是展示不同公司的技术堆栈及其各自的市场份额的维恩图（数据来自 StackShare.io）。

![公司后端编程语言的市场份额](http://oiklhfczu.bkt.clouddn.com/17-5-25/39057977.jpg)

再来看看不同语言程序员的工资情况吧，[Indeed.com](https://indeed.com) 上的数据告诉我们，在美国，Java 程序员的需求量较大，但 JavaScript 程序员的需求量也不低，如下图所示：

![](http://oiklhfczu.bkt.clouddn.com/17-5-25/81047768.jpg)

**对 JavaScript 有正面影响的其他统计数据：**

- 在 Github 上 JavaScript 开源项目的数量最多（比 Java 多出 50%）。
- NodeJS 被评为 StackOverflow 2017 年开发者调查中最受欢迎的框架。
- JavaScript 是 StackOverflow 中最流行的编程语言。

**对 JavaScript 的批评：**

我问过 Oracle 的一位朋友，他们的工程师对 JavaScript 有什么顾虑。他说“由于 JavaScript 是一门动态语言，对于系统编程来说，它并不是一门理想的编程语言”，这种针对 JavaScript 的抱怨非常普遍。JavaScript 的函数可以接受任意类型的参数，但在 Java 中，如果参数不是特定类型就会报错。

```
function doSomething(literallyAnything) { 
    return;
}
```

我又问了另外一位在谷歌工作的朋友，他向我指出 NodeJS 的一些公认的问题，他说，其中的一些问题虽然微乎其微，但使他会认为这个框架还不够成熟。

Rauch 指出，JavaScript 在垃圾回收方面并不是很理想。另一个方面，Java 和 Python 更适合数据科学类的项目，如机器学习和自然语言处理。这可能与这些语言的可用库有关，而非批判 JavaScript 的内在缺陷。学术界对 Java 和 Python 的依赖也助长了这种论调。

上述几位工程师都曾提到，每当讨论编程语言时，经常听到其他工程师贬低 JavaScript。大家对于 JavaScript 用于后端依然心存疑虑，但是大部分敌意似乎又与这门语言及其生态系统的现状无关。

## JavaScript 的现状

JavaScript 在过去 5 年中已经走过很长一段路，早期 JavaScript 的用例一般像 Facebook 的 `Like` 按钮这样，每当用户点击 `Like` 图标，页面不会刷新，但会改变页面状态，这种特性只能通过 JavaScript 在网络上实现。

开发者几年前开始通过 JavaScript 来制作单页面应用程序（SPA）。术语 `single-page` 是指在浏览器中这些应用程序只加载一次代码，所有后续视图都是通过 JavaScript 生成的。反对者认为，用户需要花很长时间才能完成初始下载，在手机上更是长达 20-30 秒！

在过去的两年中，将 JavaScript 代码发送到浏览器的技术已经大大改善（参见：webpack）。这可以解决JavaScript Web 应用的缓慢的加载速度，提升性能并提供更好的用户交互体验。这是目前 Web 开发领域最先进的技术。

伴随着技术进步，出现了新的 JavaScript 范式。状态管理库将计算机科学原理应用于用户交互，JavaScript 工程师的门槛变得更高。

在这些变化的背景下，对于发展初期的公司来说，使用 JavaScript 作为后端语言非常有意义，如果您已拥有优秀的前端 JS 攻城师，此举可以让它们更轻松地协作，审核和共享代码。

尽管 JavaScript 最初只是一门浏览器中的语言，但在计算机科学的各个方面 Web、移动端、物联网和后端服务中，它都变得更加普及。工程师们不会因为他们对语言的过时认知而忽视它。其实 JavaScript 一直是一门“真正的”编程语言，只不过这种声明会比其他任何事情更容易被误解。

## 总结

从这些观察结果可以看出，JavaScript 已经达到以下这些成为一流编程语言的标准：

- 被创业公司和大型公司用作后端服务框架（NodeJS）
- 有一个蓬勃发展的开源社区（在 GitHub 上最活跃）
- 作为一门专业技能，有大量的招聘需求中要求掌握 JavaScript 的知识（Indee.com）

最后，一家公司决定贯彻某种技术方案都是需要进行妥协的。我们在 Commandiv 这款产品中就同时使用JavaScript 作为前端和后端的变成语言，但这并不适合所有人，我们这么决定，有一部分原因我们熟悉JavaScript 这门语言。为了在创业初期快速启动，请使用你最熟悉的工具。

也就是说，我认为质疑 JavaScript 是否是一种“真正的”编程语言的时代已经过去。JavaScript 前方的路还有很长，但是其应用率和改进速度使我对其前进的道路充满信心。


欢迎大家在评论区留下你的想法和感受！
