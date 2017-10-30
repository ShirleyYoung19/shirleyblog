title: Iterator迭代器 和 Generator生成器
tags:
  - ES6
categories: []
author: Shirley
date: 2017-10-22 10:17:00
---
## 什么是迭代器

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

#### For-of循环
ES6中数组、Map、Set和字符串都有内置的迭代器。因此可以采用for-of循环。
for-of循环相当于每一次循环都会调用一次迭代器的next方法。当迭代器的done值返回为true的时候，不用返回undefined而是直接结束。

    var array = [1, 2, 3];
    for (var x of array) {
        console.log(x);
    }

#### Symbol.iterator
ES6中的集合的迭代器可以通过Symbol.iterator访问到。例如：

    var iterator = array[Symbol.iterator]()
    iterator.next() // {value: 1, done: false}
    iterator.next() // {value: 2, done: false}
    iterator.next() // {value: 3, done: false}
    iterator.next() // {value: undefined, done: true}

同样，可以一通过Symbol.iterator来将一个非迭代对象转换成迭代对象

    var collections = {
      items: [],
      *[Symbol.iterator]() {
        for (var x of this.items) {
          yield x;
        }
      }
    }
    collections.items.push(1);
    collections.items.push(3);
    collections.items.push(32);
    for (var x of collections) {
      console.log(x);
    }
    // 1
    // 3
    // 32
    
    ##可迭代对象和for-of循环
可迭代对象具有Symbol.iterator属相
ES6中数组、Map、Set和字符串都有内置的迭代器。因此可以采用for-of循环。
for-of循环相当于每一次循环都会调用一次迭代器的next方法。当迭代器的done值返回为true的时候，不用返回undefined而是直接结束。

    var array = [1, 2, 3];
    for (var x of array) {
        console.log(x);
    }

#### 访问默认迭代器Symbol.iterator
ES6中的集合的迭代器可以通过Symbol.iterator访问到。例如：

    var iterator = array[Symbol.iterator]()
    iterator.next() // {value: 1, done: false}
    iterator.next() // {value: 2, done: false}
    iterator.next() // {value: 3, done: false}
    iterator.next() // {value: undefined, done: true}

#### 创建可迭代对象
同样，可以一通过Symbol.iterator来将一个非迭代对象转换成迭代对象

    var collections = {
      items: [],
      *[Symbol.iterator]() {
        for (var x of this.items) {
          yield x;
        }
      }
    }
    collections.items.push(1);
    collections.items.push(3);
    collections.items.push(32);
    for (var x of collections) {
      console.log(x);
    }
    // 1
    // 3
    // 32
    
## 内置迭代器
在ES6中很多内建类型都有自己的内建迭代器，只有内建迭代器不能满足要求的时候才需要再去创建自己的迭代器

#### 集合对象的迭代器和默认迭代器
在ES6中，数组、Set和Map都有自己的内置迭代器，分别是entries，values和keys.
entries(): 返回一个迭代器，其值是键值对
keys(): 返回一个迭代器，值是键值对的键值
values(): 返回一个迭代器，值是键值对的值
    
对象| entries | keys | values | defualt
---|---|---|---
数组| [index, Array[index]]:返回数组序号和数组这一项的值| index: 返回数组每一项的序号值 | Array[index]返回每一项的数组的值 | 返回值与调用values一样
Set集合|[value, value]返回Set中每一项重复两遍组成的数组|返回Set集合中每一项返回的值|返回Set集合中每一项返回的值|与values的返回值一样
Map集合|返回每一项的[key, value]|返回每一项的key|返回每一项的value|与entries返回值一致


    var array = ['a', 'b', 'c'];
    var set = new Set([1, 2, 3]);
    var map = new Map([['name', 'Shirley'], ['age', 'secret']]);
    
    for (var item of array.entries()) {
      console.log(item)
    }
    // [0, 'a']
    // [1, 'b']
    // [2, 'b']
    
     for (var item of array.keys()) {
      console.log(item)
    }
    // 0
    // 1
    // 2
    
    for (var item of array) {
      console.log(item)
    }
    // a
    // b
    // c
    
    for (var item of set.entries()) {
      console.log(item)
    }
    // [1, 1]
    // [2, 2]
    // [3, 3]
    
    for (var item of set.keys()) {
      console.log(item)
    }
    // 1
    // 2
    // 3
    
    for (var item of set.values()) {
      console.log(item)
    }
    // 1
    // 2
    // 3
    
    for (var item of set) {
      console.log(item)
    }
    // 1
    // 2
    // 3
    
    for (var item of map.entries()) {
      console.log(item)
    }
    // ["name", "Shirley"]
    // ["age", "secret"]
    
    for (var item of map) {
      console.log(item)
    }
    // ["name", "Shirley"]
    // ["age", "secret"]
    
    for (var item of map.keys()) {
      console.log(item)
    }
    // name
    // age
    
    for (var item of map.values()) {
      console.log(item)
    }
    // Shirley
    // secret
    
#### 字符串迭代器
返回字符串的每一个字符，如果是双字节字符则不会返回两个而是一个。

#### NodeList迭代器
DOM标准中的NodeList也有了默认迭代器
    var divs = document.getElementsByTagName("div");
    
    for (var div of divs) {
      console.log(div.id);
    }

