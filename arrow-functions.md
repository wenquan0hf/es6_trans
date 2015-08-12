# 箭头函数

深度探索 ES6 是一个针对 ECMAScript 标准中添加到 JavaScript 编程语言新功能的系列， 简称为 ES6。

箭头函数（ Arrow Function ）很早就被应用在 JavaScript 中。第一版 JavaScript 教程中建议包装脚本并内嵌在 HTML 解释中。这将会阻止浏览器不支持 JS 时，以文本形式错误地显示你的 JS 代码。你最好这样写：

```
<script language="javascript">
<!--
  document.bgColor = "brown";  // red
// -->
</script>
```

老的浏览器会认为上面的代码是两种不支持的标签和解释；只有新的浏览器会认为是 JS 代码。

在你的浏览器中  JavaScript  引擎认为字符 `<!--` 作为注释的开始。不开玩笑地说，这真的是语言的一部分，并且直到现在，不仅仅是内联的顶部  ```<script>``` ，整个 JS 代码任意部分都属于语言的一部分。它甚至可以在节点处被执行。

凑巧的是，这种注释的风格是 ES6 首次标准化。但这不是我们谈论的箭头函数。

箭头序列 `-->` 表示单行注释。奇怪地是，在 HTML 中在字符串 `-->` 前面是注释部分，而在 JS 中字符串  `-->` 后面的部分表示注释。

令人感到奇怪的是，这个箭头只有当它出现在一行的开始才表示注释。这是因为在内容的其它部分， `-->` 表示 JS 操作符， 表示 ”跳转“操作符。

```
function countdown(n) {
  while (n --> 0)  // "n goes to zero"
    alert(n);
  blastoff();
}
```

这段代码真的会工作。循环运行直到 n 变为 0 。 这也不是 ES6 的新特性，但是一个常见特征的组合。你能想象发生了什么吗？像往常一样，这个谜题的答案可以找到堆栈溢出。
 
当然，这里也有小于或等于操作符 `<=`。 或许你还可以在 JS代码中发现更多的箭头函数。让我们停下来观察这些箭头函数，发现有个箭头函数消失了。


|箭头            |说明            |
|:---------------|:---------------|
| <\!--           | 单行注释   |
| -->              | ”跳转”操作 |
| <=              | 小于或等于操作符      | 
| =>              | ???             |


`=>`发生了什么呢？今天，让我们来寻找它。  

首先，让我们来讨论一些功能。

## 无处不在的函数表达式

JavaScript 的一个有趣特性是，任何时候你需要一个函数，你可以在函数右边加入运行代码。

举个例子，假设你试图告诉浏览器当用户点击一个特殊按钮后怎么处理。你可以输入这样的代码：

```
$("#confetti-btn").click(
```

jQuery’s .click( ) 方法需要一个参数：一个函数。毫无问题，你可以在函数右边这样输入：

```
$("#confetti-btn").click(function (event) {
  playTrumpet();
  fireConfettiCannon();
});
```

像上面方式来写代码对于我们来说很自然。所以很好奇在 JavaScript 流行以前，很多语言都还不具备这种编程风格。当然 在 1958 年时，Lisp  语言已经有了函数表达式，通常也被称为 lambda 函数。 但是像 C++， Python， C# 和Java 存在很多年了都还没有具备这种编程风格。

至少现在上述四种语言已经具备了 lambda 函数。较新的语言通常都已经内置有lambda 函数了。我们能有现在的 JavaScript 应该感谢早起的 JavaScript 开发者，他们创建了依赖于 lambdas 函数的函数库，导致这一特征被广泛采用。

这里有点略带忧伤，在我提及的所有语言中， JavaScript 关于 lambdas 函数的介绍稍微有点冗长。

```// A very simple function in six languages.
function (a) { return a > 0; } // JS
[](int a) { return a > 0; }  // C++
(lambda (a) (> a 0))  ;; Lisp
lambda a: a > 0  # Python
a => a > 0  // C#
a -> a > 0  // Java
```

## 一个新的箭头函数

ES6 提供了新的语法规则来描述函数。

