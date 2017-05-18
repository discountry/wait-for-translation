# JavaScript函数的各个方面

>原文链接：[The Many Faces of Functions in JavaScript](https://bocoup.com/blog/the-many-faces-of-functions-in-javascript)
>
>作者：[Leo Balter](https://bocoup.com/about/bocouper/leo-balter) and [Rick Waldron](https://bocoup.com/about/bocouper/rick-waldron)
>
>译者：[Liubara](https://Liubara.github.io)
>
>本文已获得作者授权，转载请注明出处。

如果你曾与JavaScript代码打过交道，你应该会很熟悉如何定义和调用函数，但是你真的知道有多少种定义函数的方法吗？对于编写和维护测试[Test262](https://github.com/tc39/test262)（浏览器JavaScript标准测试）来说，这是一个十分常见的挑战，尤其是当一个新特性出现且与现有的函数语法有关联，或者扩展了现有函数的API时。有必要确保新的或被提议的语法和API是有效的，且对语言中的每一个现有变体都是有效的。

下面是对已经存在的JavaScript函数语法格式的概述说明。本文档不包括类声明和表达式，因为这些形式生成的对象不是“可调用的”，对于本文来说，我们只关注生成“可调用的”函数对象的格式。此外，这篇文章也不包括参数列表（包括默认参数、解构，或者尾逗号），因为这些话题足够再写一篇文章了。
*译者注：类声明、默认参数、解构和尾逗号是ES6/7的新特性~~不妨碍看这篇文章哈*

### 旧方法
#### 函数声明和表达式
大家都知道，最广泛应用也是最早的函数定义形式就是函数声明和函数表达式。前者是最初设计的一部分（1995）并出现在[第一个版本的规范（1997年）](https://www.ecma-international.org/publications/files/ECMA-ST-ARCH/ECMA-262,%201st%20edition,%20June%201997.pdf)中，后者是在[第三个版本（1999）](https://www.ecma-international.org/publications/files/ECMA-ST-ARCH/ECMA-262,%203rd%20edition,%20December%201999.pdf)引入的。我们可以从这些规范中提取三种不同的定义形式：
```
// 函数声明
function BindingIdentifier() {}

// 命名函数表达式
// (BindingIdentifier对函数外部不可访问)
(function BindingIdentifier() {}); 

// 匿名函数表达式
(function() {});
```
要注意的是匿名函数表达式可能仍然有一个“名称”，具体的可以看Mike Pennisi的一篇文章[What's in a Function Name?](https://bocoup.com/blog/whats-in-a-function-name)
*译者注：能拿到授权的话我也会翻译这篇的*

#### 函数构造器
当我们在讨论一种语言的“函数API”的时候，就已经开始讨论函数构造器。在考虑最初的语言设计时，语法函数声明形式可以被解释为函数构造器的API的“文字”形式。函数构造器为定义函数提供了一种方法，通过N个字符串参数指定函数参数和函数主体，（如下例）最后一个字符串参数始终是函数主体(需要指出的是，这是动态代码评估的一种形式，可能会暴露安全性问题)。对于大多数情况来说，这种形式并不适合，因此它的使用非常稀少——但是自从第一个版本的ECMAScript以来，它就一直存在在JavaScript中了。
```
new Function('x', 'y', 'return x ** y;');
```

### 新方法
自从[ES2015](https://www.ecma-international.org/ecma-262/6.0/index.html)推出以来，已经引入了几种新的语法形式。这些形式的变化是巨大的!

#### not-so-anonymous函数声明
这是一种新的匿名函数声明形式，如果你曾用过ES Modules，应该清楚这种语法。虽然它可能看起来与匿名函数表达式非常相似，[但它实际上有一个默认名称，即“*default*”。](https://tc39.github.io/ecma262/#sec-function-definitions-static-semantics-boundnames)
```
// not-so-anonymous 函数声明
export default function() {}
```
顺便说一下，这个“名称”（指“default”）本身并不是有效的标识符，并且没有绑定在该匿名函数上。

#### 方法定义
对于下面的语句，大家应该能很快发现它定义了匿名和命名函数表达式作为属性的值。注意，这些不是不同的语法形式。它们是前面讨论过的函数表达式的示例，它是在初始对象中编写的。这些形式最初是在ES3中引入的。
```
let object = {
  propertyName: function() {},
};
let object = {
  // (BindingIdentifier在这个函数中不可访问)
  propertyName: function BindingIdentifier() {},
};
```
在ES5中引入访问器属性定义:
```
let object = {
  get propertyName() {},
  set propertyName(value) {},
};
```
从ES2015开始，JavaScript提供了一个简单的语法来定义方法，包括文字属性名称和计算的属性名称形式，以及访问器形式:
```
let object = {
  propertyName() {},
  ["computedName"]() {},
  get ["computedAccessorName"]() {},
  set ["computedAccessorName"](value) {},
};
```
我们还可以使用这些新形式作为类声明和表达式中的原型方法的定义:
```
// 类声明
class C {
  methodName() {}
  ["computedName"]() {}
  get ["computedAccessorName"]() {}
  set ["computedAccessorName"](value) {}
}

// 类表达式
let C = class {
  methodName() {}
  ["computedName"]() {}
  get ["computedAccessorName"]() {}
  set ["computedAccessorName"](value) {}
};
```
和定义静态方法:
```
// 类声明
class C {
  static methodName() {}
  static ["computedName"]() {}
  static get ["computedAccessorName"]() {}
  static set ["computedAccessorName"](value) {}
}

// 类表达式
let C = class {
  static methodName() {}
  static ["computedName"]() {}
  static get ["computedAccessorName"]() {}
  static set ["computedAccessorName"](value) {}
};
```

#### 箭头函数

作为ES2015最具争议性的函数之一，箭头函数已经变得众所周知且无处不在。箭头函数语法是这样定义的，它为函数声明提供了两种不同的格式：赋值表达式（箭头后面没有跟“{”方括号时为赋值表达式）和函数体（代码中包括0到多个语句时为函数体）。这个语法还允许在描述单个参数时不加圆括号，然而0个或一个以上参数需要加圆括号。这些语法结构允许箭头函数拥有多种书写形式：
```
// 木有参数的赋值表达式
(() => 2 ** 2);

// 单个参数，忽略括号的赋值表达式
(x => x ** 2);

// 单个参数，忽略括号且直接跟函数体
(x => { return x ** 2; });

// 一个以上括起来的参数和赋值表达式
((x, y) => x ** y);
```
在上面所示的最后一种形式中，参数被描述为一个*括起来的参数列表*，因为它们被包装在括号内。这提供了一种语法来标记参数列表或特殊的解构模式，就像({ x })= > x。

未被括起来的形式——也就是没有圆括号的形式——在箭头函数表现为只用一个标识符名称作为参数的形式。当箭头函数在异步函数或生成器中定义时，这个标识符名称需要以`await`或`yeild`作为前缀定义。但这是我们在箭头函数中能得到的最大程度的不用括号括起来参数列表的情况。

箭头函数可以(也经常)出现在初始化值或属性定义的赋值中，上面所示的箭头函数表达式已经包括了这种情况，如下面的例子所示:
```
let foo = x => x ** 2;

let object = {
  propertyName: x => x ** 2
};
```

#### 生成器
生成器有一种特殊的语法，它可以被添加在所有语法形式中，除了箭头函数和定义setter / getter方法的时候不能添加。我们可以用其生成函数声明、表达式、定义，甚至构造函数。让我们把它们列出来:
```
// 生成器声明
function *BindingIdentifer() {}

// 另一种 not-so-anonymous 生成器声明
export default function *() {}

// 生成器表达式
// (BindingIdentifier 对函数外部不可访问)
(function *BindingIdentifier() {});

// 匿名生成器表达式
(function *() {});

// 方法定义
let object = {
  *methodName() {},
  *["computedName"]() {},
};

// 类声明中的方法定义
class C {
  *methodName() {}
  *["computedName"]() {}
}

// 类声明中的静态方法定义
class C {
  static *methodName() {}
  static *["computedName"]() {}
}

// 类表达式中的方法定义
let C = class {
  *methodName() {}
  *["computedName"]() {}
};

// 类表达式中的静态方法定义
let C = class {
  static *methodName() {}
  static *["computedName"]() {}
};
```

### ES2017
#### 异步函数
经历了几年的发展，异步函数将于2017年6月发布ES2017的——EcmaScript语言规范的第8版引入。尽管如此，许多开发人员已经使用了这个特性，这要归功于[Babel](https://babeljs.io/)的早期实现支持！

Async函数语法为描述异步操作提供了一种干净而统一的方式。调用时，Async函数对象将返回一个Promise对象，这个对象将在异步函数返回时被解析。当包含一个`await`表达式时，异步函数可能暂停函数的执行，然后将其用作异步函数的返回值。

它的语法和我们从其他形式中所知道的一样:
```
// 异步函数声明
async function BindingIdentifier() { /**/ }

// not-so-anonymous 异步函数声明
export default async function() { /**/ }

// 命名异步函数表达式
// (BindingIdentifier is not accessible outside of this function)
(async function BindingIdentifier() {});

// 匿名异步函数表达式
(async function() {});

// 异步方法
let object = {
  async methodName() {},
  async ["computedName"]() {},
};

// 类声明中的异步方法
class C {
  async methodName() {}
  async ["computedName"]() {}
}

// 类声明中的静态异步方法
class C {
  static async methodName() {}
  static async ["computedName"]() {}
}

// 类声明中的异步方法
let C = class {
  async methodName() {}
  async ["computedName"]() {}
};

// 类表达式中的异步方法
let C = class {
  static async methodName() {}
  static async ["computedName"]() {}
};
```
#### 异步箭头函数
`async`和`await`并不局限于普通的声明和表达式形式，它们也可以用于箭头函数:
```
// 单个参数的赋值表达式
(async x => x ** 2);

// 单个参数的函数体
(async x => { return x ** 2; });

// 括起来的参数列表后跟赋值表达式
(async (x, y) => x ** y);

// 括起来的参数列表后跟函数体
(async (x, y) => { return x ** y; });
```
### ES2017
#### 异步生成器 `Async Generators`
在2017年，`async`和`await`关键字将被扩展以支持新的异步生成器形式。这个特性的进展可以通过[proposal’s github repository](https://github.com/tc39/proposal-async-iteration)进行跟踪。您可能已经猜到，这是`await`、`async`和现有的生成器声明和表达式语法的组合。调用时，异步生成器返回一个迭代器，它的`next()`方法返回Promise对象， 且用迭代器结果对象解决，而不是直接返回迭代器结果对象。

可以在许多地方发现异步生成器，您可能已经找到了生成器函数。
```
// 异步生成器声明
async function *BindingIdentifier() { /**/ }

// not-so-anonymous 异步生成器声明
export default async function *() {}

// 异步生成器表达式
// (BindingIdentifier在函数外部不可访问)
(async function *BindingIdentifier() {});

// 匿名函数表达式
(async function *() {});

// 方法定义
let object = {
  async *propertyName() {},
  async *["computedName"]() {},
};


// 类声明中的原型方法定义
class C {
  async *propertyName() {}
  async *["computedName"]() {}
}

// 类表达式中的原型方法定义
let C = class {
  async *propertyName() {}
  async *["computedName"]() {}
};

// 类声明中的静态方法定义
class C {
  static async *propertyName() {}
  static async *["computedName"]() {}
}

// 类表达式中的静态方法定义
let C = class {
  static async *propertyName() {}
  static async *["computedName"]() {}
};
```

### 复杂的挑战
每个函数语法格式不仅对学习和使用是挑战，而且对JS runtimes和Test262的实现和维护也是一个挑战。当引入新的语法形式时，Test262必须与所有相关的语法规则一起测试该格式。例如，将默认参数语法的测试限制为简单的函数声明形式，并假设在其他格式下也正常起作用是不明智的。每一个语法规则都必须经过测试，将这些测试任务分配给一个人是不合理的。所以导致了测试生成工具的设计和实现。测试生成工具提供了一种确保能够覆盖（函数格式的多少）更详尽的方法。

这个项目现在包含了一系列由不同的测试用例和模板组成的[源文件](https://github.com/tc39/test262/tree/master/src)，例如，[如何检查每个函数格式的参数](https://github.com/tc39/test262/tree/master/src/arguments)，或者[函数格式测试](https://github.com/tc39/test262/tree/master/src/function-forms)，甚至更多超出范围的函数形式，在这些函数形式中，[解构绑定](https://github.com/tc39/test262/tree/master/src/dstr-binding)和[解构赋值](https://github.com/tc39/test262/tree/master/src/dstr-assignment)都是适用的。

尽管它可能导致密集的和长时间的发送请求，但是覆盖率总是会提高，而且可能[总是会发现新的错误](https://github.com/tc39/test262/pull/651)。

### 为什么了解所有的函数格式是很重要的?

如果不需要在Test262上编写测试，计算和列出所有函数表单可能并不重要。这里已经列出了[许多格式的模板](https://github.com/tc39/test262/tree/master/src/function-forms/default)。新的测试可以很容易地使用现有的模板作为起点。

确保EcmaScript规范的良好测试是Test262的主要任务。这对所有的JavaScript运行时间都有直接的影响，我们识别的格式越多，覆盖率就越全面，这将帮助新功能更无缝地集成，不管您使用的平台是什么。