# 在浏览器中使用 ECMAScript Modules

> 原文链接：[ECMAScript Modules in Browsers](https://jakearchibald.com/2017/es-modules-in-browsers/)
>
> 作者：[Jake Archibald](https://twitter.com/jaffathecake)
>
> 译者：张弈
>
> 校对：[余博伦](https://www.zhihu.com/people/yubolun)
>
> 转载请注明出处。
 
ES Modules 正在登陆各大浏览器！下列浏览器版本均已支持：

* Safari 10.1。
* Chrome Canary 60 – 在 `chrome:flags` 设置页面中 the Experimental Web Platform flag的相关参数设置。
* Firefox 54 – 在 `about:config` 设置页面中 `dom.moduleScripts.enabled`的相关参数设置。
* Edge 15 – 在 `about:flags`页面中the Experimental JavaScript Features的相关参数设置。

```javascript
<script type="module">
  import {addTextToBody} from './utils.js';

  addTextToBody('Modules are pretty cool.');
</script>
```

```javascript
// utils.js
export function addTextToBody(text) {
  const div = document.createElement('div');
  div.textContent = text;
  document.body.appendChild(div);
}
```
 
[演示页面](https://cdn.rawgit.com/jakearchibald/a298d5af601982c338186cd355e624a8/raw/aaa2cbee9a5810d14b01ae965e52ecb9b2965a44/)

你需要做的只是为 `script` 标签添加 `type=module` 属性, 这样浏览器就会将你引入或编写的脚本视为 ECMAScript module 处理。

网络上已经有了一些[介绍 module 的精彩文章](https://ponyfoo.com/articles/es6-modules-in-depth), 但是我仍然想要分享一些读到且尝试过的，浏览器相关的 modules 特性：

## `import` 引入路径支持还不完善

```javascript
// 支持如下路径格式:
import {foo} from 'https://jakearchibald.com/utils/bar.js';
import {foo} from '/utils/bar.js';
import {foo} from './bar.js';
import {foo} from '../bar.js';

// 下列格式不支持:
import {foo} from 'bar.js';
import {foo} from 'utils/bar.js';
```

引入模块的路径必须符合以下条件之一：

* 一个完整的绝对路径，可以通过 `new URL(moduleSpecifier)` 不报错。
* 以 / 开头。
* 以 ./ 开头。
* 以 ../ 开头。

其他的路径格式可能在将来会有支持，例如导入内置模块。

## 引入 nomodule 属性向后兼容

```javascript
<script type="module" src="module.js"></script>
<script nomodule src="fallback.js"></script>
```

[演示页面](https://cdn.rawgit.com/jakearchibald/6110fb6df717ebca44c2e40814cc12af/raw/7fc79ed89199c2512a4579c9a3ba19f72c219bd8/)

支持 `type=module` 属性的浏览器可以忽略附带 `nomodule`属性的脚本标签。在上面这个示例中，如果浏览器支持载入模块则会加载前一个标签的脚本，而其他不支持模块的浏览器则会载入后一个名为 `fallback.js` 的脚本。

### 浏览器遗留问题

* ~~Firefox不支持~~ `nomodule` ([~~issue~~](https://bugzilla.mozilla.org/show_bug.cgi?id=1330900)). 在其nightly版本中已经被修复!
* Edge 不支持 `nomodule` ([issue](https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/10525830/))。
* Safari 10.1 不支持 `nomodule`, 但它在最新的技术预览中修复了。 10.1版本中有一个 [很棒的支持方案](https://gist.github.com/samthor/64b114e4a4f539915a95b91ffd340acc)。

## 默认延迟

```javascript
<!-- This script will execute after… -->
<script type="module" src="1.js"></script>

<!-- …this script… -->
<script src="2.js"></script>

<!-- …but before this script. -->
<script defer src="3.js"></script>
```

[演示页面](https://cdn.rawgit.com/jakearchibald/d6808ea2665f8b3994380160dc2c0bc1/raw/c0a194aa70dda1339c960c6f05b2e16988ee66ac/)顺序应该是 `2.js, 1.js, 3.js`。

脚本阻塞HTML加载是非常糟糕的情况。 你可以为脚本标签添加一个 `defer` 属性来防止页面阻塞发生，同时也会使得这段脚本在文档完成解析之后才会运行。默认的，module 类型的脚本的加载也类似于添加 `defer` 属性的脚本 - 毕竟我们毫无理由让这些脚本阻塞页面的加载。

module 脚本与添加 `defer` 属性的脚本使用相同的执行队列。

## 内联脚本同样会延迟加载

```javascript
<!-- This script will execute after… -->
<script type="module">
  addTextToBody("Inline module executed");
</script>

<!-- …this script… -->
<script src="1.js"></script>

<!-- …and this script… -->
<script defer>
  addTextToBody("Inline script executed");
</script>

<!-- …but before this script. -->
<script defer src="2.js"></script>
```

[演示页面](https://cdn.rawgit.com/jakearchibald/7026f72c0675898196f7669699e3231e/raw/fc7521aabd9485f30dbd5189b407313cd350cf2b/)加载顺序为 `1.js`, 内联脚本, 内联模块, `2.js`。

通常内联脚本会忽略 `defer` 属性，与此同时，不管内联的 module 脚本是否有引入什么，都会被延迟加载。

## 外部和内联 module 的 `async` 如何工作

```javascript
<!-- This executes as soon as its imports have fetched -->
<script async type="module">
  import {addTextToBody} from './utils.js';

  addTextToBody('Inline module executed.');
</script>

<!-- This executes as soon as it & its imports have fetched -->
<script async type="module" src="1.js"></script>
```

[演示页面](https://module-script-tests-rgjnxtrtqq.now.sh/async) 下载的较快的脚本会先运行。

`async` 属性可以使脚本不阻塞HTML加载并且在下载完成后立即运行。与常规脚本不同，`async`也适用于内联 module.

我们使用 `async`时，脚本可能不会按照它在DOM中的书写的顺序执行。

### 浏览器遗留问题

* Firefox 在内联模块脚本上不支持 `async` ([issue](https://bugzilla.mozilla.org/show_bug.cgi?id=1361369))。

## Module 只执行一次

```javascript
<!-- 1.js only executes once -->
<script type="module" src="1.js"></script>
<script type="module" src="1.js"></script>
<script type="module">
  import "./1.js";
</script>

<!-- Whereas normal scripts execute multiple times -->
<script src="2.js"></script>
<script src="2.js"></script>
```

[演示页面](https://cdn.rawgit.com/jakearchibald/f7f6d37ef1b4d8a4f908f3e80d50f4fe/raw/1fcedde007a2b90049a7ea438781aebe69e22762/)

如果你对ES Modules比较了解，你应该知道我们可以多次使用 `import` 导入，但 module 只会执行一次。这一特性同样适用于HTML中的 module 脚本，从一个 URL 引入的 module 脚本在一个页面中只会执行一次。

### 浏览器遗留问题

*Edge 可以多次执行模块 ([issue](https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/11865922/)).

## module 均以跨域资源共享方式载入

```javascript
<!-- This will not execute, as it fails a CORS check -->
<script type="module" src="https://….now.sh/no-cors"></script>

<!-- This will not execute, as one of its imports fails a CORS check -->
<script type="module">
  import 'https://….now.sh/no-cors';

  addTextToBody("This will not execute.");
</script>

<!-- This will execute as it passes CORS checks -->
<script type="module" src="https://….now.sh/cors"></script>
```

[演示页面](https://cdn.rawgit.com/jakearchibald/2b8d4bc7624ca6a2c7f3c35f6e17fe2d/raw/fe04e60b0b7021f261e79b8ef28b0ccd132c1585/)

与常规脚本不同，module 脚本都是通过跨域资源共享的方式载入的。这意味着载入的 module 脚本必须返回正确的CORS头，例如 `Access-Control-Allow-Origin: *` .

### 浏览器遗留问题

* Firefox 无法加载演示页面 ([issue](https://bugzilla.mozilla.org/show_bug.cgi?id=1361373))。
* Edge 加载演示页面无需CORS头 ([issue[(https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/11865934/))。

## 不带凭据

```javascript
<!-- Fetched with credentials (cookies etc) -->
<script src="1.js"></script>

<!-- Fetched without credentials -->
<script type="module" src="1.js"></script>

<!-- Fetched with credentials -->
<script type="module" crossorigin src="1.js?"></script>

<!-- Fetched without credentials -->
<script type="module" crossorigin src="https://other-origin/1.js"></script>

<!-- Fetched with credentials-->
<script type="module" crossorigin="use-credentials" src="https://other-origin/1.js?"></script>
```

[演示页面](https://module-script-tests-zoelmqooyv.now.sh/cookie-page)

如果是相同的来源的请求，大多数基于CORS的API都会发送凭据（cookie等），但 `fetch（）` 和 module 脚本是例外 —— 默认是不携带凭据的。

你可以通过添加 `crossorigin` 属性，来使同源 module 携带凭据（对此我感到有点奇怪，[在规范讨论中我已经提出质疑](https://github.com/whatwg/html/issues/2557)）。如果是非同源的情况，你可以设置 `crossorigin="use-credentials"` 属性。注意，请求的来源必须有包涵 `Access-Control-Allow-Credentials: true`头部信息的响应。

另外，还有一个与“module 只执行一次”规则有关的疑难杂症。每个 module 都是根据引入它的URL来标记的，因此如果你先请求来不带凭据的模块，再使用携带凭据的请求方式依然会获得之前的无凭据的 module，这就是为什么我在上面的示例中，在URL后面加上了“?”来确保载入 module 的唯一性。

### 浏览器遗留问题

* Chrome 要求有凭据且相同来源的模块 ([issue](https://bugs.chromium.org/p/chromium/issues/detail?id=717525))。
* Safari 要求没有凭据且相同来源的模块，即使你使用`crossorigin`属性 ([issue](https://bugs.webkit.org/show_bug.cgi?id=171550))。
* Edge 默认情况下，它会将凭据发送到相同的源，但是如果添加了`crossorigin` 属性，则不会发送([issue](https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/11865956/))。

Firefox是唯一做得好的。

## Mime-types

与常规脚本不同，module 脚本必须使用其中一种[有效的JavaScript MIME类型](https://html.spec.whatwg.org/multipage/scripting.html#javascript-mime-type) 否则它们将不会执行。

[演示页面](https://module-script-tests-zoelmqooyv.now.sh/mime)

### 浏览器遗留问题

* Edge 使用无效的MIME类型执行脚本([issue](https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/11865977/)).

以上就是我迄今为止所了解到的有关浏览器对 ES Modules 支持的情况，不用说为此我有多兴奋了吧！