```
// ES5
var selected = allJobs.filter(function (job) {
  return job.isSelected();
});

// ES6
var selected = allJobs.filter(job => job.isSelected());
```

当你仅仅只需要带有一个参数的函数，这个新的箭头函数语法简单地描述为：标识符 => 表达式 
( Identifier => Expression ) 。你可以跳过编辑函数名和返回值，以及一些大括号，小括号和分号。

（ 我个人非常感激这个特征。对于我来说，不用再次编辑函数很重要，因为我不可避免会输错函数名，这样我不得不再回头检查并且改正。 ）

在编写一个带有多个参数的函数时（或许没有参数，或许有默认参数，或者是前面提到的非结构化参数），你需要在参数列表的外面加一层括号。

```
// ES5
var total = values.reduce(function (a, b) {
  return a + b;
}, 0);

// ES6
var total = values.reduce((a, b) => a + b, 0);
```

我认为这样的书写那个时候看起来很美观。

箭头函数就像是在函数库上编写一样， 显得美观。实际上，[Immutable’s documentation](https://facebook.github.io/immutable-js/docs/#/)  很多例子都是用 ES6 来编写的，因此里面的很多例子都使用到了箭头函数。

箭头函数内除了可以有表达式外，还可以有一段语句。回想早期的例子：

```
// ES5
$("#confetti-btn").click(function (event) {
  playTrumpet();
  fireConfettiCannon();
});
```

这里是用 ES6 写的代码：

```
// ES6
$("#confetti-btn").click(event => {
  playTrumpet();
  fireConfettiCannon();
});
```

注意到一个箭头函数，用块来描述的时候可能没有自动返回一个值。我们可以用 return 声明来返回。

这里有个注意事项当用箭头函数来创建一个空对象时，要用括号括起来：

```
// create a new empty object for each puppy to play with
var chewToys = puppies.map(puppy => {});   // BUG!
var chewToys = puppies.map(puppy => ({})); // ok
```

不幸的是，一个空的对象 {} 和一个空块 {} 看起来是一样的。在 ES6 当中 ｛ 紧跟在箭头函数后面表示的是一个块的开始，而不是一个对象的开始。这样代码 puppy => {}  表示的是一个箭头函数，什么都不做，空操作。返回一个未定义。

更令人困惑的是，像 {key: value} 这样的对象声明看起来像是一个包含有标签的块声明。至少对你的 JavaScript 引擎来说是这样的。幸运地是，你可以用括号将 { 这个歧义字符括起来。

## 什么是 this 指针？

普通的函数和箭头函数还是存在细微的差别。箭头函数没有自己内部的 this 指针。在箭头函数中， this 指针是继承于其所在的作用域。

让我们在实际使用的过程中来搞明白这到底是什么意思。

在JavaScript 中 this 指针是怎么工作的？它的值又是来自哪里？这里没有简洁的答案。如果你认为这是很简单的，那是因为你对于 this 指针已经很熟悉了。

这一问题经常出现的一个原因是 function 函数自动获得 this 指针，不管你想要与否。你曾经写过这样的代码吗？

```
{
  ...
  addAll: function addAll(pieces) {
    var self = this;
    _.each(pieces, function (piece) {
      self.add(piece);
    });
  },
  ...
}
```

这里你所写的内部函数是  this.add(piece)。不幸地是，这个内部函数没有继承外部函数的 this 指针。这样内部的函数用 this 将会返回未定义。这个临时的变量 self 会调用外部的 this 到内部的函数来。（ 另外一种方式在内部函数中调用 .bind(this) 。其他方式都不是很令人满意。 ）

在 ES6 中， this 指针的 hacks 代码可以避免如果你遵循下面的规则：

- 用非箭头函数方法，将使用 object.method() 语法。这样函数将会从调用者那里获得一个有意义的 this 指针。
- 用箭头函数。

```
// ES6
{
  ...
  addAll: function addAll(pieces) {
    _.each(pieces, piece => this.add(piece));
  },
  ...
}
```

ES6 版本中，注意到 addAll  方法从调用者那里获得一个 this 指针。上面内部的函数是一个箭头函数，它从作用域处继承 this  指针。

作为奖励， ES6 通常提供了一个更短的方式来描述对象！ 因此上面的代码可以变得更简洁：

```
// ES6 with method syntax
{
  ...
  addAll(pieces) {
    _.each(pieces, piece => this.add(piece));
  },
  ...
}
```

比较上面的两种方式，可以发现用箭头函数描述可以不再输入 function 了。这是一个很好的想法。

普通函数和箭头函数还有一个小差异，那就是箭头函数不需要参数。当然， 在 ES6 中，你可能宁愿用使用其他参数或者默认参数。

## 箭头函数在计算科学的发展史

我们这里讨论一些关于箭头函数的实际应用。这里我会讨论一个用例： ES6 箭头函数作为一个学习工具，来发现一些深层次的计算本质。不管实际与否，你可以自己来判断。

早在 1936 年，Alan Church 和 Alan Turing 独立开发了具有强大计算功能的数学模型。Turing 称它的模型为一个机器，即其余人所称道的图灵机。Church  的模型被他称为  λ- 演算。（ λ 是一个希腊小写字母 lambda 。 ）这也是为什么 Lisp 语言也用 LAMBDA 单词来表示函数的原因，这也是今天我们称函数表达式为 “lambdas” 的原因。

但是什么又是 λ- 演算呢？什么又是计算模型的思想呢？

这里很难用几句话来解释，但是这里是我的一些尝试性解释： λ- 演算是最早的编程语言。它的目的不是为了设计编程语言——在此之后，直到十年或到二十年后的样子才有了存储程序计算机，但是相当简单，它只能执行一些你想要的纯粹数学表达式。 Church 想让他的模型能够证明一些概括性的计算。

并且他只需要一样东西在他的系统里就是：函数。

他的想法是多么的奇怪。没有对象，没有数组，没有数字，还没有 if 声明， while 循环 , 分号和赋值，还没有逻辑操作符，或者是事件，这对于 JavaScript 来说是可能的，只用函数来重新构建每种类型。

这里给出一个数学上的程序，用 Church’s  的 λ 表示法：

```
fix = λf.(λx.f(λv.x(x)(v)))(λx.f(λv.x(x)(v)))
```  

与上面等价的 JavaScript 函数可以这样来写：

```
var fix = f => (x => f(v => x(x)(v)))
               (x => f(v => x(x)(v)));
```

也就是说， JavaScript 包含了一个实现 λ- 演算的部分。 即 λ- 演算在JavaScript 中。
               
关于 Alonzo Church  和后续的研究者关于 λ- 演算做的研究，以及它揭示了 λ- 演算如何成为大多数语言中的一部分，这些内容超出了这一章节的范围。但是如果你对计算机科学的起源很感兴趣，或者你对一个语言最开始只有函数没有任何其他类型感兴趣，你可以花费你雨天的中午来阅读 [Church numerals](https://en.wikipedia.org/wiki/Church_encoding) 和  [fixed-point combinator](https://en.wikipedia.org/wiki/Fixed-point_combinator#Strict_fixed_point_combinator)，并且你还可以在你的 Firefox 控制台上写。由于 ES6 在箭头函数方面的优势， JavaScript 可以合理地声明自己的语言对于探索 λ- 演算是最好的语言。

## 我们什么时候能够用箭头函数？

ES6 的箭头函数是由我于 2013 年在 Firefox 中实现。 Jan de Mooij 使得箭头函数运行变得更快。感谢  Tooru Fujisawa 和 ziyunfei 提供补丁。

箭头函数也在微软的新版本中实现。他们也在 [Babel](http://babeljs.io/)，[Traceur](https://github.com/google/traceur-compiler#what-is-traceur)，和 [TypeScript](http://www.typescriptlang.org/) 得到实现，如果你对于在网上使用箭头函数很感兴趣。

在下一章节中我们又会讨论 ES6 的一个奇怪用法。我们将看到 typeof x  会返回一个全新的值。我们将会发问：到底什么时候是一个名称什么时候又是一个字符串呢？我们会努力思考这些问题。这些都令人感到新奇。所以请下周继续加入我们来深度探索 ES6 关于符号的新特征。
