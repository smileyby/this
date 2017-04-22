JavaScript中的this
=====================

当有人问你JavaScriot有什么特点的时候，你可能立马想到了单线程、事件驱动、面向对象等一堆词语，但是如果真的让你解释一下这些改变，可能真解释不清楚。有句话这么水：“如果你不能像一个6岁小孩解释清楚一个东西，那么你自己也不懂这个东西。”这句话或许有些夸张，但是极其有道理。个人觉得，如果需要掌握一门语言，掌握它的API只是学了皮毛，这门语言的精髓才是重点。提及JavaScript的精髓，this、闭包、作用域链、函数是当之无愧的。这门语言正是因为这几个东西变得魅力无穷。

最近被同时问及关于this的问题，突然发现自己对this也是半懂不懂的状态。所以决定好好的了解下this这个神奇的东东。

## JavaScript中的this

原文链接:[all this](http://bjorn.tipling.com/all-this)

JavaScript中很多时候会用到this，下面详细介绍每一种情况。这里我们首先介绍下宿主环境这个改变。一门语言在运行的时候，需要一个环境，叫做宿主环境。对于JavaScript，宿主环境最常见的是web浏览器，浏览器提供了一个JavaScript运行环境，这个环境里面，需要提供一些接口，好让JavaScript引擎能够和宿主环境对接。JavaScript引擎才是真正执行JavaScript代码的地方，常见的引擎有V8（目前最快JavaScript引擎、google生产）、JavaScript core。

JavaScript引擎主要做了下面几件事：

*	（1）一套与宿主环境相连接的规则；
*	（2）JavaScript引擎内核（基本语法规范、逻辑、命令和算法）；
*	（3）一组内置对象和API；
*	（4）其他约定。

但是环境不是唯一的，也就是JavaScript不仅仅能够在浏览器里面跑，也能够在其他的浏览器里面跑，也能够在其他提供了宿主环境的程序里面跑，最常见的就是nodejs。同样最为一个宿主环境，nodejs也有自己的JavaScript引擎-V8。根据官方的定义Node.js is a platform built on Chrome’s JavaScript runtime for easily building fast, scalable network applications。

### global this

*	（1）在浏览器里，在全局范围内，this等价于window对象

```js

<script type="text/javascript">
	console.log(this === window); // true
</script>

```

*	（2）在浏览器里，在全局范围内，用var声明一个变量和给this或者window添加属性是等价的。

```js

<script type="text/javascript">
	var foo = "bar";
	console.log(this.foo); // logs "bar"
	console.log(window.foo); // logs "bar"
</script>

```

*	（3）如果你在声明一个变量的时候没有使用var或者let（ECMAScript 6），你就会在给全局的this添加或者改变属性值。

```js

<script type="text/javascript">
	foo = "bar";
	function testThis(){
		foo = "foo";
	};
	
	console.log(this.foo); // logs "bar"
	testThis();
	console.log(this.foo); // logs "foo"
</script>

```

*	（4）在node环境里，如果使用REPL（Read-Eval-Print Loop，简称REPL：读取-求职-输出，是一个简单的，交互的编程模式），this并不是最高级的命名空间，最高级的是global。

```js

> this
{ ArrayBuffer: [Function: ArrayBuffer],
  Int8Array: { [Function: Int8Array] BYTES_PER_ELEMENT: 1 },
  Uint8Array: { [Function: Uint8Array] BYTES_PER_ELEMENT: 1 },
  ...
> global === this
true

```

*	（5）在node环境里，如果执行一个js脚本，在全局范围内，this以一个空对象开始最为最高级的命名空间，这个时候，它和global不是等价的。

```

test.js脚本内容：
 
console.log(this);
console.log(this === global);
 
REPL运行脚本：

$ node test.js
{}
false

```

*	（6）在node环境里，在全局范围内，如果你用REPL执行一个**脚本文件**，用var声明一个变量并不会再浏览器里面一样将这个变量添加给this。

```

test.js

var foo = "bar";
console.log(this.foo);

$ node test.js

undefined 

```

* （7）如果你不是用REPL执行脚本文件，而是直接**执行代码**，结果和在浏览器里面是一样的

```

var foo = "bar";
this.foo
bar

global.foo
bar

```

*	（8）在node环境里，用REPL运行**脚本文件**的时候，如果在声明变量的时候没有使用var或者let，这个变量会自动添加到global对象，但不会自动添加给this对象。如果是直接执行代码，则会同时添加给global和this

```

test.js

foo = "bar";
console.log(this.foo);
console.log(global.foo);

node test.js
undefined
bar

```

上面的八种情况可能大家已经绕晕了，总结起来就是：在**浏览器里面**this是老大，它等价于window对象，如果你声明一些全局变量（不管在任何地方），这些变量都会作为this的属性。在node里面，有两种执行JavaScript代码的方式，一种是直接执行写好的JavaScript文件，另一种是直接在里面执行一行行代码。对于直接运行一行行代码的方式，global才是老大，this和它是等价的，在这种情况下，和浏览器比较类似，也就是声明一些全局变量会自动添加给global，顺带也会添加给this。但是在node里面直接脚本就不一样了，你生命的群居变量不会自动添加到this，但是会添加到global对象。所以相同点是，在全局范围内，全局变量中就属于老大的。

http://www.cnblogs.com/yuanzm/p/4150558.html

