#深度探索 ES6: 代理

深度探索 ES6 是一个针对 ECMAScript 标准中添加到 JavaScript 编程语言新功能的系列， 简称为 ES6。

下面是我们今天要学习一些的知识：

```
var obj = new Proxy({}, {
  get: function (target, key, receiver) {
    console.log(`getting ${key}!`);
    return Reflect.get(target, key, receiver);
  },
  set: function (target, key, value, receiver) {
    console.log(`setting ${key}!`);
    return Reflect.set(target, key, value, receiver);
  }
});
```

第一个例子稍微有点复杂。我将在后面给出详细解释。现在，来检查一下刚才创建的对象：

```
> obj.count = 1;
    setting count!
> ++obj.count;
    getting count!
    setting count!
    2
```

执行上面的代码将会发生什么？ 我门获得了访问对象的属性，并重载了 “.”  操作符。

##如何实现上述操作？

最好的方式 是虚拟化。这是一个非常通用的技术来实现惊人的事情。下面给出它是如何工作的。

1 取一张图片。

![Alt text](./power-plant.jpg)

2 在图片内部画一个轮廓。

![Alt text](./power-plant-with-outline.png)

3 现在可以替换轮廓内部的图像，也可以替换轮廓外部的图像。用你完全意向不到的图片来替换。这里只有一个规则：向后兼容规则。在你替换后，要保证其余的人看到替换后的图不会觉得这幅图有什么改变。

![Alt text](./wind-farm.png)

你可能从跟计算机相关的电影中了解关于黑客的一些知识，像是 The Truman Show 和The Matrix 电影。

为了满足后向兼容规则，你的替换需要巧妙地设计。但是关键的部分是正确画出轮廓。

这里的轮廓，是一个 API 的边界。一个接口，接口定义了两部分的代码如何交互和一部分的代码对另一部分的期望。

所以一个系统接口的设计，就像是在图中画一个合适的轮廓一样。你知道你可以替换任意轮廓内外的图像，但是其余未被替换的图像不知道整幅图像的改变。

当系统中没有接口的时候，你可以创建一个接口。有史以来最酷炫的黑客软件涉及绘制 API 边界，并把接口植入已有的大工程中。

虚拟存储，硬件虚拟化， Docker， Valgrind， rr —— 这些项目不同程度涉及到在已经存在的系统中，操纵新的或者是意想不到的接口。在有些例子中，需要新的操作系统和新的硬件支撑来完成虚拟特性。

ES6 引入了 JavaScript 最根本虚拟化支持概念： 对象。

##什么是对象？

让我们来回想一下，我们是什么时候接触对象的。

![Alt text](./thinker.jpg)

这个问题对于我来说太难了！我到现在都还没有听到一个满意的定义。

很吃惊吗？定以基础概念总是很困难—— 就像定义欧几里得中的最初定义。 ECMAScript 有良好规范的语言，它定义对象为“类型对象的成员”。

稍后，规范又添加了“一个对象是属性的集合”来定义对象。这并不是坏事。如果你想有一个定义，那么我们就来做。稍后我们还会来讲解。

我之前说关于写 API 的一些事，你必须理解它。因此，在某种程度上，我已经向大家承诺如果我们通过学习这些知识有助于对于对象这个概念更加了解。

所以让我们来跟着 ECMAScript 标准协会的脚印来学习如何定义一个 API ，一个接口和 JavaScript 对象。我们需要什么样的方法？对象能做什么？

这取决于对象。 DOM 元素对象可以实现某些功能；AudioNode  对象可以实现其他的功能。但是所有的对象都可以共享一些基础功能：
<li>对象都有属性。 你可以通过 get  和 set  操作来获得和设置属性，还可以删除属性，和其他属性操作方式。</li>
<li>属性都有原型。这是继承 JS 中的工作。</li>
<li>有些对象有函数或者构造器功能。你可以直接调用它们。</li>

JS 程序中大部分处理对象都用到了属性，原型和函数。甚至是一个单元或者一个 AudioNode  对象都是通过调用继承属性的方法来访问。

所以当 ECMAScript 标准协会定义 14 个内部方法，即所有对象的通用接口，毫无疑问，他们所有工作的重点最终都集中在这三种基础构建上：属性，原型和方法。

