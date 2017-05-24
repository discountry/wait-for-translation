# 最终的Angular-CLI参考指南

>原文链接：[The Ultimate Angular CLI Reference Guide](https://www.sitepoint.com/ultimate-angular-cli-reference/)
>
>作者：[Jurgen Van de Moere](https://www.sitepoint.com/author/jvandemoere/)
>
>译者：[Liubara](https://Liubara.github.io)
>
>本文已获得作者授权，转载请注明出处。

2017年4月25日： Angular CLI v1.0在3月24日发布。这篇文章也已经根据最新版本的变化做了相应的更新。如果你想在早期版本的Angular项目中增加Angular-CLI v1.0的特性，请查看[Angular CLI v1.0迁移指南](https://github.com/angular/angular-cli/wiki/stories-1.0-update)

2017年2月17日： 截止2017年2月9日，`ng deploy`命名从Angular CLI的语法中被移除，点击[这里](https://github.com/angular/angular-cli/pull/4385)查看更多。

2017年1月27日： 截止2017年1月27日，官方推荐使用*AngularJS*作为Angular 1.x和2+版本的名字。这篇文章也已经更新来响应[官方指南](http://angularjs.blogspot.be/2017/01/branding-guidelines-for-angular-and.html)

*****
以下是初步计划的系列文章，将从4个部分来展示怎么制作一个Angular TODO应用。
1 第0部分--[最终的Angular-CLI参考指南]()
2 第1部分--[启动并运行我们的第一个版本的Todo应用程序]()
3 第2部分--[创建单独的组件来显示todo的列表和一个todo]()
4 第3部分--[更新Todo服务以与REST API通信]()
5 第4部分--[使用组件路由器链接到不同的组件]()
(译者会翻译接下来的部分，敬请期待)

*****
在2016年9月15日，发布了[Angular的最终版本](http://angularjs.blogspot.be/2016/09/angular2-final.html)

虽然[AngularJS](https://angularjs.org/) 1.x版本仅仅局限于一个框架，但是[Angular](https://angular.io/)已经成为一个野心勃勃的平台，它允许我们在所有平台上开发快速和可伸缩的应用程序，如web、移动端web、本地移动，甚至本地桌面。

由于这个从框架到平台的过渡，使得工具比以前更加重要。然而，设置和配置工具往往没有那么简单。为了确保Angular的开发者以及尽可能少的摩擦来关注构建应用程序，Angular团队在为开发者提供更高质量的开发工具集上做了很多努力。

这些工具集的一部分是由大量的IDE和编辑器紧密集成的。工具集的另一部分就是[Angular CLI](https://cli.angular.io/)

在这篇文章中我们将会学习到Angular CLI是什么，它可以为我们做什么和它在后台怎么施展它的魔力。即便你曾经使用过Angular CLI，这篇文章也可以为你提供一个参考来帮助你更好地理解它是怎样工作的。

