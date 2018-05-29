---
title: react笔记(一)——render()
date: 2018-05-29 15:02:12
tags: react
---
### 表达式插入

在 JSX 当中你可以插入 JavaScript 的表达式，表达式返回的结果会相应地渲染到页面上。表达式用 {} 包裹。
```
...
render () {
  const word = 'is good'
  return (
    <div>
      <h1>React 小书 {word}</h1>
    </div>
  )
}
...
```
<!--more-->

{} 内可以放任何 JavaScript 的代码，包括变量、表达式计算、函数执行等等。 render 会把这些代码返回的内容如实地渲染到页面上，非常的灵活。

表达式插入不仅仅可以用在标签内部，也可以用在标签的属性上，例如：
```
...
render () {
  const className = 'header'
  return (
    <div className={className}>
      <h1>React 小书</h1>
    </div>
  )
}
...
```

注意，直接使用 class 在 React.js 的元素上添加类名如 &#60;div class="xxx"&#62; 这种方式是不合法的。因为 class 是 JavaScript 的关键字，所以 React.js 中定义了一种新的方式：className 来帮助我们给元素添加类名。

还有一个特例就是 for 属性，例如 &#60;label for='male'&#62;Male&#60;/label&#62;，因为 for 也是 JavaScript 的关键字，所以在 JSX 用 htmlFor 替代，即 &#60;label htmlFor='male'&#62;Male&#60;/label&#62;。而其他的 HTML 属性例如 style 、data-* 等就可以像普通的 HTML 属性那样直接添加上去。

### 条件返回

{} 上面说了，JSX 可以放置任何表达式内容。所以也可以放 JSX，实际上，我们可以在 render 函数内部根据不同条件返回不同的 JSX。例如:

```
...
render () {
  const isGoodWord = true
  return (
    <div>
      <h1>
        React 小书
        {isGoodWord
          ? <strong> is good</strong>
          : <span> is not good</span>
        }
      </h1>
    </div>
  )
}
...
```
如果你在表达式插入里面返回 null ，那么 React.js 会什么都不显示，相当于忽略了该表达式插入.
### JSX 元素变量
如果你能理解 JSX 元素就是 JavaScript 对象。那么你就可以联想到，JSX 元素其实可以像 JavaScript 对象那样自由地赋值给变量，或者作为函数参数传递、或者作为函数的返回值。
```
...
render () {
  const isGoodWord = true
  const goodWord = <strong> is good</strong>
  const badWord = <span> is not good</span>
  return (
    <div>
      <h1>
        React 小书
        {isGoodWord ? goodWord : badWord}
      </h1>
    </div>
  )
}
...
```
这里给把两个 JSX 元素赋值给了 goodWord 和 badWord 两个变量，然后把它们作为表达式插入的条件返回值。达到效果和上面的例子一样，随机返回不同的页面效果呈现。