关于定义的 14 个内部方法详细信息都在 ES6 标准版本的表格5和表格6当中，参见： http://www.ecma-international.org/ecma-262/6.0/index.html#table-5 。这里我只对部分方法做解释。 这个奇怪的双层中括号 [[ ]] 表示这个方法是内部方法，你不能像其他普通方法一样调用，删除，或者重写。

<li>obj.[[Get]](key, receiver)  —— 获得属性值。</li>
当给出 obj.prop 或者 obj[key] 就能直接调用该方法。

obj 是当前正被搜寻的对象；receiver  是我们正在搜寻属性的对象。有时候我们需要梭巡一些对象。 obj 可能是 receiver’s 原型链上的一个对象。

<li>obj.[[Set]](key, value, receiver)  —— 分配对象属性。</li>
当给出 obj.prop = value 或者 obj[key] = value 就能直接调用该方法。

在赋值式子中，像 obj.prop += 2，最先调用的是 [[Get]] 方法，接着调用 [[Set]] 方法。赋值操作同样适用于 ++ 和 - 。

<li>obj.[[HasProperty]](key) —— 测试是否有属性存在。</li>
当给出 key in obj 就能直接调用该方法。

<li>obj.[[Enumerate]]( ) —— 列出 obj 的可列举属性。</li>
当给出 for (key in obj)  就能直接调用该方法。

该方法返回一个可迭代的对象，通过 for-in 循环获得对象的属性名称。

<li>obj.[[GetPrototypeOf]]( ) —— 返回对象的原型。 </li>
当给出 obj.__proto__ 或者 Object.getPrototypeOf(obj) 就能直接调用该方法。

<li>functionObj.[[Call]](thisValue, arguments) —— 调用方法。</li>
当给出  functionObj( ) 或者 x.method( ) 就能直接调用该方法。

该方法是可选的，并不是所有的对象都有方法。

<li>constructorObj.[[Construct]](arguments, newTarget) —— 调用构造器。 </li>
当给出  new Date(2890, 6, 2) 类似的 JS 代码就能直接调用该方法。

该方法是可选的，并不是所有的对象都有方法。

newTarget 参数在之类中有发挥作用。我们将在未来的章节进行讨论。

或许你可以猜想其余七种方法是什么样。

整个 ES6 标准，任何一点语法或者内置函数关于对象的操作都依据这 14 个内部方法。 ES6 给对象的大脑描绘出一个清晰的轮廓。而代理所做的事就是用任意的 JS 代码代替这个标准的大脑。

当我们开始讨论重写这些内部方法的时候，请记住，我们是在讨论重写核心语法的行为，像 obj.prop ，内置函数像 Object.keys( ) ， 等等。

##代理（Proxy）

ES6 定义了新的全局构造器， Proxy 。Proxy 需要两个参数：一个目标对象和一个处理对象。下面给出关于代理 Proxy 的一个简单例子：

```
var target = {}, handler = {};
var proxy = new Proxy(target, handler);
```

让我们先把 handler  对象放在一边，这会主要关注如何 Proxy 和 target 的相关知识。

我可以告诉你在一个句子中如何实现代理。所有代理 proxy 的内部方法都将会转发给 target 。那就是，所谓的代理。也就是当调用 proxy. [[Enumerate]]( ) ， 它将会返回一个 target.[[Enumerate]]( )对象 。


让我们来试试。我们将来试试代理操作。 [[Set]]( ) 方法将会被调用。

```
proxy.color = "pink";
```

OK ， 接下来什么会发生？ proxy.[[Set]]( ) 会调用target.[[Set]]( ), 所以，在 target 对象上也应该有新的属性，是这样么？

```
> target.color
    "pink"
```

果真是这样。对于其他的内部方法效果也是一样的。代理所作的就是在 target 对象中实现同样的方法。

上面的例子给我们一个幻觉是代理对象 proxy 和目标对象 target 是一样的。但实际上并非如此： proxy !== target 。并且有时候 proxy 会做一些类型检查处理，而 target 不会。甚至当 proxy 的目标是一个 DOM 元素。举个例子， proxy 不会是一个真的 DOM 元素，所以像  document.body.appendChild(proxy) 代码会因为类型检查失败而无法执行。

## Proxy 处理程序

现在让我们返回到处理程序对象上。这就是使 proxy 变得有用的部分。

处理对象的方法可以重载代理 proxy 的任意内部方法。

举个例子，如果你想拦截所有试图给对象属性赋值的尝试，你可以通过定义 handler.set( ) 来解决。

