# 迭代器和 for-of 循环

深度探索 ES6 是一个针对 ECMAScript 标准中添加到 JavaScript 编程语言新功能的系列， 简称为 ES6。

你是如何遍历数组中的元素的？ 二十年前，当 JavaScript 被发布的时候，你可能是这么做的：

``` c
for (var index = 0; index < myArray.length; index++) {
  console.log(myArray[index]);
}
```

从 ES5 开始，你可能使用内置的 forEach 方法。  

``` javascript
myArray.forEach(function (value) {
  console.log(value);
});
```

这样是比较短一点，但是其也有一个小缺陷：你不能通过使用一个 break 语句以退出循环或者使用 return 语句从封闭函数中返回。

如果在数组元素上只有一个 for 循环的话肯定是很好的。

但是，如果是一个 for－in 循环的话，那是怎样的呢？  

``` javascript
for (var index in myArray) {    // don't actually do this
  console.log(myArray[index]);
}
```

这将是一个很糟糕的想法，原因如下： 
 
- 在这段代码里面，分配给索引的值为字符串“0”，“1”，“2”，等等，而不是真正的数字类型的数字。这样使用是不方便，不符合原意的，因此你可能不希望使用字符串运算 ( “2”+1==“21” )。  
- 循环体除了访问数组中的每个元素外，还会访问数组的 expando 属性。举个例子，如果你的数组拥有的属性 myArray.name，则循环除了访问数组本身的元素外还会访问该属性值，访问该属性时 index = ”name“。甚至是数组原型链上的属性也会被访问到。
- 最令人惊讶的是，在某些情况下，这种代码可以以任意顺序遍历数组元素。

总之，for-in 被设计用来与以字符串作为键的普通的旧的对象工作。对于数组而言，它不是那么有效。

## for-of 循环的威猛之处  

记得上周我承诺 ES6 不会破坏你已经写好的代码。 那么，数以百万计的网站依赖于 for-in-yes 的语义，甚至其在数组上的用法。 因此，“修复” for-in 在数组中的用法是没有任何问题的。 对于 ES6，唯一重要的改进就是添加新的语法。

在这里，它是：

``` javascript
for (var value of myArray) {
  console.log(value);
}
```

嗯，这看起来并没有很令人印象深刻，不是吗？好了，我们将看到 for-of 是否有些巧妙的技巧。 现在，只需要注意的是：  

- 这是对数组元素进行循环的最简洁最直接的语法，
- 其避免了 for-in 的所有的缺点，
- 不同于 forEach( )，其可以配合 break， continue 以及 return 一起使用。
 
for-in 循环是在对象的属性上进行循环。  

for-of 循环是在数据上进行循环，如数组中的值。  

但是这些并不是全部。 

## 其他的集合也支持 for-of  

for-of 不仅仅在数组上使用。其也在大多是与数组类似的对象上使用，比如 DOM NodeListS。

其也适用于在字符串，其将字符串作为 Unicode 字符序列进行处理：

``` javascript
for (var chr of "😺😲") {
  alert(chr);
}
```

其也适用于 Map 以及 Set 对象。

。可能你还没有听说过 M哦，我很抱歉ap 和 Set 对象。 好吧，他们是新的东西，是在 ES6 中才出现的。 我们将在后面找时间给他们做一个整体的介绍。如果你已经在其他语言中使用过 map 和 set 了，这应该不是一个很大的惊喜了。

举例来说， 一个 Set 对象在消除重复的时候很好用：  

``` javascript
// 从一个单词数组中生成一个集合。
var uniqueWords = new Set(words);
```

一旦你获得了一个集合，你可能愿意遍历它的内容。简单例子如下：  

``` javascript
for (var word of uniqueWords) {
  console.log(word);
}
```

一个 Map 有略微的不同： 其数据是由一个键－值对组成。所以，你要使用解构的方法来解压键和值以使得他们成为两个独立的变量：

