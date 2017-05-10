# 浏览器中的ECMAScript模块

>原文链接：[ECMAScript Modules in Browsers](https://jakearchibald.com/2017/es-modules-in-browsers/)

>作者：[Jake Archibald](https://twitter.com/jaffathecake)

>译者：张弈

>转载请注明出处。
 
ES模块正在登陆各大浏览器！他们在：

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

所有你需要的只是在script元素中的 `type=module`, 并且浏览器会将内联和外部脚本视为一个ECMAScript module处理。

已经有一些[关于该模块的精彩文章](https://ponyfoo.com/articles/es6-modules-in-depth), 但是我仍然想要分享一些我所测试过的浏览器特性。

## 目前还不支持"Bare" 导入说明符

```javascript
// Supported:
import {foo} from 'https://jakearchibald.com/utils/bar.js';
import {foo} from '/utils/bar.js';
import {foo} from './bar.js';
import {foo} from '../bar.js';

// Not supported:
import {foo} from 'bar.js';
import {foo} from 'utils/bar.js';
```

有效的模块说明符必须符合以下条件之一：

* 一个完整的非相对链接。当通过`new URL(moduleSpecifier)`时它不会抛出错误。
* 以 /开头。
* 以 ./开头。
* 以 ../开头。

其他说明符保留以供将来使用，如导入内置模块。

## nomodule的向后兼容性

```javascript
<script type="module" src="module.js"></script>
<script nomodule src="fallback.js"></script>
```

[演示页面](https://cdn.rawgit.com/jakearchibald/6110fb6df717ebca44c2e40814cc12af/raw/7fc79ed89199c2512a4579c9a3ba19f72c219bd8/)

能够理解`type=module`属性的浏览器可以忽略具有 `nomodule`属性的脚本。这意味着你可以将模块树提供给支持模块的浏览器，同时对其他浏览器提供一个回退。

### Browser issues

* ~~Firefox不支持~~ `nomodule` ([~~issue~~](https://bugzilla.mozilla.org/show_bug.cgi?id=1330900)). 在其nightly版本中已经被修复!
* Edge 不支持 `nomodule` ([issue](https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/10525830/))。
* Safari 10.1 不支持 `nomodule`, 但它在最新的技术预览中修复了。 10.1版本中有一个 [完美方案](https://gist.github.com/samthor/64b114e4a4f539915a95b91ffd340acc)。

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

脚本在抓取过程中阻止HTML解析器的方法是baaaad。 使用常规脚本，你可以使用`defer`来防止阻塞，这也会拖延脚本执行，直到文档完成解析，并使用其他延迟脚本维护执行顺序。 默认情况下，模块脚本的行为类似于`defer` - 在抓取时，无法使模块脚本阻止HTML解析器。

模块脚本使用与`defer`的相同执行队列。

## 内联脚本同样会延迟

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

[演示页面](https://cdn.rawgit.com/jakearchibald/7026f72c0675898196f7669699e3231e/raw/fc7521aabd9485f30dbd5189b407313cd350cf2b/)顺序应该是 `1.js`, 内联脚本, 内联模块, `2.js`。

常规内联脚本忽略`defer`，而无论它们是否导入任何东西，内联模块脚本总是被延迟。

## 在外部和内联模块上的异步工作

```javascript
<!-- This executes as soon as its imports have fetched -->
<script async type="module">
  import {addTextToBody} from './utils.js';

  addTextToBody('Inline module executed.');
</script>

<!-- This executes as soon as it & its imports have fetched -->
<script async type="module" src="1.js"></script>
```

[演示页面](https://module-script-tests-rgjnxtrtqq.now.sh/async)执行快速下载脚本应该在执行慢速下载脚本之前。

与常规脚本一样，`async`会导致脚本不阻止HTML解析器的同时下载并执行。与常规脚本不同，`async`也适用于内联模块。

与 `async`一样，脚本可能不会按照它在DOM中的显示顺序执行。

### Browser issues

* Firefox 在内联模块脚本上不支持 `async` ([issue](https://bugzilla.mozilla.org/show_bug.cgi?id=1361369))。

## 模块只执行一次

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

如果你了解ES模块，就会知道可以多次导入它们，但他们只会执行一次。这同样适用于HTML中的脚本模块，特定URL的模块脚本只能每页执行一次。

### Browser issues

*Edge 可以多次执行模块 ([issue](https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/11865922/)).

## Always CORS

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

与常规脚本不同，使用CORS获取模块脚本（及其导入）。这意味着跨原始模块脚本必须返回有效的CORS头，例如`Access-Control-Allow-Origin: *`。

### Browser issues

* Firefox 无法加载演示页面 ([issue](https://bugzilla.mozilla.org/show_bug.cgi?id=1361373))。
* Edge 加载演示页面无需CORS头 ([issue[(https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/11865934/))。

## 没有凭据

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

如果是相同的来源的请求，则大多数基于CORS的API将发送凭据（cookie等），但`fetch（）`和模块脚本是例外 - 除非有要求，否则它们将不会发送凭据。

可以通过添加`crossorigin`属性（我感到有点奇怪，[在规范中已经提出质疑](https://github.com/whatwg/html/issues/2557)），可以将相同原始模块的凭据添加到同一原始模块。如果你也想发送凭据到其他来源，请使用`crossorigin="use-credentials"`。注意，另一个来源必须使用 `Access-Control-Allow-Credentials: true`头进行响应。

另外，还有一个与“模块只执行一次”规则有关的内容。模块被其URL绑定，因此如果你请求没有凭据的模块，那么使用凭据请求将获得一样无凭证的模块。这就是为什么我在上面的URL中使用了“`?`”来确保它们唯一。

### Browser issues

* Chrome 要求有凭据且相同来源的模块 ([issue](https://bugs.chromium.org/p/chromium/issues/detail?id=717525))。
* Safari 要求没有凭据且相同来源的模块，即使你使用`crossorigin`属性 ([issue](https://bugs.webkit.org/show_bug.cgi?id=171550))。
* Edge 默认情况下，它会将凭据发送到相同的源，但是如果添加了`crossorigin` 属性，则不会发送([issue](https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/11865956/))。

Firefox是唯一做得好的。

## Mime-types

与常规脚本不同，模块脚本必须使用其中一种[有效的JavaScript MIME类型](https://html.spec.whatwg.org/multipage/scripting.html#javascript-mime-type) 否则它们将不会执行。

[演示页面](https://module-script-tests-zoelmqooyv.now.sh/mime)

### Browser issues

* Edge使用无效的MIME类型执行脚本([issue](https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/11865977/)).

这就是我迄今为止所学到的，不用多说，我非常高兴ES模块登陆浏览器！