# 生成器

深度探索 ES6 是一个针对 ECMAScript 标准中添加到 JavaScript 编程语言新功能的系列， 简称为 ES6。

关于今天的内容，我感到非常兴奋。今天，我们将要讨论  ES6 的最神奇的功能。

我所说的“神奇”的意思是什么呢？对于初学者来说，该功能与 JS 中已经存在的功能非常不同。这可能会让初学者们在第一次见到这个功能的时候感觉晦涩难懂。从某种意义上来讲，这个功能使得语言的内在的很平常的行为展露在开发者面前。如果这还不算是神奇，那我不知道什么才算了。

不仅如此：这个功能能够简化代码并且使得在超类上的“回调地狱”更加直白。

是不是我太看重这个了？让我们一起深入，然后你自己就可以进行判断了。

## 介绍 ES6 生成器

什么是生成器？

我们先从下面的这里例子开始。

``` javascript
function* quips(name) {
  yield "hello " + name + "!";
  yield "i hope you are enjoying the blog posts";
  if (name.startsWith("X")) {
    yield "it's cool how your name starts with X, " + name;
  }
  yield "see you later!";
}
```

这段代码是一个对话猫，这可能是当前网络上最重要的一类应用。( 继续，点击链接，与猫一起玩耍。当你彻底困惑了以后，你再回到这里看一下解释。)

这个在一定程度上看起来像一个函数，对不？这就被称为生成器函数，同时它与函数之间也有很多相似之处。但是你一下子就能发现两个不同之处：
- 普通的函数使用 function 作为开始。生成器函数以 function* 开始。
- 在一个生成器函数中，yield 是一个关键字，语法和 return 很相似。 区别在于，一个函数( 甚至是生成器函数)，只能返回一次，但是一个生成器函数能够 yield 很多次。 yield 表达式暂停生成器的运行，然后它能够在之后重新被使用。

就是这样的，以上就是普通的函数和生成器函数之间的大区别。普通的函数不能自己暂停。然而生成器函数可以自己暂停运行。

## 生成器的用处

当你调用 quips( ) 生成器函数时将会发生什么？

``` javascript
> var iter = quips("jorendorff");
  [object Generator]
> iter.next()
  { value: "hello jorendorff!", done: false }
> iter.next()
  { value: "i hope you are enjoying the blog posts", done: false }
> iter.next()
  { value: "see you later!", done: false }
> iter.next()
  { value: undefined, done: true }
```

你可能非常习惯于普通的函数以及他们的表现。当你调用他们的时候，他们立即开始运行，当遇到 return 或者 throw 的时候，他们停止运行。任何一个 JS 程序员都非常习惯于上述的过程。

调用一个生成器看起来是一样的：quips("jorendorff")。 但是当你调用一个生成器，他还不开始运行。反而，它返回一个暂停的生成器对象( 在上述的例子中被称为 iter )。你可以认为这个生成器对象是一个函数调用，暂时停止。特别的是，其在生成器函数一开始就停止了，在运行代码的第一行之前。

每次你调用生成器对象的 .next( ) 方法，函数将其自己解冻并运行直到其到达下一个 yield 表达式。

这就是上面代码中我们为什么要调用 iter.next( )，调用后我们获得一个不同的字符串值。这些值都是由 quips( ) 里的 yield 表达式产生的。

在最后一个 iter.next( ) 调用中，我们最后结束了生成器函数，所以结果的 .done 领域的值为true。 一个函数的结束就像是返回 undefined，而且这也是为什么结果的 .value 领域是不确定的 ( undefined)。

现在可能是一个好机会来返回到上面的对话猫的例子那页面上，同时真正地可以玩转代码。尝试着在一个循环中加入一个 yield。这会发生什么呢？

在技术层面上，每一次一个生成器进行 yield 操作，其堆栈桢，包括局部变量，参数，临时值，以及在生成器中的执行的当前位置，被从栈中删除。然而，生成器对象保有这个堆栈帧的引用（或者是副本）。因此，接下来的调用 .next( ) 可以重新激活它并继续执行。

值得指出的是，生成器都没有线程。在能使用线程的语言中，多份代码可以在同一时间运行，这通常导致了竞争条件，非确定性和非常非常好的性能。生成器和这完全不同。当生成器运行时，它与调用者运行在同一个线程中。执行的顺序是连续且确定的，并永远不会并发。不同于系统线程，生成器只会在代码中用 yield 标记的地方才会悬挂。

好了。我们知道生成器是什么了。我们已经看到了生成器器运行，暂停，然后恢复执行。现在的大问题是，这样奇怪的能力怎么可能是有用的呢？

