# 代理

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

执行上面的代码将会发生什么？ 我门拦截了对象的属性访问过程，重载了 “.”  操作符。

## 如何实现上述操作？

最好的方式是虚拟化。这是一个用来实现惊人的事情的非常通用的技术。下面给出它是如何工作的。

1.  取一张图片。

    ![Alt text](image/power-plant.jpg)

2. 在图片内部画一个轮廓。

    ![Alt text](image/power-plant-with-outline.png)

3. 现在可以替换轮廓内部的图像，也可以替换轮廓外部的图像。用你完全意向不到的图片来替换。这里只有一个规则：向后兼容规则。在你替换后，要保证其余的人看到替换后的图不会觉得这幅图有什么改变。

    ![Alt text](image/wind-farm.png)

你可能从像 《The Truman Show》 或 《The Matrix》 这类关于计算机科学的经典电影作品中看到过类似的 hack 方法，即把把一个人放到一个盒子里，而他周围的世界都被一个精心设计的假像所替代。

为了满足后向兼容规则，你需要巧妙地设计你的替换方法。但是关键的部分是如何正确画出轮廓。

这里所谓的轮廓是指一个 API 的边界。一个接口定义了两部分的代码，一是如何交互，另一个是一部分的代码对另一部分的期望。所以一个系统接口的设计，就像是在图中画一个合适的轮廓一样。你知道你可以替换任意轮廓内外的图像，但是其余未被替换的图像并不在意这种替换。

当系统中没有接口的时候，你可以创建一个接口。有史以来最酷炫的黑客软件涉及绘制 API 边界，并把接口植入已有的大工程中。

