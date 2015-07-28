# 深度探索 ES6: 非结构化赋值

深度探索 ES6 是一个针对 ECMAScript 标准中添加到 JavaScript 编程语言新功能的系列， 简称为 ES6。

编者注解：一个早期 ES6 的版本是由 Firefox 开发工具的工程师尼克•菲茨杰拉德完成，最初以 ES6 非结构化赋值出现在尼克的博客中。

## 什么是非结构化赋值？

非结构化赋值允许你通过使用语法规则将一个数组或对象的属性分配给变量，类似于数组或对象文本。这个语法规则非常简单，同时还表现出比传统的属性访问更清晰。

如果没有非结构化赋值，你可能运用下面的方式访问数组中的前三个项目：

```
var first = someArray[0];
var second = someArray[1];
var third = someArray[2];
```

当有了非结构化赋值，你可以运用与上面功能等价的非结构化赋值代码，使得代码更加简洁和可读：

```
var [first, second, third] = someArray;
```

SpiderMonkey ( Firefox 提供的 JavaScript 引擎 ) 提供支持大部分的非结构化，但是并不是全部。追踪 SpiderMonkey 的支持的非结构操作参见：[https://bugzilla.mozilla.org/show_bug.cgi?id=694100](https://bugzilla.mozilla.org/show_bug.cgi?id=694100) 。

## 非结构化数组和迭代器

我们已经看到了基于数组的非结构化赋值的例子。语法的一般形式是:

```
[ variable1, variable2, ..., variableN ] = array;
```

这将会从数组 array 中对应的项目值赋值给变量1 （ variable1 ） 到变量N （ variableN ）。与此同时，如果你想定义你自己的变量，你可以增加 var , let , 或者 const 在赋值语句的最前面。  
不像普通的 字符串那样，模板字符可以覆盖多行：

```
ar [ variable1, variable2, ..., variableN ] = array;
let [ variable1, variable2, ..., variableN ] = array;
const [ variable1, variable2, ..., variableN ] = array;
```

事实上，用变量 ( variable ) 来描述是不恰当的，因为你还可以用深层次的嵌套模式:  

```
var [foo, [[bar], baz]] = [1, [[2], 3]];
console.log(foo);
// 1
console.log(bar);
// 2
console.log(baz);
// 3
```

此外，你可以在非结构化赋值中跳过一些项目，像下面的代码一样：

```
var [,,third] = ["foo", "bar", "baz"];
console.log(third);
// "baz"
```

并且你可以通过“休息”模式，获得数组中所有后续的项目:

```
var [head, ...tail] = [1, 2, 3, 4];
console.log(tail);
// [2, 3, 4]
```

当你访问数组中的项目时，产生越界或者访问不存在，你会得到同样的索引结果：未定义的。

```
console.log([][0]);
// undefined

var [missing] = [];
console.log(missing);
// undefined
```

注意到基于数组赋值模式的非结构化赋值通常可以运用在任意的迭代器上：

```
function* fibs() {
  var a = 0;
  var b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

var [first, second, third, fourth, fifth, sixth] = fibs();
console.log(sixth);
// 5
```

## 非结构化对象赋值  

非结构化对象操作允许你将变量绑定到对象的不同属性上。你首先指定将要被绑定的属性，紧跟着将要被赋予绑定值的变量。

```
var robotA = { name: "Bender" };
var robotB = { name: "Flexo" };

var { name: nameA } = robotA;
var { name: nameB } = robotB;

console.log(nameA);
// "Bender"
console.log(nameB);
// "Flexo"
```

当属性和变量名称是相同时，这是一个有用的语法快捷方式：

```
var { foo, bar } = { foo: "lorem", bar: "ipsum" };
console.log(foo);
// "lorem"
console.log(bar);
// "ipsum"
```

并且像非结构化数组赋值一样，你可以嵌套和结合深层次的非结构化：

```
var complicatedObj = {
  arrayProp: [
    "Zapp",
    { second: "Brannigan" }
  ]
};

var { arrayProp: [first, { second }] } = complicatedObj;

console.log(first);
// "Zapp"
console.log(second);
// "Brannigan"
```

当你非结构化赋值的属性没有被定义，你能得到值：未定义：

```
var { missing } = {};
console.log(missing);
// undefined
```

一个潜在的陷阱你应该知道的是当你使用非结构化对象赋值来分配的变量,但是你没有声明它们：

```
{ blowUp } = { blowUp: 10 };
// Syntax error
```

这将发生因为 JavaScript 语法告诉引擎去解释任意的以 { 开头的块声明。（举个例子， { console } 是一个有效的块声明）。 解决方案是把整个表达式的用括号括起来:

```
({ safe } = {});
// No errors
```

## 对非对象、数组或迭代类型进行非结构化赋值

当你试图运用非结构化在 null 或者 undefined 上时，你将会得到一个类型错误的值：

```
var {blowUp} = null;
// TypeError: null has no properties
```

然而，你可以运用非结构化在其它原始的类型中，比如：布尔类型，数字，和字符串，和未定义 undefinded 类型中：

```
var {wtf} = NaN;
console.log(wtf);
// undefined
```

这可能是意想不到的，但在进一步检查中，其实原因也很简单。当你运用对象赋值的模式时，被非结构化的值需要被对象支配。大多数的类型能够转换成对象类型，但是 null 和 undefined 不能转换。当你用数组赋值模式时，被去结构化的值必须是迭代器的形式。  

## 默认值

当你非结构化的属性值未被定义时，你还可以提供默认值：

```
var [missing = true] = [];
console.log(missing);
// true

var { message: msg = "Something went wrong" } = {};
console.log(msg);
// "Something went wrong"

var { x = 3 } = {};
console.log(x);
// 3
```

(编者注释：这个默认值特性现在已经在 Firfox 中实现，但是只支持前两种形式，不包括第三种。参见bug 932080 : [https://bugzilla.mozilla.org/show_bug.cgi?id=932080](https://bugzilla.mozilla.org/show_bug.cgi?id=932080) )

## 非结构化的实际应用

### 参数定义

作为开发者，我们通常公开一些 API 通过接受一个有多个属性的对象作为一个参数，取而代之强制我们的 API 调用者去记住许多个别参数的顺序。我们可以利用去结构化来避免重复这个有单一属性的对象，当我们要引用其中的一个属性时：

```
function removeBreakpoint({ url, line, column }) {
  // ...
}
```

这是一个来自 Firefox 开发工具的简化真实世界的代码（ 在 JavaScript 中被实现 ）。我们发现这种写法很受欢迎。

### 配置对象参数

扩展前面的示例，我们也可以用非结构化赋值给对象的属性设置默认值。当我们有一个负责提供配置信息的对象，该对象的属性值已经有合理的默认值，用非结构化给对象设置默认值就显得很有帮助了。举个例子， jQuery 的 ajax 函数将一个配置对象作为它的第二个参数，并且被写为：

```
jQuery.ajax = function (url, {
  async = true,
  beforeSend = noop,
  cache = true,
  complete = noop,
  crossDomain = false,
  global = true,
  // ... more config
}) {
  // ... do stuff
};
```

这样做避免了对于这个配置对象的每个属性操作重复： var foo = config.foo || theDefaultFoo。

（ 编者注解：不幸地是，对象的默认值这种速写语法在 Firefox 中还未实现。我知道，我们已经用了一些段落来学习关于非结构化赋值的知识。最新更新参见 [bug 932080]( https://bugzilla.mozilla.org/show_bug.cgi?id=932080) ）


### ES6 的迭代协议

ECMAScript 6 还定义了一个迭代协议，在这里我们主要讨论早起的迭代协议。当迭代 Map 类型时，你得到一些列的 [key, value]  对。我们可以通过非结构化这些对，来获得键和值：

```
var map = new Map();
map.set(window, "the global");
map.set(document, "the document");

for (var [key, value] of map) {
  console.log(key + " is " + value);
}
// "[object Window] is the global"
// "[object HTMLDocument] is the document"
```

只在键上面迭代：

```
for (var [key] of map) {
  // ...
}
```

或者只在值上面迭代：

```
for (var [,value] of map) {
  // ...
}
```

### 多个返回值

尽管多个返回值没有内置到语言本体中，但是也并不需要将返回值内置，因为你可以返回一个数组或者非结构化结果：

```
function returnMultipleValues() {
  return [1, 2];
}
var [foo, bar] = returnMultipleValues();
```

或者，你也可以用一个对象为容器和指定返回值：

```
function returnMultipleValues() {
  return {
    foo: 1,
    bar: 2
  };
}
var { foo, bar } = returnMultipleValues();
```

这两种模式最终比持有临时容器要好:

```
function returnMultipleValues() {
  return {
    foo: 1,
    bar: 2
  };
}
var temp = returnMultipleValues();
var foo = temp.foo;
var bar = temp.bar;
```

或者使用连续地传递模式：

```
function returnMultipleValues(k) {
  k(1, 2);
}
returnMultipleValues((foo, bar) => ...);
```

### 导入  COMMONJS 模块

仍然不想用 ES6 模块？还想用 COMMONJS 模块？没问题！当导入一些 COMMONJS 模块 X，这是很显然 X 导入一些比你打算使用的函数更多的函数。通过非结构化赋值的方式，你可以显式的导入你需要用到的函数，从面避免命名空间的污染：

```
const { SourceMapConsumer, SourceNode } = require("source-map");
```

## 结论

因此，正如你所看到的，非结构化赋值在许多场景中是有用的。在 Mozilla  我们已经有了很多关于非结构化赋值的经验。多年前，Lars Hansen 就已经将 JS 非结构化赋值引入到了 opera 中，Brendan Eich 稍后也将在 Firefox 中增加非结构化赋值。它会被引入在 Firefox 2 版本中。因此我们知道非结构化在你每天使用的语言当中，安静地使你的代码变得简洁和清晰。

五周以前我们说过 ES6  将会改变你写 JavaScript 的方式。这种特性是我们想要的：简单的改进能够被一次性学习。综上所述，最以革命的方式进化终影响你在每个项目的工作。

更新非结构化赋值来遵守ES6一直是一个团队努力的结果。尤其感谢 Tooru Fujisawa (arai) and Arpad Borsos (Swatinem)  的杰出贡献。

支持非结构化赋值的 Chrome 当前正在开发与此同时其他浏览器无疑也会增加支持非结构化赋值操作。现在来说，如果你想在 Web 上运用非结构化，你需要用到 [Babel](http://babeljs.io/) 或者 [Traceur](https://github.com/google/traceur-compiler#what-is-traceur) 。

再次感谢 Nick Fitzgerald 提供本周的文章。

下周，我们将会涉及或多或少关于 JS 的新特性——构建块基础的语言知识。你会在乎吗？略短的语法你可以感到兴奋吗？我非常相信答案是肯定的。下周继续加入我们，我们将会深度学习 ES6 箭头函数。

Jason Orendorff

深度探索 ES6 编辑者