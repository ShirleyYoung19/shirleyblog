title: Iterator迭代器 和 Generator生成器
tags:
  - ES6
categories: []
author: Shirley
date: 2017-10-22 10:17:00
---

### 什么是迭代器

迭代器是一种特殊的对象。所有迭代器都有一个特殊的方法next().next方法会返回一个对象。这个结果对象有两个属性value和done。其中value表示的是next内部的指针现在对应的value值，而done则用来表示是否还有未返回的值。

如果在最后一个值返回之后再次调用next方法，首先done是返回fasle的，value其实是返回迭代器的值，而不是数据集合的某一个值，如果没有定义的话，value返回undefined。

例如ES5的语法可以模拟一个简单的迭代器：

```bash
function createIterator(items) {
  var i = 0;
  return {
    next: function () {
      var done = (i >= items.length);
      var value = done ? undefined : items[i++];
      
      return { value: value, done: done};
    }
  }
}

var iterator = createIterator([1, 2, 3]);
iterator.next(); // {value: 1, done: fasle}
iterator.next(); // {value: 2, done: fasle}
iterator.next(): // {value: 3, done: fasle}
iterator.next(): // {value: undefined, done: true}
```
上述代码是简单的生成了一个iterator,在ES6中引入了一个生成器对象，它可以简化创建迭代器的过程


## 什么是生成器

生成器是返回值是迭代器的函数。通过在function后面添加*号来表示与其他普通函数的区别。同时生成器内部还引入了一个新的关键字 yield。yield可以返回任何值或者表达式。我们可以如下改造上的createIterator函数

``` bash
function *createIterator(items) {
  for (var i = 0; i < items.length; i ++) {
    yield items[i];
  }
}
```
生成器有趣的地方就在于，每次执行完一条yield语句之后就会停止执行，然后等待下一次迭代器调用next方法的时候才会继续执行。
同时，注意yield这个关键字只能在生成器函数内调用，而且不能穿过函数的边界（在生成器内部的函数内部调用也不可以）。

#### 生成器函数表达式
生成器也可以通过函数表达式来定义。
``` bash
var *createIterator = function (items) {
  for (var i = 0; i < items.length; i ++) {
    yield items[i];
  }
}
```
#### 生成器对象方法
生成器函数同样可以做为对象方法，例如：
``` bash
var object = {
  *createIterator(items) {
     for (var i = 0; i < items.length; i ++) {
    yield items[i];
  }
 }
}
```