``` javascript
for (var [key, value] of phoneBookMap) {
  console.log(key + "'s phone number is: " + value);
}
```  

解构也是 ES6 的一个新的功能。其也是我们在博客中将要讨论的一大话题。我应该要将这些都记录下来。

到现在为止，你应该获得这样的直观感受：JS 已经有了相当多的不同的集合类， 甚至更多的还在开发中。for-of 被设计为你使用的循环语句中的主力循环语句。

for-of 在普通的旧的对象上不能正常工作，但是如果你想要在一个对象的属性上进行循环的话，你可以使用 for–in ( 本来就是它的功能 ) 或者内置的 Object.keys( ) :

``` javascript
// 将对象的自己的枚举属性转储到控制台
for (var key of Object.keys(someObject)) {
  console.log(key + ": " + someObject[key]);
}
```

## 在底层  

“能工摹形，巧匠窃意。”—— 巴勃罗·毕加索

ES6 的一个正火热的主题是：被添加到语言的新功能的推出毫无进展。新功能的大多数都已经在其他语言中被尝试并被证实为有效的。

for-of 循环，在其他语言也有类似的循环语句，比如，C++, Java, C#, 和 Python。和这些语言一样，其工作基于语言提供的几种不同的数据结构以及其标准库。但是，这仍然是语言的一个拓展点。

就像在其他语言中的 for/foreach 语句，for-of 完全在方法调用方面起作用。我们经常谈论的，比如数组，map，集合以及其他的一些对象，他们都有一个共同点，即他们都有一个迭代的方法。

同时，还存在另一种对象，它也有一个迭代的方法：任何你想要的对象。

正如你可以对任何一个对象添加一个 myObject.toString( ) 方法后，JS 就 突然就知道了应该如何将该对象转换为一个字符串一样。你可以对任何一个对象添加一个 `myObject[Symbol.iterator]( ) `方法，JS 突然就知道了应该如何在该对象上进行循环迭代。

举例来说，假设你正在使用 jQuery ，虽然你非常喜欢 .each( ) ，但是你也想让 jQuery 对象与 for-of 很好的一起工作。下述就是怎样去让这两个很好的一起工作：

``` javascript
// 既然 jQuery 对象和数组比较类似，
// 将数组的迭代方法用在 jQuery 对象上。
jQuery.prototype[Symbol.iterator] =
  Array.prototype[Symbol.iterator];
```

好吧，我知道你在想什么。这 [Symbol.iterator] 语法看上去似乎不可思议。这到底是怎么回事呢？我们从方法的名称开始说起。标准委员会可以只称这种方法为 .iterator( )。但是，你现有的代码可能已经有一些对象有 .iterator( ) 方法，这可能会使事情变得相当混乱。所以标准委员会使用一个符号，而不是字符串，作为此方法的名称。

符号在 ES6 中也是新的。随后，我们会告诉你关于他们的全部内容——你猜对了——在未来的博客文章中。现在，所有你需要知道的是，该标准可以定义一个全新的符号，比如 Symbol.iterator，同时它保证不会与任何现有代码产生冲突。需要权衡的是，这样的语法有点怪异。但对于其多功能的新特性和出色的向后兼容性，这只是一个很小的代价。

一个具有 `[Symbol.iterator]( )` 方法的对象被称为可迭代的。在后续的几周内，我们将看到迭代对象的概念被用于整个语言，不仅仅用于 for-of 中，也用于 Map 和 Set 构造函数，赋值解构以及新的传播算子中。

## 迭代器对象

现在，有一个机会，你将永远不用从零开始实现自己的迭代器对象。我们将在下周解释原因。但为了完整性，让我们来看看 iterator 对象是长什么样的。 ( 如果您跳过这一整章节，你可能主要会缺少一点技术细节。)

一个 for-of 循环通过调用集合上的 `[Symbol.iterator]( )` 方法进行启动。这个操作将会返回一个新的迭代对象。一个迭代对象可以是具有 .next( ) 方法的任意对象。 然后，for-of 循环将会反复调用此方法，即每次循环都要调用一次。举例来说，下述应该是我能想到的最简单的迭代器对象：