```
var target = {};
var handler = {
  set: function (target, key, value, receiver) {
    throw new Error("Please don't set properties on this object.");
  }
};
var proxy = new Proxy(target, handler);

> proxy.name = "angelina";
    Error: Please don't set properties on this object.
```

关于完整的代理 proxy 处理程序参见：https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy#Methods_of_the_handler_object 。这里有 14 种方法，这 14 种内部方法都在 ES6 中有详细解释。

所有的处理程序方法都是可选的。如果一个内部的方法没有被处理程序截获，那么它就会把信息转发给 target ， 就像我们之前看到的例子。

##举例说明：“不可能”自动填充对象

我们已经知道代理 proxy 试图来做一些神奇的事，但是有些事是代理不可能完成的。

这里给出第一个例子。函数 Tree( ) 代码见下：

```
> var tree = Tree();
> tree
    { }
> tree.branch1.branch2.twig = "green";
> tree
    { branch1: { branch2: { twig: "green" } } }
> tree.branch1.branch3.twig = "yellow";
    { branch1: { branch2: { twig: "green" },
                 branch3: { twig: "yellow" }}}
```

注意到内部对象 branch1 ，branch2，和 branch3 如何在需要的时候神奇般地被创建了。 上述代码是合适吗，对吗？它怎么可能工作呢？

直到现在，上面代码都无法工作。但是代理只用几行代码就能搬到。我们只需要利用 tree.[[Get]]( ) 。如果你是喜欢挑战类型的人，那么你可能在阅读  tree.[[Get]]( ) 源码之前，想试着实现它。

![Alt text](./maple-tap.jpg)

下面给出我的解决方案：

```
function Tree() {
  return new Proxy({}, handler);
}

var handler = {
  get: function (target, key, receiver) {
    if (!(key in target)) {
      target[key] = Tree();  // auto-create a sub-Tree
    }
    return Reflect.get(target, key, receiver);
  }
};
```

注意到最后调用的 Reflect.get( ) 函数。在代理 proxy 处理程序中，已经被证明是一个非常普遍的需求，可以说“现在只需要通过默认的行为来删除 target 。” 因此 ES6 定义了一个新的 Reflect object 对象，该对象包含了默认的 14 种内部方法，这样你可以就可以实现上述目标了。

##举例说明：一个只读视图

我想我可能给读者造成一个错误的印象，那就是代理 proxy 易于使用。让我们来看看更多例子来确认是不是这样。

这次我们的赋值任务将会变得复杂：我们要实现一个函数：readOnlyView(object) ，该函数以任何对象为输入并且返回一个代理，代理的行为就仅仅像是没有变异能力的对象。所以，举个例子，它应该像这样：

```
> var newMath = readOnlyView(Math);
> newMath.min(54, 40);
    40
> newMath.max = Math.min;
    Error: can't modify read-only view
> delete newMath.sin;
    Error: can't modify read-only view
```

我们怎样能实现它呢？

第一步就是拦截所有的内部方法，阻止内部方法修改目标对象。这个有五个。

```
function NOPE() {
  throw new Error("can't modify read-only view");
}

var handler = {
  // Override all five mutating methods.
  set: NOPE,
  defineProperty: NOPE,
  deleteProperty: NOPE,
  preventExtensions: NOPE,
  setPrototypeOf: NOPE
};

function readOnlyView(target) {
  return new Proxy(target, handler);
}
```

通过上面的代码，可以阻止赋值运算，属性定义和通过只读视图的其他行为。

这个方案有什么漏洞吗？

最大的问题就是  [[Get]]  方法和其他方法可能导致可变的对象。因此虽然某个对象 x 是只读视图，但是 x.prop 可能是可变异的！这就是一个巨大的漏洞。

为了弥补这个漏洞，我们必须加入 handler.get( ) 方法：

```
var handler = {
  ...

  // Wrap other results in read-only views.
  get: function (target, key, receiver) {
    // Start by just doing the default behavior.
    var result = Reflect.get(target, key, receiver);

    // Make sure not to return a mutable object!
    if (Object(result) === result) {
      // result is an object.
      return readOnlyView(result);
    }
    // result is a primitive, so already immutable.
    return result;
  },

  ...
};
```

这样还不够。其他方法也需要上述代码，包括 getPrototypeOf  和 getOwnPropertyDescriptor 。