## 生成器是迭代器  

上周，我们已经看到了 ES6 的迭代器不只是一个简单的内置类。他们是语言的扩展点。你可以通过实现两种方法来创建你自己的迭代器，这两种方法是：`[ Symbol.iterator ]( )`和 .next( )。 

但是，实现接口总是至少还是有一点工作量的。让我们来看看一个迭代器实现在实践中看起来是什么样的。因为是一个例子，让我们使用一个简单的范围 ( range ) 迭代器，只简单的从一个数字数到另一个数字，就像一个老式的 C 语言的 for ( ; ; ) 循环。

``` javascript
// 这个应该三次发出“叮”的声音
for (var value of range(0, 3)) {
  alert("Ding! at floor #" + value);
}
```

这里是一个使用 ES6 的类的解决方案。( 如果类的语言没有完全的清楚，不要担心——我们将在未来的博客中解决这个问题。)

``` javascript
class RangeIterator {
  constructor(start, stop) {
    this.value = start;
    this.stop = stop;
  }

  [Symbol.iterator]() { return this; }

  next() {
    var value = this.value;
    if (value < this.stop) {
      this.value++;
      return {done: false, value: value};
    } else {
      return {done: true, value: undefined};
    }
  }
}

// 返回一个新的从“开始”数到“结束”的迭代器。
function range(start, stop) {
  return new RangeIterator(start, stop);
}
``` 