``` javascript
var zeroesForeverIterator = {
  [Symbol.iterator]: function () {
    return this;
  },
  next: function () {
    return {done: false, value: 0};
  }
};
```

每次 .next() 方法被调用都会返回同样的结果，以告诉 for-of 循环以下两个信息：( a ) 我们还没有结束迭代；( b ) 下一个值为 0。这就意味着 for ( zeroesForeverIterator 的值 )｛｝将是一个无限循环。当然，一个典型的迭代器不会这么的微不足道。

这种迭代器的设计，及其 .done 和 .value 属性，是与其他语言中的迭代器的工作原理是完全不同的。在 Java 语言中，迭代器有 .hasNext( ) 和 .next( ) 方法。在 Python 语言中, 他们只有一个 single .next( ) 方法以在没有更多值的时候抛出停止迭代 ( StopIteration ) 的异常。 但是这三种设计本质上都返回相同的信息。

一个迭代器对象能够可选择地实现 .return( ) 和 .throw(exc) 方法。for–of 循环能通过调用 .return() 方法来处理过早退出循环的情况。这种过早退出的情况可能是由于出现异常，中断或者 return 语句。如果迭代器需要做一些清理工作或者是释放出其正在使用的一部分资源，其能够通过实现 .return( ) 以达到这一目的。绝大多数的迭代器对象不需要实现这一方法。.throw(exc) 可能更是一个特例： for–of 基本不需要调用这个方法。但是我们下周将会学习更多关于这个方法的内容。

现在，我们已经知道了所有的细节，我们可以采取一个简单的 for-of 循环并且在底层方法调用方式进行重写。

首先，for-of 循环：  

``` javascript
for (VAR of ITERABLE) {
  STATEMENTS
}
```

这是一个大致相等的，使用底层方法和一些临时变量的例子：  

``` javascript
var $iterator = ITERABLE[Symbol.iterator]();
var $result = $iterator.next();
while (!$result.done) {
  VAR = $result.value;
  STATEMENTS
  $result = $iterator.next();
}
```

上述代码没有说明怎样处理 .return( )。我们可以补充一些内容，但是我认为这样的话会掩盖正在发生的事情而不是使其变得更加明白易懂。 for-of 是易于使用的，但是很多更深层的东西需要探讨。


## 我什么时候才能开始使用？

当前所有的 Firefox 发行版本都支持 for-of 循环。 要使得其被 Chrome 支持，你需要使用 `chrome://flags` 并且激活“实验性的 JavaScript”。其在微软的浏览器斯巴达也可以运行，但不能在 IE 浏览器的 shipping 版本中运行。如果您想在网络上使用这种新语法，你需要支持 IE 和 Safari 浏览器，你可以使用一个像通天塔或谷歌的 Traceur 的编译器来将你的 ES6 的代码转换为 Web 友好的 ES5 的代码。

在服务器上，你并不需要一个编译器，你今天就可以在 io.js ( 或者 Node 以 --harmony 选项启动 ) 上开始使用 for-of。

( 更新：以前被忽视但仍需要一提的是，for-of 在默认情况下是被 Chrome 浏览器禁用的。 )
 
## { 结束 : 真的 }

好了，我们今天结束了，但是我们仍然不能完全掌握 for-of 循环。

在 ES6 中，有新一类的对象能够与 for-of 完美地配合使用。 我没有提到它，是因为它是下周博客的话题。我觉得这个新功能是 ES6 中最神奇的事情。如果你还没有在 Python 和 C＃语言中遇到，你可能最初觉得它是多么令人难以置信。但它是写迭代器的最简单的方法，它在重构时是很有用的，它可能改变了我们无论在浏览器还是在服务器上编写异步代码的方式。因此，下周加入我们，让我们看看 ES6 生成器并进行深入探讨。

