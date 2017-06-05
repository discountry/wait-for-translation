# Understanding Node.js Event-Driven Architecture

> [Understanding Node.js Event-Driven Architecture](https://medium.freecodecamp.com/understanding-node-js-event-driven-architecture-223292fcbc2d)
>
> 作者：Samer Buna
>
> 译者：李序锴
>
>转载请注明出处

## 理解Node.js事件驱动架构

Most of Node’s objects — like HTTP requests, responses, and streams — implement the `EventEmitter` module so they can provide a way to emit and listen to events.

大多数的Node对象（如HTTP requests, responses以及streams）都实现了`EventEmitter`模块，这样它们就能够触发和监听事件。

![Alt text](./images/1.png)

The simplest form of the event-driven nature is the callback style of some of the popular Node.js functions — for example, `fs.readFile`. In this analogy, the event will be fired once (when Node is ready to call the callback) and the callback acts as the event handler.

事件驱动特性最简单的形式就是通用的Node.js函数当中的一些回调风格（例如：`fs.readFile`）。在这个类比中，事件会立即启动（当Node准备调用回调时）而回调则充当事件处理器的角色。

**Call me when you’re ready, Node!**

**当你准备好请调用我，Node！**

The original way Node handled asynchronous events was with callback. This was a long time ago, before JavaScript had native promises support and the async/await feature.

Node处理异步事件的最初方式就是通过回调。而这已经是JavaScript拥有原生promises支持以及async/await特性之前很就以前的事情了。

Callbacks are basically just functions that you pass to other functions. This is possible in JavaScript because functions are first class objects.

回调基本上都是你传递给其它函数的函数。这可能是因为在JavaScript当中函数是第一类对象。

It’s important to understand that callbacks do not indicate an asynchronous call in the code. A function can call the callback both synchronously and asynchronously.
For example, here’s a host function fileSize that accepts a callback function cb and can invoke that callback function both synchronously and asynchronously based on a condition:

回调并不会在代码当中注明它是异步调用，理解这一点很关键。函数可以通过同步和异步两种方式触发回调。例如：如下是一个宿主函数fileSize，它接收一个回调函数cb，根据条件的不同它可以执行同步和异步两种方式的回调。

```
function fileSize (fileName, cb) {
  if (typeof fileName !== 'string') {
    return cb(new TypeError('argument should be string')); // Sync
  }
  fs.stat(fileName, (err, stats) => {
    if (err) { return cb(err); } // Async
    cb(null, stats.size); // Async
  });
}
```

Note that this is a bad practice that leads to unexpected errors. Design host functions to consume callback either always synchronously or always asynchronously.

但是这是一种会导致预料之外错误的糟糕实践。它使得宿主函数总是以同步或者异步方式执行回调。

Let’s explore a simple example of a typical asynchronous Node function that’s written with a callback style:

我们来探寻一个使用回调风格书写的异步Node函数的典型例子。

```
const readFileAsArray = function(file, cb) {
  fs.readFile(file, function(err, data) {
    if (err) {
      return cb(err);
    }
    const lines = data.toString().trim().split('\n');
    cb(null, lines);
  });
};
```
`readFileAsArray` takes a file path and a callback function. It reads the file content, splits it into an array of lines, and calls the callback function with that array.

`readFileAsArray`接受文件路径和回调函数作为参数。其读取文件内容并将文件内容分割为数组，再利用该数组调用回调函数。

Here’s an example use for it. Assuming that we have the file `numbers.txt` in the same directory with content like this:

下方是一个应用实例。假设我们在同级目录下有一个`numbers.txt`文件，其内容如下：

```
10
11
12
13
14
15
```
If we have a task to count the odd numbers in that file, we can use `readFileAsArray` to simplify the code:

假设我们的任务是统计该文件当中的奇数个数，我们可以使用`readFileAsArray`来简化代码：

```
readFileAsArray('./numbers.txt', (err, lines) => {
  if (err) throw err;
  const numbers = lines.map(Number);
  const oddNumbers = numbers.filter(n => n%2 === 1);
  console.log('Odd numbers count:', oddNumbers.length);
});
```

The code reads the numbers content into an array of strings, parses them as numbers, and counts the odd ones.

这段代码将数字内容读取成了字符串数组，之后解析成数字，再之后统计了奇数的数量。

Node’s callback style is used purely here. The callback has an error-first argument err that’s nullable and we pass the callback as the last argument for the host function. You should always do that in your functions because users will probably assume that. Make the host function receive the callback as its last argument and make the callback expect an error object as its first argument.

Node回调风格在此处获得了充分应用。该回调的第一个参数是可为空的err参数，我们将该回调作为最后一个参数传递给宿主函数。由于用户的阅读习惯问题，因此你应该在你的函数一直按这种形式书写。让回调作为宿主函数的的最后一个参数，将error对象作为回调函数的第一个参数。

**The modern JavaScript alternative to Callbacks**

**新版JavaScript对于回调的替代形式**

In modern JavaScript, we have promise objects. Promises can be an alternative to callbacks for asynchronous APIs. Instead of passing a callback as an argument and handling the error in the same place, a promise object allows us to handle success and error cases separately and it also allows us to chain multiple asynchronous calls instead of nesting them.

在新版JavaScript当中，我们有了promise对象。在异步APIs中Promises可以作为异步回调的一种替代形式。promise对象允许我们分别处理success和error的cases而非在同一处同时传递callback和error两个参数，并且promise也允许我们串联多重异步调用而不是进行嵌套。

If the `readFileAsArray` function supports promises, we can use it as follows:

如果`readFileAsArray`函数支持promises，我们可以做如下应用：

```
readFileAsArray('./numbers.txt')
  .then(lines => {
    const numbers = lines.map(Number);
    const oddNumbers = numbers.filter(n => n%2 === 1);
    console.log('Odd numbers count:', oddNumbers.length);
  })
  .catch(console.error);
```

Instead of passing in a callback function, we called a `.then` function on the return value of the host function. This `.then` function usually gives us access to the same lines array that we get in the callback version, and we can do our processing on it as before. To handle errors, we add a `.catch` call on the result and that gives us access to an error when it happens.

我们没有传递callback函数，而是调用了`.then`函数作为宿主函数的的返回值。这个`.then`函数通常能让我们达到跟利用带有callback函数的代码同样的效果，而且我们也能够像之前一样在其上做处理。为了处理errors，我们在末尾添加了`.catch`代码块，当发生错误时我们利用`.catch`代码块进行处理。

Making the host function support a promise interface is easier in modern JavaScript thanks to the new Promise object. Here’s the `readFileAsArray` function modified to support a promise interface in addition to the callback interface it already supports:

由于新Promise对象的存在，让宿主函数在新版JavaScript支持promise接口变得更加容易。修改如下的`readFileAsArray`函数让其支持promise接口，以及支持之前已经支持的callback接口。

```
const readFileAsArray = function(file, cb = () => {}) {
  return new Promise((resolve, reject) => {
    fs.readFile(file, function(err, data) {
      if (err) {
        reject(err);
        return cb(err);
      }
      const lines = data.toString().trim().split('\n');
      resolve(lines);
      cb(null, lines);
    });
  });
};
```

So we make the function return a Promise object, which wraps the `fs.readFile` async call. The promise object exposes two arguments, a `resolve` function and a `reject` function.

因此我们让函数返回一个Promise对象，这个Promise对象包含着`fs.readFile`异步回调。该promise对象对外暴露两个参数（一个`resolve`函数以及一个`reject`函数）。

Whenever we want to invoke the callback with an error we use the promise `reject` function as well, and whenever we want to invoke the callback with data we use the promise `resolve` function as well.

我们总是会运用promise的`reject`函数执行回调来处理error，同时也总是利用`resolve`函数执行回调来处理data。

The only other thing we needed to do in this case is to have a default value for this callback argument in case the code is being used with the promise interface. We can use a simple, default empty function in the argument for that case: () => {}.

另外我们在这个例子当中需要为回调参数设置一个默认值，因为这段代码有可能会被用于promise接口。我们可以使用一种简单的空函数作为默认值，如：() => {}。

**Consuming promises with async/await**

**以async/await方式执行promises**

Adding a promise interface makes your code a lot easier to work with when there is a need to loop over an async function. With callbacks, things become messy.

当需要循环嵌套异步函数时，添加promise接口会让你的代码更易于维护。回调则会让情况变得复杂。

Promises improve that a little bit, and function generators improve on that a little bit more. This said, a more recent alternative to working with async code is to use the `async` function, which allows us to treat async code as if it was synchronous, making it a lot more readable overall.

Promises稍稍改善了一些这种状况，而函数生成器则带来更多的优化。即是说处理异步代码更新近的替代方式是使用`async`函数，它能让我们像是以一种同步的方式处理异步的代码，这回让代码更具有可读性。

Here’s how we can consume the `readFileAsArray` function with async/await:

我们通过async/await方式执行`readFileAsArray`函数，代码如下：

```
async function countOdd () {
  try {
    const lines = await readFileAsArray('./numbers');
    const numbers = lines.map(Number);
    const oddCount = numbers.filter(n => n%2 === 1).length;
    console.log('Odd numbers count:', oddCount);
  } catch(err) {
    console.error(err);
  }
}
countOdd();
```

We first create an async function, which is just a normal function with the word async before it. Inside the async function, we call the readFileAsArray function as if it returns the lines variable, and to make that work, we use the keyword await. After that, we continue the code as if the readFileAsArray call was synchronous.

我们先创建了一个异步函数，就是在普通函数的function之前加上关键字async。在这个异步函数当中，我们调用readFileAsArray函数（假设它返回了lines变量），为了让这种方式生效，我们使用关键字await。之后，我们继续执行代码就如同对readFileAsArray进行同步调用一样。

To get things to run, we execute the async function. This is very simple and more readable. To work with errors, we need to wrap the async call in a try/catch statement.

为了让代码顺利运行，我们执行异步函数。这让代码变得非常简洁且易于阅读。而为了能够处理errors，我们需要将异步调用包裹在try/catch语句中。

With this async/await feature, we did not have to use any special API (like .then and .catch). We just labeled functions differently and used pure JavaScript for the code.

通过这种async/await特性，我们就不必使用其它特殊的API(例如.then和.catch)。我们仅仅需要特别标记一下函数就可以使用纯粹的JavaScript进行编程。

We can use the async/await feature with any function that supports a promise interface. However, we can’t use it with callback-style async functions (like setTimeout for example).

我们可以在任何支持promise接口的函数当中使用async/await特性，但是我们不能将其用于回调风格的异步函数之中(例如:setTimeout)。

## The EventEmitter Module

## EventEmitter模块

The EventEmitter is a module that facilitates communication between objects in Node. EventEmitter is at the core of Node asynchronous event-driven architecture. Many of Node’s built-in modules inherit from EventEmitter.

EventEmitter是一种支持Node当中对象间通信的模块。EventEmitter是Node异步事件驱动架构的核心。很多Node内置模块都是继承自EventEmitter。

The concept is simple: emitter objects emit named events that cause previously registered listeners to be called. So, an emitter object basically has two main features:

概念很简单：emitter对象发出已经命名好的事件，该事件会触发之前注册好的监听器。因此，一个emitter对象通常有两个主要特性：

* Emitting name events
* 发出命名事件
* Registering and unregistering listener functions.
* 注册和注销监听函数

To work with the EventEmitter, we just create a class that extends EventEmitter.

为了理解EventEmitter，我们创建一个继承自EventEmitter的类(class)
```
class MyEmitter extends EventEmitter {

}
```
Emitter objects are what we instantiate from the EventEmitter-based classes:

Emitter对象是我们通过MyEmitter类(继承自类EventEmitter)实例化生成的。

```
const myEmitter = new MyEmitter();
```

At any point in the lifecycle of those emitter objects, we can use the emit function to emit any named event we want.

在这些emitter对象生命周期的任何阶段，我们都可以通过emit函数发射我们想要发射的任何命名事件。

```
myEmitter.emit('something-happened');
```

Emitting an event is the signal that some condition has occurred. This condition is usually about a state change in the emitting object.

单次事件的触发表明已经满足某些条件。该条件在触发对象中通常是一种状态变化。

We can add listener functions using the on method, and those listener functions will be executed every time the emitter object emits their associated name event.

我们可以通过on方法添加监听函数，每当发射器对象触发相关联的命名事件时这些监听函数都会执行。

**Events !== Asynchrony**

**事件!==异步**

Let’s take a look at an example:

我们来看一个例子：
```
const EventEmitter = require('events');

class WithLog extends EventEmitter {
  execute(taskFunc) {
    console.log('Before executing');
    this.emit('begin');
    taskFunc();
    this.emit('end');
    console.log('After executing');
  }
}

const withLog = new WithLog();

withLog.on('begin', () => console.log('About to execute'));
withLog.on('end', () => console.log('Done with execute'));

withLog.execute(() => console.log('*** Executing task ***'));
```

Class WithLog is an event emitter. It defines one instance function execute. This execute function receives one argument, a task function, and wraps its execution with log statements. It fires events before and after the execution.

WithLog类是一个事件发射器。它定义了一个函数执行的实例。该执行函数接受一个参数(一个任务函数)并用日志声明包裹该执行函数。这些日志声明会在函数执行前后触发。

To see the sequence of what will happen here, we register listeners on both named events and finally execute a sample task to trigger things.

为了查看函数的执行顺序，我们在两个命名事件上添加了监听器，最后会执行一个样本任务以启动其它函数。

Here’s the output of that:

如下是函数的输出结果：

```
Before executing
About to execute
*** Executing task ***
Done with execute
After executing
```

What I want you to notice about the output above is that it all happens synchronously. There is nothing asynchronous about this code.

对于这些输出结果我希望你注意到它们都是同步执行的。这段代码当中没有异步操作。

* We get the “Before executing” line first.

* 首先输出了"Before executing"

* The begin named event then causes the “About to execute” line.

* 以begin命名的事件输出了"About to execute"

* The actual execution line then outputs the “*** Executing task ***” line.

* 实际执行行(hang)之后输出了"*** Executing task ***"

* The end named event then causes the “Done with execute” line

* 以end命名的事件输出了"Done with execute"

* We get the “After executing” line last.

* 最后我们得到了"After executing"

Just like plain-old callbacks, do not assume that events mean synchronous or asynchronous code.

如同plain-old回调一样，事件跟代码是同步还是异步执行没有什么关联。

This is important, because if we pass an asynchronous taskFunc to execute, the events emitted will no longer be accurate.

这点很重要，因为如果我们传入异步taskFunc函数去执行的话，事件的触发将会变得不够精准。

We can simulate the case with a setImmediate call:

我们可以用带有setImmediate的调用来模拟上面的例子：

```
// ...

withLog.execute(() => {
  setImmediate(() => {
    console.log('*** Executing task ***')
  });
});
```

Now the output would be:

输出就会变成像下面这样：
```
Before executing
About to execute
Done with execute
After executing
*** Executing task ***
```

This is wrong. The lines after the async call, which were caused the “Done with execute” and “After executing” calls, are not accurate any more.

这是错误的。异步调用(其调用了"Done with execute"和"After executing")之后的那几行已经不再准确。

To emit an event after an asynchronous function is done, we’ll need to combine callbacks (or promises) with this event-based communication. The example below demonstrates that.

为了在一个异步函数执行完成之后触发事件，我们需要结合基于事件通信的回调机制。下面的例子会进行说明。

One benefit of using events instead of regular callbacks is that we can react to the same signal multiple times by defining multiple listeners. To accomplish the same with callbacks, we have to write more logic inside the single available callback. Events are a great way for applications to allow multiple external plugins to build functionality on top of the application’s core. You can think of them as hook points to allow for customizing the story around a state change.

使用事件而非回调的好处在于我们可以通过定义多个监听器对同一信号响应多次。用回调实现相同功能的话，我们需要在单个回调当中书写更多的逻辑。事件是一种很好的实现方式，它让应用程序能够通过多个外部插件在其核心之上构建功能。你可以将它们当做hook points，它们会为状态变化作特定的记录。


**Asynchronous Events**

**异步事件**

Let’s convert the synchronous sample example into something asynchronous and a little bit more useful.

我们现在将这个同步的简单例子改写成更加实用的异步代码。

```
const fs = require('fs');
const EventEmitter = require('events');

class WithTime extends EventEmitter {
  execute(asyncFunc, ...args) {
    this.emit('begin');
    console.time('execute');
    asyncFunc(...args, (err, data) => {
      if (err) {
        return this.emit('error', err);
      }

      this.emit('data', data);
      console.timeEnd('execute');
      this.emit('end');
    });
  }
}

const withTime = new WithTime();

withTime.on('begin', () => console.log('About to execute'));
withTime.on('end', () => console.log('Done with execute'));

withTime.execute(fs.readFile, __filename);
```

The WithTime class executes an asyncFunc and reports the time that’s taken by that asyncFunc using console.time and console.timeEnd calls. It emits the right sequence of events before and after the execution. And also emits error/data events to work with the usual signals of asynchronous calls.

WithTime类执行了asyncFunc并通过使用console.time以及console.timeEnd记录了asyncFunc的执行时间。在程序执行前后，它触发了事件执行的正确顺序。同时利用error/data事件去处理异步调用当中的常见信号。

We test a withTime emitter by passing it an fs.readFile call, which is an asynchronous function. Instead of handling file data with a callback, we can now listen to the data event.

我们传入fs.readFile函数(它是异步函数)来测试withTime发射器。而非用回调来处理文件，如此我便能够监听数据对象了。

When we execute this code , we get the right sequence of events, as expected, and we get a reported time for the execution, which is helpful:

当执行这段代码时，我们得到事件的正确执行顺序。不出所料，我们获得了指定代码的执行时间，这很有用处：

```
About to execute
execute: 4.507ms
Done with execute
```

Note how we needed to combine a callback with an event emitter to accomplish that. If the asynFunc supported promises as well, we could use the async/await feature to do the same:

我们怎样结合带有事件监听器的回调来实现呢？如果asynFunc函数也支持promises的话，我们可以使用async/await特性来实现相同的功能：

```
class WithTime extends EventEmitter {
  async execute(asyncFunc, ...args) {
    this.emit('begin');
    try {
      console.time('execute');
      const data = await asyncFunc(...args);
      this.emit('data', data);
      console.timeEnd('execute');
      this.emit('end');
    } catch(err) {
      this.emit('error', err);
    }
  }
}
```

I don’t know about you, but this is much more readable to me than the callback-based code or any .then/.catch lines. The async/await feature brings us as close as possible to the JavaScript language itself, which I think is a big win.

我不太清楚你的情况，但是相较于基于回调和带有.then/.catch的代码段我觉得以上代码更容易理解。async/await特性让我们更接近于JavaScript语言本身，我认为这是一个重大进展。

**Events Arguments and Errors**

**事件参数和错误**

In the previous example, there were two events that were emitted with extra arguments.

在前面的例子当中，有两个事件都是由额外的参数来触发。

The error event is emitted with an error object.

error事件是通过一个error对象触发。

```
this.emit('error', err);
```

The data event is emitted with a data object.

data事件则是由一个data对象触发。

```
this.emit('data', data);
```

We can use as many arguments as we need after the named event, and all these arguments will be available inside the listener functions we register for these named events.

我们可以根据需要在命名事件之后使用任意数量的参数，所有参数在我们为这些命名事件注册的监听函数当中都可以使用。

For example, to work with the data event, the listener function that we register will get access to the data argument that was passed to the emitted event and that data object is exactly what the asyncFunc exposes.

例如，为了处理data事件，我们注册的监听事件将能够获得我们传递给被发射事件的数据参数，而其就是asyncFunc函数暴露的数据对象。

```
withTime.on('data', (data) => {
  // do something with data
});
```

The `error` event is usually a special one. In our callback-based example, if we don’t handle the error event with a listener, the node process will actually exit.

`error`事件通常是一种特殊情况。在基于回调的例子当中，如果我们不使用监听器处理error事件的话，node进程实际上会退出。

To demonstrate that, make another call to the execute method with a bad argument:

为了说明这种情况，我们用一个恶性参数对执行方法做再一次调用：

```
class WithTime extends EventEmitter {
  execute(asyncFunc, ...args) {
    console.time('execute');
    asyncFunc(...args, (err, data) => {
      if (err) {
        return this.emit('error', err); // Not Handled
      }

      console.timeEnd('execute');
    });
  }
}

const withTime = new WithTime();

withTime.execute(fs.readFile, ''); // BAD CALL
withTime.execute(fs.readFile, __filename);
```

The first execute call above will trigger an error. The node process is going to crash and exit:

如上代码的第一次执行会引起错误。node进程将会崩溃并退出：

```
events.js:163
      throw er; // Unhandled 'error' event
      ^
Error: ENOENT: no such file or directory, open ''
```

The second execute call will be affected by this crash and will potentially not get executed at all.

第二次执行会受到崩溃影响根本不会继续往下执行。

If we register a listener for the special error event, the behavior of the node process will change. For example:

如果为该特殊error事件注册监听器的话，node进程的行为将会发生变化。例如：

```
withTime.on('error', (err) => {
  // do something with err, for example log it somewhere
  console.log(err)
});
```
If we do the above, the error from the first execute call will be reported but the node process will not crash and exit. The other execute call will finish normally:

如果执行以上代码，第一次执行的错误会被播报，但是node进程不会崩溃和退出。其它的执行调用会正常结束：

```
{ Error: ENOENT: no such file or directory, open '' errno: -2, code: 'ENOENT', syscall: 'open', path: '' }
execute: 4.276ms
```

Note that Node currently behaves differently with promise-based functions and just outputs a warning, but that will eventually change:

现在基于promise的函数会有不同的行为并且只是输出警告，但是最终会发生变化：
```
UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): Error: ENOENT: no such file or directory, open ''

DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
```
The other way to handle exceptions from emitted errors is to register a listener for the global uncaughtException process event. However, catching errors globally with that event is a bad idea.

处理已触发错误当中异常的另一种方式是为全局uncaughtException进程事件注册一个监听器。然而，全部捕捉errors是一种糟糕的想法。

The standard advice about uncaughtException is to avoid using it, but if you must do (say to report what happened or do cleanups), you should just let the process exit anyway:

对uncaughtException的通常建议是避免使用它，但是如果非用不可的话(播报发生的情况或者直接清除)，你应该直接让进程退出。

```
process.on('uncaughtException', (err) => {
  // something went unhandled.
  // Do any cleanup and exit anyway!

  console.error(err); // don't do just that.

  // FORCE exit the process too.
  process.exit(1);
});
```

However, imagine that multiple error events happen at the exact same time. This means the uncaughtException listener above will be triggered multiple times, which might be a problem for some cleanup code. An example of this is when multiple calls are made to a database shutdown action.

然而，想象一下多个error事件在同一时间发生。这意味着上面的uncaughtException监听器会启动多次，对于cleanup代码这会是个问题。一个典型的例子就是有多次调用用于数据库关闭操作。

The EventEmitter module exposes a once method. This method signals to invoke the listener just once, not every time it happens. So, this is a practical use case to use with the uncaughtException because with the first uncaught exception we’ll start doing the cleanup and we know that we’re going to exit the process anyway.

EventEmitter模块对外暴露一个一次性的方法。该方法只会启动一次监听器，不会每次都进行响应。因此，这是一个使用uncaughtException的实际用例，因为通过第一个未捕捉的异常我们将会进行清理操作，而且无论如何我们都将退出进程。

**Order of Listeners**

**监听器的顺序**

If we register multiple listeners for the same event, the invocation of those listeners will be in order. The first listener that we register is the first listener that gets invoked.

如果我们为同一事件注册了多个监听器，这些监听器会按照顺序执行。注册的第一个监听器将会第一个执行。

```
// प्रथम
withTime.on('data', (data) => {
  console.log(`Length: ${data.length}`);
});

// दूसरा
withTime.on('data', (data) => {
  console.log(`Characters: ${data.toString().length}`);
});

withTime.execute(fs.readFile, __filename);
```

The above code will cause the “Length” line to be logged before the “Characters” line, because that’s the order in which we defined those listeners.

上方代码中带有"Length"的那一行会先于带有"Characters"的那一行执行，因为这是我们定义监听器的顺序。

If you need to define a new listener, but have that listener invoked first, you can use the prependListener method:

如果你需要定义一个新监听器但是需要该监听器第一个执行，你可以使用prependListener方法：

```
// प्रथम
withTime.on('data', (data) => {
  console.log(`Length: ${data.length}`);
});

// दूसरा
withTime.prependListener('data', (data) => {
  console.log(`Characters: ${data.toString().length}`);
});

withTime.execute(fs.readFile, __filename);
```

The above will cause the “Characters” line to be logged first.

上方代码会先执行带有"Character"的那一行。

And finally, if you need to remove a listener, you can use the removeListener method.

最后，如果需要移除一个监听器，你可以使用removeListener方法。

That’s all I have for this topic. Thanks for reading! Until next time!

这就是我关于主题的全部阐述。多谢阅读！下次见！