在实际运行中看这段代码 ( <http://codepen.io/anon/pen/NqGgOQ> )。

这就是像是在 Java 或 Swift 语言里实现一个迭代器。它不是那么糟糕。但它也并不是那么简单。在这个代码中有没有任何错误？这就不好说了。它看起来完全不像我们想在这里模仿的原来的 for ( ; ; ) 循环：迭代器协议迫使我们抛弃了循环。

在这一点上，你可能会对迭代器不太热情。他们可能对使用来说很棒，但他们似乎很难实现。

你可能不会建议我们只是为了简单的创建迭代器，而在 JS 语言中引进一个复杂的新的控制流结构。但是，因为我们确实有生成器，我们能在这里使用它们吗？让我们试一试：

``` script
function* range(start, stop) {
  for (var i = start; i < stop; i++)
    yield i;
}
```
在这里看代码的具体运行 ( <http://codepen.io/anon/pen/mJewga> ) 。

上述的四行的生成器是对先前 range( ) 的二十三行的实现的一个直接替代，包括了整个 RangeIterator 类。这可能是因为生成器是迭代器。所有的生成器都有一个内置的对 .next() 已经 `[Symbol.iterator]( )`方法的实现。

不使用生成器来实现迭代器就像是被强迫用被动语气写一封很长的邮件。本来想简单地想表达你的意思，可能到最后你说的会变得相当令人费解。RangeIterator 是很长且怪异的，因为它必须不使用循环语法来描述一个循环的功能。生成器是答案。

我们还能如何使用生成器作为迭代器的能力？

- 使对象可迭代。只要写一个迭代器函数来一直调用 this，在其出现的地方生成每一个值。然后以该对象的 [Symbol.iterator] 方法来安装该生成器函数。  
- 简化数组构建函数。实现一个函数，每当其被调用就会返回一个数组，如下面的这个例子： 
 
``` javascript
// 将一维数组 'icons'分为长度为 'rowLength'的数组。
function splitIntoRows(icons, rowLength) {
  var rows = [];
  for (var i = 0; i < icons.length; i += rowLength) {
    rows.push(icons.slice(i, i + rowLength));
  }
  return rows;
}
```

生成器可以将代码缩短很多：

``` javascript
function* splitIntoRows(icons, rowLength) {
  for (var i = 0; i < icons.length; i += rowLength) {
    yield icons.slice(i, i + rowLength);
  }
}
```

在行为上唯一的不同之处在于，取代一次性计算所有的结果，并返回他们的一个数组，这里返回一个迭代器且其可以按需一个一个地计算结果。

- 特别大量的结果。你不能构建一个无穷大的数组。但是你可以返回一个生成器，其可以生成一个无限大的序列，同时每一个调用者都可以使用它不管他们需要多少个值。

- 重构复杂的循环。你有庞大而丑陋的函数吗？你是不是想将它分为两个简单的部分呢？生成器就是可以帮助你达成这一目标的成套的重构工具。当你面对一个复杂的循环，你可以将产生数据的代码抽取出来编程一个独立的生成器函数。然后改变循环为 for 循环 ( myNewGenerator(args) 的 var 数据)。

- 使用迭代的工具。ES6 不提供扩展的库来进行过滤，映射以及一般可以访问任意可迭代的数据集。但生成器是伟大的，你只需要用几行代码就可以构建你需要的工具。

举个例子说，假设你需要一个东西等同于 Array.prototype.filter，其是在 DOM NodeLists 上工作的，不只是一个数组。  
这用代码来实现就是小意思：

``` javascript
function* filter(test, iterable) {
  for (var item of iterable) {
    if (test(item))
      yield item;
  }
}
```  

这个就是为什么生成器如此有用吗？当然。他们是要实现自定义的迭代器的简单的方法。同时，迭代器在整个 ES6 中是用于数据和循环的新标准。

但是，这还不是生成器能做的事情的全部。这甚至有可能不是他们做的最重要的事情。

## 生成器和异步代码  

这里是一个 JS 代码，我写了一个 while 的 back 部分。

``` javascript
		   };
        })
      });
    });
  });
});
```

可能这看起来和你代码的一部分比较相像。 异步 API 通常情况下需要一个回调，这就意味着你做一些事情时要写一个额外的匿名函数。所以如果你有一部分代码做三件事情，而不是三行代码，你是在看三个缩进层次的代码。

这里有一些我已经写好的 JS 代码：  

``` javascript
}).on('close', function () {
  done(undefined, undefined);
}).on('error', function (error) {
  done(error);
});
```

异步 API 有错误处理的约定，但不是使用异常。不同的 API 有不同的约定。在他们中的大多数中，错误在默认情况下被默默地删除。其中有一些，即使是普通的圆满完成，在默认情况下都会被删除。

直到现在，这些问题都只是简单的转画为我们进行异步编程的代价了。我们已经开始接受异步代码了，他们只是看起来不是像同步代码那样美好和简单。

生成器提供了新的希望，可以不用这样做的。

Q.async( ) 是一个试验性的尝试。其使用迭代器来生成类似于同步代码的异步代码。  
举个例子：   

``` javascript
// 
function makeNoise() {
  shake();
  rattle();
  roll();
}

// Asynchronous code to make some noise.
// Returns a Promise object that becomes resolved
// when we're done making noise.
function makeNoise_async() {
  return Q.async(function* () {
    yield shake_async();
    yield rattle_async();
    yield roll_async();
  });
}
```

主要的区别在于，异步版本必须在每个需要调用异步函数的地方添加 yield 关键字。

在 Q.async 版本中添加一点小东西，如 if 语句或 try/ catch 块，与在普通同步版本中添加是完全相同的。相比于编写异步代码的其他方式，有种不是在学习一个全新的语言的感觉。

如果你已经远远得到了这些东西，你可能会喜欢上 James Long 的在这个话题上面的非常详细的[博客](http://jlongster.com/A-Study-on-Solving-Callbacks-with-JavaScript-Generators)。

所以生成器指出了一个更适合人类大脑的新的异步编程模型。这项工作正在进行中。除其他事项外，更好的语法可能有所帮助。

异步函数的提出，建立在双方的承诺和生成器的基础上，并从在 C＃类似的功能中采取灵感，这些都是 ES7 要做的事情。

## 我什么时候可以用这疯狂的东西？

在服务器端，我们现在可以在 io.js 中使用 ES6 生成器。  
如果你启用 --harmony  选项，我们在 Node 中可也已使用 ES6 生成器。

在浏览器中，现今只有火狐27版本以上以及谷歌浏览器39版本以上的支持 ES6 生成器。为了在现今的网络上面使用生成器，你将需要使用 Babel 或者 Traceur 来将你的 ES6 的代码翻译为网络友好的 ES5 代码。


一些重要的事件值得了解：生成器是由布伦丹·艾希首次在 JS 上实现的。布伦丹·艾希的设计是紧紧跟随由 Icon 启发的 Python 生成器。他们早在2006年就运用在火狐 2.0 版本上了。但是标准化的道路是崎岖不平的，而且语法和行为在这个过程中改变了很多。ES6 生成器是由编程黑客温格安迪在火狐浏览器和谷歌浏览器中实现的。这项工作是由 Bloomberg 赞助的。

## yield  

关于生成器还有更多的说法。我们没有包含 .throw() 和  .return() 方法，可选的参数 .next()，或 yield* 表达式语法。但我认为这个帖子已经很长了，且现在已经足够扑朔迷离了。像生成器本身，我们应该停下来休息一下。

“下集预告”：下周将讨论可以嵌入到你们每天所写的代码中的特性。请下周加入我们，让我们一起来看看 ES6 模板字符串。