虚拟存储，硬件虚拟化， [Docker](https://en.wikipedia.org/wiki/Docker_%28software%29)， [Valgrind](http://valgrind.org/)， [rr](http://rr-project.org/) —— 这些项目不同程度地都会将新的接口引入到原有系统中。在某些情况下，要想这些新的接口能很好的工作甚至需要花费数年的时间，还需要操作系统新特性的支持，甚至需要新的硬件的支持。  

ES6 引入了虚拟化来支持 javascript 最核心的概念：对象。

## 什么是对象？

让我们来回想一下，我们是什么时候接触对象的。

    ![Alt text](image/thinker.jpg)

这个问题对于我来说太难了！我到现在都还没有听到一个满意的定义。

很吃惊吗？定义基础概念总是很困难的—— 就像定义欧几里得中的最初定义。 ECMAScript 是有良好规范的语言，它将对象定义为“类型对象的成员”。

稍后，规范又添加了“一个对象是属性的集合”来定义对象。这并不是坏事。如果你想有一个定义，那么这就是一个定义。稍后我们还会来讲解。

我之前说关于写 API 的一些事，你必须理解它。因此，在某种程度上，我已经向大家承诺如果我们学习这些知识将有助于对于对象这个概念更加了解。

所以让我们来跟着 ECMAScript 标准协会的脚步来学习如何定义一个 API ，一个接口和 JavaScript 对象。我们需要什么样的方法？对象能做什么？

这取决于对象。 DOM 元素对象可以实现某些功能；AudioNode  对象可以实现其他的功能。但是所有的对象都可以共享一些基础功能：  

- 对象都有属性。 你可以通过 get  和 set  操作来获得和设置属性，还可以删除属性，和其他属性操作方式。
- 属性都有原型。这是继承 JS 中的工作。
- 有些对象有函数或者构造器功能。你可以直接调用它们。

JS 程序中大部分处理对象都用到了属性，原型和函数。甚至是一个单元或者一个 AudioNode  对象都是通过调用继承属性的方法来访问。

所以当 ECMAScript 标准协会定义 14 个内部方法，即所有对象的通用接口，毫无疑问，他们所有工作的重点最终都集中在这三种基础构建上：属性，原型和方法。

关于定义的 14 个内部方法详细信息都在 ES6 标准版本的表格5和表格6当中，参见：[ http://www.ecma-international.org/ecma-262/6.0/index.html#table-5]( http://www.ecma-international.org/ecma-262/6.0/index.html#table-5) 。这里我只对部分方法做解释。 这个奇怪的双层中括号 [[ ]] 表示这个方法是内部方法，你不能像其他普通方法一样调用，删除，或者重写。

- obj.[[Get]]\(key, receiver)  —— 获得属性值。  
    当给出 obj.prop 或者 obj[key] 就能直接调用该方法。

    obj 是当前正被搜寻的对象；receiver  是我们正在搜寻属性的对象。有时候我们需要搜巡一些对象。 obj 可能是 receiver’s 原型链上的一个对象。

- obj.[[Set]]\(key, value, receiver)  —— 分配对象属性。   
    当给出 obj.prop = value 或者 obj[key] = value 就能直接调用该方法。

     在赋值式子中，像 obj.prop += 2，最先调用的是 [[Get]] 方法，接着调用 [[Set]] 方法。赋值操作同样适用于 ++ 和 - 。

- obj.[[HasProperty]]\(key) —— 测试是否有属性存在。  
    当给出 key in obj 就能直接调用该方法。

- obj.[[Enumerate]]\( ) —— 列出 obj 的可列举属性。  
    当给出 for (key in obj)  就能直接调用该方法。

    该方法返回一个可迭代的对象，通过 for-in 循环获得对象的属性名称。

- obj.[[GetPrototypeOf]]\( ) —— 返回对象的原型。   
    当给出 obj.__proto__ 或者 Object.getPrototypeOf(obj) 就能直接调用该方法。

- functionObj.[[Call]]\(thisValue, arguments) —— 调用方法。  
    当给出  functionObj( ) 或者 x.method( ) 就能直接调用该方法。

    该方法是可选的，并不是所有的对象都有方法。

- constructorObj.[[Construct]]\(arguments, newTarget) —— 调用构造器。  
    当给出  new Date(2890, 6, 2) 类似的 JS 代码就能直接调用该方法。

    该方法是可选的，并不是所有的对象都有此方法。

     newTarget 参数在子类中有发挥作用。我们将在未来的章节进行讨论。

或许你可以猜想其余七种方法是什么样。

整个 ES6 标准，任何一点语法或者内置函数关于对象的操作都依据这 14 个内部方法。 ES6 给对象的大脑描绘出一个清晰的轮廓。而代理所做的事就是用任意的 JS 代码代替这个标准的大脑。

当我们开始讨论重写这些内部方法的时候，请记住，我们是在讨论重写核心语法的行为，像 obj.prop ，内置函数像 Object.keys( ) ， 等等。

##  代理（Proxy）

ES6 定义了新的全局构造器 Proxy 。Proxy 需要两个参数：一个目标对象和一个处理对象。下面给出关于代理 Proxy 的一个简单例子：

```
var target = {}, handler = {};
var proxy = new Proxy(target, handler);
```

让我们先把 handler  对象放在一边，这会主要关注如何 Proxy 和 target 的相关知识。

我可以告诉你在一个句子中如何实现代理。所有代理的内部方法都将会转发给 target 。那就是，所谓的代理。也就是当调用 proxy. [[Enumerate]]\( ) ， 它将会返回一个 target.[[Enumerate]]\( )对象 。


让我们来试试。我们将来试试代理操作。 [[Set]]\( ) 方法将会被调用。

```
proxy.color = "pink";
```

OK ， 接下来什么会发生？ proxy.[[Set]]\( ) 会调用 target.[[Set]]\( ), 所以，在 target 对象上也应该有新的属性，是这样么？

```
> target.color
    "pink"
```

果真是这样。对于其他的内部方法效果也是一样的。代理所作的就是在 target 对象中实现同样的方法。

上面的例子给我们一个幻觉是代理对象 proxy 和目标对象 target 是一样的。但实际上并非如此： proxy !== target 。并且有时候 proxy 会做一些类型检查处理，而 target 不会。举个例子，即使 proxy 的目标是一个 DOM 元素， proxy 不会是一个真的 DOM 元素，所以像  document.body.appendChild(proxy) 代码会因为类型检查失败而无法执行。

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

代理的所有处理函数列表可参见：[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy#Methods_of_the_handler_object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy#Methods_of_the_handler_object) 。这里有 14 种方法，这 14 种内部方法都在 ES6 中有详细解释。

所有的处理程序方法都是可选的。如果一个内部的方法没有被处理程序截获，那么它就会把信息转发给 target ， 就像我们之前看到的例子。

## 举例说明：“不可能”自动填充对象

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

注意到内部对象 branch1 ，branch2，和 branch3 是如何在需要的时候神奇般地被创建的。 很方便，对吗？它怎么可能工作呢？

直到现在，上面代码都无法工作。但是代理只用几行代码就能搬到。我们只需要利用 tree.[[Get]]\( ) 。如果你是喜欢挑战类型的人，那么你可能在阅读  tree.[[Get]]\( ) 源码之前，想试着实现它。

![Alt text](images/maple-tap.jpg)

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

注意到最后调用的 Reflect.get( ) 函数。在代理 proxy 处理程序中，使用被代理对象的默认方法被证明是一种非常普遍的需求。 因此 ES6 定义了一个新的 Reflect object 对象，该对象包含了默认的 14 种内部方法，这样你可以就可以实现上述目标了。

## 举例说明：一个只读视图

我想我可能给读者造成一个错误的印象，那就是代理 proxy 易于使用。让我们来看看更多例子来确认是不是这样。

这次我们的赋值任务将会变得复杂：我们要实现一个函数：readOnlyView(object) ，该函数以任何对象为输入并且返回一个代理，代理的行为和被代理的对象一样，只是让被代理的对象变得不可修改。所以，举个例子，它应该像这样：

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

上面的代码通过只读视图阻止赋值运算，属性定义以及其他赋值行为。

这个方案有什么漏洞吗？

最大的问题就是  [[Get]]  方法和其他方法可能导致可变的对象。因此虽然某个对象 x 是只读的，但是 x.prop 可能是可修改的！这就是一个巨大的漏洞。

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

接下来还有一些问题。当通过这种形式的代理调用 getter 或者方法时，传递给 getter 或者方法的值实际上是代理本身。但是像早前我们看到那样，很多访问函数会执行类型检查，而代理对象却不能通过这样的类型检查。在这种情况下，用目标对象本身替换掉代理。你能弄清楚如何实现吗？

这一部分的知识告诉我们创建一个代理对象是很容易的，但是创建一个直觉正确的代理确是非常困难。

## 零碎知识

- 代理真的有那么好吗？  

    他们非常有用当你想要观察或者记录对象的访问。他们对于调试很方便。测试框架能用它们来模仿真实的对象。

    代理是有用的，如果你需要的只是普通对象能做到行为：就像是填充属性。

    我讨厌提到这个，但是看代码运行哪一步最好的方式是用代理。。。具体来说，就是将代理处理程序对象封装到另一个代理中，该代理在每次处理程序方法的访问的时候记录输出到控制台。

    代理可以用来限制对象的访问，就像 readOnlyView 的行为一样。这一类用法在应用代码中很少见，但是 Firefox 用代理内部实现不同区域边界的安全。并且代理是我们安全模型的关键部分。

- 代理  ♥  WeakMap  

    在我们 readOnlyView  的例子中，我们在每次对象访问的时候都创建了一个新的代理。把每次创建的代理缓存在 WeakMap 中可以节省大量的空间，因为无论一个对象被传递给 readOnlyView 多少次都只会为其创建一个代理。

    这是一用 WeakMap 的例子。

- 可撤销的代理
    
    ES6 还定义了其他函数， Proxy.revocable(target, handler)，该函数创建了一个代理，像 new Proxy(target, handler) ，不同之处在于该函数创建的代理可以撤销。（Proxy.revocable 返回一个 .proxy 属性和 .revoke 方法。）一旦对象被取消，它将不再工作；所有内部方法都将消失。

- 对象不变性
 
    在某些情况下， ES6 需要代理处理程序方法报告与目标对象状态一致的结果。这样做的目的是保证所有对象的不变性，包括代理对象本身。举个例子，一个代理不能声明不可扩展，除非它的 target 对象也是不可扩展的。

    不变性的规则太复杂了，这里我们就不具体阐述。但是你如果看到这样的一条错误信息，像是 “proxy can't report a non-existent property as non-configurable” ，这就是原因。最可能的补救方法就是改变代理的声明。另一种可能方法就是运行时修改目标对象来匹配代理的声明。

## 现在，什么是对象？

我认为现在剩下的就是：“一个对象是属性的集合。” “An Object is a collection of properties.”

在今天来看，我对于这个定义并不十分满意，即便我们默认把原型考虑在内，这也并不能让我感到满意。我认为这个词 “collection ” 太大了，基本没有定义清楚代理到底是什么。然而代理的处理程序方法能够做任何事。它们甚至能够返回随机结果。

 ECMAScript 标准协会通过清明地说明对象可以做什么，标准化这些方法，以及将虚拟化作为头等特征，已经极大扩大了这个可能的范围。

对象几乎已经是一切了。

或许可以用将 12 个必需的内部方法作为对 “什么是一个对象？” 这个问题的回答。一个 JS 程序中的对象能实现 [[Get]] 操作和 [[Set]]  操作，等等。

我们已经对对象有更深的理解了吗？我不大确定！我们做了令人惊奇的事吗？是的。我们做了在 JS  以前从不可能的事。

## 今天，我能使用代理吗？

不！只有 Firefox 支持代理，并没有 polyfill 。所以，随意尝试代理吧！创建一个大厅的镜子，似乎每个对象有成千上万的副本，它们非常相似，调试它们几乎是不可能的！但是现在是时候了。因为即使不成熟的代码放到了线上环境也不会有任何的风险。

代理最初在 2010 年由 Andreas Gal 实现，并由 Blake Kaplan 进行代码评审。然后标准协会重新设计了特征。 Eddy Bruel  在 2012 实现饿了新的规格。

我实现了 Reflect ，由  Jeff Walden 进行代码评审。除了 Reflect.enumerate( ) 没完成以外，Reflect 的所有内容本周末都会出现在 Firefox Nightly 中。

后面，我们还会谈论 ES6 最具争议性的特性，