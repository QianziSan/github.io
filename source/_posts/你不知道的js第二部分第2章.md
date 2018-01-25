---
title: 你不知道的js第二部分第2章--this全面解析
date: 2018-01-23 15:35:10
tags:
---

上一章我们明白了this是在调用的时候被绑定的。

 开篇知识点：
 1. 四条绑定规则优先级：new > 显式绑定（call，apply,bind） > 隐式绑定（上下文）>默认绑定（全局or undefined）；
 2. 有些调用会在无意中使用默认绑定规则，可以使用一个MDZ对象。
 3. ES6中的箭头函数不适用四条规则，而是继承外层函数调用的this绑定，类似self = this。

## 调用位置
1. 调用栈：可以想象成一个函数调用链。由当前执行位置往上数。<br>
2. 调用位置：在调用栈中找到当前执行位置，调用位置就在执行的上一级。<br>
（浏览器开发者工具中打断点能更快的找到）

## 绑定规则

### - 默认绑定
最常用的函数调用类型：独立调用。可以把这条规则看作是无法应用其他规则时的默认规则。非严格模式下，指向全局。
```
function(){
    console.log(this.a);
}
var a = 2;

foo();// 2
```
通过分析调用位置看foo()如何调用的。在代码中，foo()是直接使用不带任何修饰的函数进行调用的。因此只能使用默认规则。this.a被解析为全局变量a。<br>
### - 隐式绑定
另一个规则是考虑调用位置是否有上下文，或者说是被某个对象拥有或包含。
```
function(){
    console.log(this.a);
}
var obj={
    a:2,
    foo:foo
};
obj.foo();//2
```
这段代码中，foo()在全局作用于中声明。然后被当作引用属性添加到obj中。无论是在obj中定义<sup>[1]</sup>还是在先定义再添加为引用。foo()都不属于obj对象。<br>

然而，调用位置会使用obj上下文来引用函数。当foo()被调用时，他的前面确实加上了对obj的引用。可以说函数被调用时，obj对象“拥有”或者“包含”它。隐式绑定规则会把函数调用中的this绑定到这个上下文对象。因此this.a 和 obj.a是一样的。<br>

对象属性引用链中只有上一层或者说最后一层在调用位置中起作用。举例：
```
function(){
    console.log(this.a);
}
var obj2={
    a:42,
    foo:foo
};
var obj1 = {
    a:2,
    obj2:obj2
};
obj1.obj2.foo();//42
```
**隐式丢失**：被隐式绑定的函数有一个常见的问题，就是会丢失绑定对象，应用到默认绑定，从而被当的那个到全局或者undefined（严格模式）。 <br>


* 函数别名
```
function(){
    console.log(this.a);
}
var obj={
    a:2,
    foo:foo
};

var bar = obj.foo;//函数别名！

var a = 'oops,global';

bar();//'oops,global'
```
虽然bar是obj.foo的一个引用，但实际上，它引用的是foo函数本身，因此此时的bar()其实是一个不带任何修饰的函数调用，因此应用了默认绑定。

* 传入回调函数
```
function(){
    console.log(this.a);
}

function doFoo(fn){
    //fn其实引用的foo
    
    fn();// 调用位置
}

var obj={
    a:2,
    foo:foo
};

var a = 'oops,global';

doFoo( obj.foo ); //'oops,global'
```
参数传递其实就是一种隐式赋值，因此我们传入函数时也会被隐式赋值，多以结果和上一个例子一样。<br>

如果函数传入语言内部的函数而不是传入自己声明的函数，结果会是一样的。
```
function(){
    console.log(this.a);
}

var obj={
    a:2,
    foo:foo
};

var a = 'oops,global';

setTimeout( obj.foo ,100);//'oops,global'
```
javascript环境内置的setTimeout()函数实现和下面的伪代码类似：
```
function setTimeout( fn,delay){
    fn();
}
```
正如上边一样，回调函数丢失this绑定是非常常见的。除此之外，还有一种情况this的行为会出乎我们的意料。调用回调函数的函数可能会修改this的指向。在一些流行的javascript库中事件处理器常会把回调函数的this强制绑定到触发事件的dom上。在一些情况下会很有用，但又是会让人很郁闷。

### - 显式绑定
就行刚才那样，在分析隐身绑定时，我们必须在一个对象的内部包含一个指向函数的属性，并通过这个属性间接的引用函数，从而把this间接（隐式）绑定到这个对象上。

如果不想在对象内部包含函数引用。可以用javascript函数自带的 **call(..)** 和 **apply(..)** 方法。

它们的第一个参数是一个对象，是给this准备的，接着在调用函数时将其绑定到this，因为可以直接指定this的绑定对象，所以我们称之为 **显式绑定**。

```
function(){
    console.log(this.a)
}

var obj={
    a:2
};

//通过call我们在调用foo时强制把它的this绑定到obj上。
foo.call(obj);// 2
```
可惜，显式绑定仍然无法解决我们之前提出的绑定丢失问题。

* 硬绑定