## 展开运算符和非数组可迭代对象
数组可以使用展开运算符将值展开，同样的非数组的可迭代对象也可以使用展开运算符。

    var set = new Set([1, 2, 3]);
    var map = new Map([['name', 'Shirley'], ['age', 'secret']]);
    var arr1 = [...set];
    // arr1 = [1, 2, 3];
    var arr2 = [...map];
    // arr2 = [['name', 'Shirley'], ['age', 'secret']]
NodeList可以使用展开运算符

##高级迭代器功能
#### 给迭代器传递参数

    var createIterator = function *()  {
      var result1 = yield 1;
      var result2 = yield result1 + 2;
      var result3 = yield result2 + 4;
    }
    var iterator = createIterator();
    iterator.next(); // {value: 1, done: false}
    iterator.next(4); // {value: 6, done: false}
    iterator.next(4); // {value: 8, done: false}

当第一次调用next方法的时候，是执行到蓝色字体所在的位置就暂停了。注意这个时候即使传入参数也不会有什么作用。
var result1 = <font color=#7FFFD4>yield 1</font>;

第二次调用next方法的时候，也是接着蓝色的地方开始执行的，yield返回的值，即调用next的时候传入的值赋给result1。然后执行完红色的部分的yield就停止。
如果这个时候调用next的时候不传入任何值，那么就会返回undefined。然后undefined + 2 => NaN。所以返回值为{value: NaN, done: false}。
<font color=#F00>var result1 = </font> <font color=#7FFFD4>yield 1</font>;
var result2 = <font color=#F00> yield result1 + 2 </font>;

#### 在迭代器中抛出错误
迭代器可以调用throw方法，当其恢复执行的时候可以主动抛出错误。
    var createIterator = function *()  {
      var result1 = yield 1;
      var result2;
      try {
        result2 = yield result1 + 3;
      } catch(error) {
        result2 = 5;
      }
    
      yield result2 + 4;
    }
    var iterator = createIterator();
    iterator.next(); // {value: 1, done: false}
    iterator.throw(new Error('Oops...')); // 会抛出错误停止执行
    iterator.next(); // {value: undefined, done: true}
    
当iterator.throw()之后，迭代器抛再次执行时，即执行var result1赋值的时候就会报错，停止代码的执行。

    var iterator = createIterator();
    iterator.next(); // {value: 1, done: false}
    iterator.next(0); // {value: 3, done: false}
    iterator.throw(new Error('Oops...')); // {value: 9, done: false}
    iterator.next(); // {value: undefined, done: true}

此时当抛出错误的时候，迭代器会继续执行result2的赋值，因为捕获到了错误因此，会执行result2 = 5的赋值，从而可以执行后面的代码。
因此如果对错误的捕获处理正确的话，throw()和next()一样也会返回一条语句。

#### 生成器返回语句
生成器中可以像函数一样调用return语句返回一个返回值。而且也是会中断后面的代码。在for-of的循环中，迭代器会忽略return返回的值。

    function *createIterator() {
     yield 1;
     return 5;
     yield 4;
    }
    var iterator = createIterator();
    iterator.next(); //{value: 1, done: false}
    iterator.next(); //{value: 5, done: true}
    iterator.next(); //{value: undefined, done: true}
    
但是要注意，return只会返回一次。后面再次调用next方法，就只会返回undefined的了。

#### 委托生成器
某些情况下，需要将两个生成器组合才能形成一个新的生成器。这个时候需要再yield语句和生成器函数名之间调用*将生成数据的过程委托给其它的生成器。

    function *createNumberIterator() {
      yield 1;
      yield 2;
      return 3;
    }
    function *createRepeatingIterator(count) {
      for(let i = 0; i < count; i++) {
        yield "repeat"; 
      } 
    }
    function *createCombinedIterator (){
      var result = yield *createNumberIterator();
      yield *createRepeatingIterator(result);
    }
    var iterator = createCombinedIterator();
    iterator.next(); //{value: 1, done: false}
    iterator.next(); //{value: 2, done: false}
    iterator.next(); //{value: 'repeat', done: false}
    iterator.next(); //{value: 'repeat', done: false}
    iterator.next(); //{value: 'repeat', done: false}
    iterator.next(); //{value: undefined, done: true}
    
要注意的是，return的值并不会被返回。除非在代码中yield出来。

## 异步任务执行

    // taskDef是一个生成器
    function run(taskDef) {
      // 首先创建一个迭代器
      var task = taskDef();
      // 先调用一次迭代器
      var result = task.next();
    
      function step() {
        // 首先判定一下迭代器是否已经调用结束了
        if(!result.done) {
          // 判断result的value值是否是一个函数
          if(typeof result.value === 'function') {
            //result.value应该是一个异步执行的函数，向这个函数中传入一个回调函数
            result.value(function(err, data)){
              if(err) {
                // 如果发生了错误，那么就用迭代器抛出错误并且停止函数的执行
                task.throw(err);
                return;
              }
              // 如果没有错误，则进行再一次调用迭代器
              result = task.next(data);
              step();
            }
          } else {
            //如果result的value不是函数，那么就当做下一次的参数调用next方法，并将返回的结果赋值给result
            result = task.next(result.value);
            step();
          }
        }
      }
    
      // 调用step方法
      step();
      
    }