接下来还有一些问题。当一个 getter 或者 方法通过这种形式的代理调用，值传递到 getter 或者方法实际上是代理本身。但是像早前我们看到那样，一些访问和方法执行典型的类型检查，然而代理不执行类型检查。这里用代理来代替 target 对象是一种好的方式。你能弄清楚如何实现吗？

这一部分的知识告诉我们创建一个对象是很容易的，但是创建一个有直觉的代理却很难。

##零碎知识

<li>代理真的有那么好吗？</li>
他们非常有用当你想要观察或者记录对象的访问。他们对于调试很方便。测试框架能用来模仿真实的对象。

代理是有用的，如果你需要的只是普通对象能做到行为：就像是填充属性。

我讨厌提到这个，但是看代码进行哪一步最好的方式是用代理。。。具体来说，就是将代理处理程序对象封装到另一个代理中，该代理在每次处理程序方法的访问的时候登录到控制台。

代理可以用来限制对象的进入，就像 readOnlyView 的行为一样。这一类用法在应用代码中很少见，但是 Firefox 用代理内部实现不同区域边界的安全。并且代理是我们安全模型的关键部分。

<li>代理  ♥  WeakMap</li> 在我们 readOnlyView  的例子中，我们在每次对象访问的时候都创建了一个新的代理。当我们用 WeakMap 时就能节省很多空间了，所以很多时候一个对象访问 readOnlyView ，只需创建一次代理。

这是一用 WeakMap 的例子。

<li>可撤销的代理</li> ES6 还定义了其他函数， Proxy.revocable(target, handler)，该函数创建了一个代理，像 new Proxy(target, handler) ，不同之处在于该函数创建的代理可以撤销。（Proxy.revocable 返回一个 .proxy 属性和 .revoke 方法。）一旦对象呗取消，它将不再工作；所有内部方法都将消失。
<li>对象不变性</li>在确定的情况下， ES6 需要代理处理程序方法来报告结果，以此来看是否与目标对象状态一致。为了保持一致性，所有对象执行不变性规则。举个例子，一个代理不能声明不能扩展，除非它的 target 对象也是不能扩展的。

不变性的规则太复杂了，这里我们就不具体阐述。但是你如果看到这样的一条错误信息，像是 “proxy can't report a non-existent property as non-configurable” ，这就是原因。最可能的补救方法就是改变代理关于自身的报告。另一种可能方法就是使 target 变异来动态反映代理的报告。

##现在，什么是对象？

我认为现在剩下的就是：“一个对象是属性的集合。” “An Object is a collection of properties.”

在今天来看，我对于这个定义并不十分满意，即便该定义被认为是理所当然的。对于原型和可赎回的定义也同样不是十分满意。我认为这个词 “collection ” 太大了，鉴于代理的定义并不那么完善。然而代理的处理程序方法能够做任何事。它们能够返回随机结果。

为了阐述清楚对象能实现的行为，标准化方法，和添加虚拟化作为一个一流的特征， ECMAScript 标准协会已经扩大了对象能实现行为的可能性。

对象能够实现大多数的行为了。

或许关于“什么是一个对象”这个问题最诚实的答案来自于 12 个所需的内部方法。一个 JS 程序中的对象能实现 [[Get]] 操作和 [[Set]]  操作，等等。

我们已经对对象有更深的理解了吗？我不大确定！我们做了令人惊奇的事吗？是的。我们做了在 JS  以前从不可能的事。

##今天，我能使用代理吗？

不！只有 Firefox 支持代理，并没有 polyfill 。所以，随意尝试代理吧！创建一个大厅的镜子，似乎每个对象有成千上万的副本，所有类似的，和调试任何事情是不可能的！但是现在是时候了。你的不明智的代理代码仍然能逃离生产…。

代理最初在 2010 年由 Andreas Gal 实现，并由 Blake Kaplan 进行代码评审。然后标准协会重新设计了特征。 Eddy Bruel  在 2012 实现饿了新的规格。

我实现了 Reflect ，由  Jeff Walden 进行代码评审。在 Firefox Nightly 中—— 除了  Reflect.enumerate( ) ，现在仍未实现。

接下来，我们将谈论 ES6 最具争议性的特性，呈现它功能的人胜过 Firefox 实现它的人吗？所以下周请继续加入我们，Mozilla 工程师 Eric Faust 将深度讲解 ES6 类。