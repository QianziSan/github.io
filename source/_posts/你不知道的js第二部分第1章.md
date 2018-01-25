---
title: 你不知道的js第二部分第1章--关于this
date: 2018-01-10 10:59:40
tags:
---
## 为什么使用this    

 显式传递上下文会让代码变得越来越混乱。this提供了一种更优雅的隐式“传递”一个对象的引用，可以将API设计的更加简洁并且易于复用。

## 对this的误解
1. this指向自身

    人们把this理解成指向函数自身,这个腿短从英语的语法角度来说是说的通的。
    但在javascript中this并不像我们所想的那样。
    思考一下下边的代码
    ```
    function foo (num){
        console.log( "foo:" + num);
        this.count++; // 记录foo被调用的次数
    }

    foo.count = 0;

    var i;
    
    for(i=0; i<10; i++){
        if(i>5){
            foo(i);
        }
    }
    // foo:6
    // foo:7
    // foo:8
    // foo:9

    //foo 被调用了多少次？
    console.log( foo.count );//0
    ```
    console.log语句产生了4条输出，证明foo(..)确实被调用了4次。但foo.count仍然是0。显然从字面意思理解this是错误的。<br>
   
    执行foo.count=0时，的确向函数对象foo 添加了一个属性count。但是函数内部代码this.count中的this并不是指向那个函数对象，所以虽然属性名相同，跟对象却不相同。<br>

    如果不深究为什么this的行为和预期不一样。可以用*词法作用域的*的方法改写
    ```
    function foo (num){
        foo.count=0;
    }
    ```
    注意：setTimeout(function(){},10)之类的匿名函数，无法指向自身。建议在需要引用时使用具名函数（表达式）。arguments.callee已经被弃用，不建议使用。

    除了使用foo标识符代替this来引用函数，另一种方法是强制this指向foo函数对象：
    ```
    function foo (num){
        console.log( "foo:" + num);
        this.count++; // 记录foo被调用的次数
    }

    foo.count = 0;

    var i;
    
    for(i=0; i<10; i++){
        if(i>5){
            //使用call(...)可以确保this指向函数对象foo本身
            foo.call(foo,i);
        }
    }
    // foo:6
    // foo:7
    // foo:8
    // foo:9

    //foo 被调用了多少次？
    console.log( foo.count );//4
    ```
2. 它的作用域

    第二种常见的误解是，this指向它的函数作用域。这个问题有点复杂，因为在某种情况下是正确的，其他情况下是错误的。<br>

    需要明确的是，this在任何情况下都不指向函数的词法作用域。在javascript内部，作用域确实和对象类似，课件的标识符都是他的属性。但是作用域“对象”无法通过javascript代码访问，他存在于javascript**引擎**内部。

    思考一下下面的代码，它试图跨越边界，使用this来隐式的引用函数词法作用域：
    ```
    function foo(){
        var a = 2;
        this.bar();
    }

    function bar(){
        console.log(this.a)
    }
    foo()// ReferenceError:a is not defined
    ```
    这段代码非常完美的膳食了this多么容易误导人。

    首先，这段代码视图通过this.bar()来引用bar()函数。这样调用能成功纯属意外，之后解释原因。调用bar()最自然的方法是省略前面的this，直接用词法引用标识符。

    此外，这段代码还试图用this联通foo()和bar()的词法作用域，从而让bar()可以访问foo()作用域的变量a。这是不可能实现的。

    每当你想要把this和词法作用域的查找混合使用时，一定要提醒自己，这无法无法实现。

    ## this到底是什么

    排除一些错误理解之后，看看this到底是一种什么机制。

    this是运行时进行绑定的，并不是编写时绑定，他的上写文取决于函数调用时的各种条件。this的绑定和函数声明的位置没有任何关系，只取决于函数的调用。

    当一个函数被调用时，会创建一个活动记录（有时也被称为执行上下文）。这个记录会包含函数在哪里调用（调用栈）、函数的调用方式、传入的参数参数等信息。this就是记录的一个属性，会在执行的过程中用到。

    ## 小结
    ```
    1.学习this的第一步是明白this既不指向函数自身，也不指向函数的词法作用域。
    2.this实际是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用。
    ```
    