显式绑定的一个变种，可以解决。
```
function(){
    console.log(this.a)
}

var obj={
    a:2
};

var bar = function(){
    foo.call(obj);
}

bar();//2
setTimeout(bar,100);//2

//硬绑定的bar不能再修改它的this
bar.call(window);//2
```
我们常见了函数bar();并在它的内部手动调用了foo.call(obj),因此强制把foo的this绑定到了obj上。之后无论如何调用bar,它总会手动在obj上调用foo。这种绑定是一种显式的强制绑定，因此称之为硬绑定。

硬绑定的典型应用就是创建一个包裹函数，负责接收参数，并返回值：
```
function foo(somethis){
    console.log(this.a,something);
    return this.a + something;
}

var obj={
    a:2
};

var bar = function(){
    return foo.apply(obj,arguments);
}

var b = bar( 3 ); //2 3
console.log( b ); //5
```

另一种使用方法是创建一个可重复使用的辅助函数：

```
function foo(somethis){
    console.log(this.a,something);
    return this.a + something;
}

function bind(fn,obj){
    return function(){
        return fn.apply(obj,arguments);
    };
}
var obj={
    a:2
};

var bar = bind(foo,obj);

var b = bar( 3 ); //2 3
console.log( b ); //5
```
由于硬绑定是一种非常常用的模式，es5提供了内置的方法，Functoin.prototype.bind,用法如下：
```
function foo(somethis){
    console.log(this.a,something);
    return this.a + something;
}

var obj={
    a:2
};

var bar = foo.bind(obj); //内置bind方法！

var b = bar( 3 ); //2 3
console.log( b ); //5
```
bind(..)会返回一个硬编码的新函数，它会把你指定的参数设置为this的上下文并调用原始函数。

* API调用上下文

第三方库的许多函数，以及javascript语言和宿主环境中许多新的内置函数都体够了可选的参数，通常被称之为‘上下文’，其作用和bind(..)一样，确保你的回调函数中使用指定的this。例如：forEach()等。

### - new绑定
 在讲解new绑定前要澄清一个误解：javascript中new操作符和传统的面向类的语言不一样。javascript中的“构造函数”只是使用new操作符时被调用的函数。他们不属于某个类，也不会实例化一个类。他们只是被new操作符调用的普通函数，所以并不存在所谓的“构造函数”，只有对于函数的“构造调用”。

 使用new来调用函数，或者发生构造函数调用时，会自动执行下面的操作：

 1. 创建（或者说构造）一个全新的对象。
 2. 这个对象会被执行[[Prototype]]链接。
 3. 这个对象会绑定到函数调用的this.
 4. 如果这个函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。
```
function foo(a){
    this.a = a;
}

var bar = new foo(2);
console.log(bar);//2
```
使用new来调用foo(...)时，我们会常造一个新的对象并把它绑定到foo(...)调用中的this上。

## 优先级
我们了解了函数调用中this的四条绑定规则，需要做的就是找到函数调用位置，然后判断应用那条规则。但是如某个调用位置可以应用多条规则该怎么办？为了解决这个问题，必须给这些规则设定优先级。

1. **默认绑定的优先级最低。**
2. 隐式绑定与显式绑定的比较：
```
function foo(){
    console.log(this.a);
}

var obj1 = {
    a:2,
    foo:foo
};

var obj2 = {
    a:3,
    foo:foo
};

obj1.foo();//2
obj2.foo();//3

obj1.foo.call(obj2);//3
obj2.foo.call(obj1);//2
```
**可以看到显式绑定优先级更高，也就是说在判断式先考虑是否可以存在显式绑定。**

3. new绑定和隐式绑定：
```
function foo(something){
    this.a = something;
}
var obj1 = {
    foo:foo
};

obj1.foo(2);
console.log(obj1.a);//2

var bar = new obj1.foo(4);
console.log(obj1.a); //2
console.log(bar.a);  //4
```
**可以看到4被绑定到了bar上而不是obj1，new 绑定比隐式绑定优先级高。**

4. new绑定与显式绑定
new 和 call/apply 无法一起使用，因此无法通过new foo.call(obj)来直接测试。但是可以用硬绑定来测它俩的优先级。
```
function foo(something){
    this.a = something;
}
var obj1 = {};

var bar = foo.bind(obj1);
bar(2);
console.log(obj1.a);//2

var baz = new bar(3);
console.log(obj1.a);//2
console.logO(baz.a);//3
```
**new 修改了硬绑定（到obj1的）调用bar(..)中的this。new绑定比显式绑定优先级高。**
(javascript中会判断硬绑定函数是否被new调用，如果是的话就会使用新创建的this替代硬绑定的this)

**判断this：**
1. 函数是否在new中调用？如果是的话*this绑定的是新创建的对象*。var bar = new foo()
2. 函数是否通过call、apply或者bind绑定调用？如果是的话*this绑定的是之指定的对象*。var bar = foo.call(obj2);
3. 函数是否在某个上下文对象中调用？如果是的话*this绑定的是那个上下文对象*。var bar = obj1.foo();
4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则就绑定到全局对象。

**绑定例外：**
有些调用可能在无意中使用默认绑定规则。如果想“更安全”地忽略this绑定，你可以使用一个DMZ对象。

**箭头函数中的this：**
ES6中的箭头函数并不会使用四条标准的绑定规则，而是根据当前的词法作用域来决定this，具体来说，箭头函数会继承外层函数调用的this绑定。这其实和ES6之前代码中的self = this 机制一样。


