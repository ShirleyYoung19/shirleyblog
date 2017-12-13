title: 代理和反射
author: Shirley Yang
date: 2017-12-11 22:20:54
tags:
---
Proxy构造函数接收两个参数，第一个是被代理的对象，第二个参数是处理程序。处理程序是包含一个或者多个陷阱的对象。

如果处理程序定义成一个空的对象的话，那么就是被代理的对象和代理之间的属性是相通的。

    var target = {};
    var proxy = new Proxy(target, {});
    
    proxy.name = "proxy";
    console.log(target.name); // "proxy"
    target.name = "target";
    console.log(proxy.name); // "target"
    

代理中的set陷阱
set陷阱接受四个参数： trapTarget, key, value, receiver.
返回值是true/fasle，根据是否成功设置该属性。

例如想保证target中新创建的属性的值都是数字类型：
    var target = {
        name: "target"
    };
    
    var proxy = new Proxy(target, {
        set(trapTarget, key, value, receiver) {
            if(!trapTarget.hasOwnProperty(key)){
                if(isNaN(value)) {
                    throw new TypeError('property must be a number');
                }
            }
            
            return Reflect.set(trapTarget, key, value, receiver);
        }
    })
    
    proxy.name = "proxy";
    console.log(target.name); // "proxy"
    proxy.age = "secret"; // TypeError
    proxy.age = 33;
    console.log(target.age) //33
    target.age = "secret";
    console.log(proxy.age) // "secret"
    // 要注意，处理程序中的陷阱是针对proxy起作用的，对于target对象没有影响
    
    
#### 代理中的get
    在JS中，访问一个对象的属性，即使这个属性不存在，也不会报错，只是会返回undefined。如果想避免这种情况，想让属性值不存在的时候直接报错，可以用过代理来实现。
    
    get的陷阱函数接受trapTarget, key两个参数。
    Reflect.get的返回值是该属性的的值。
    
    var target = {
        name: 'target',
    }
    
    var proxy = new Proxy(target, {
        get(trapTarget, key, receiver) {
            if( !(key in receiver) ) {
                throw new TypeError(key + ' is not exit');
            }
            return Reflect.get(trapTarget, key, receiver);
        }
    })
    
注意： 用in操作符判断某个属性是否存在的时候，是用receiver来访问的，而不是用trapTarget来访问，这是为了避免有has陷阱。（比如，通过has陷阱将name屏蔽掉了，那么应该proxy调用name的时候是报错的，但是如果对trapTarget使用in操作符，就能够访问到name，这是不对的）


#### 代理中的has陷阱
用has陷阱可以屏蔽到一些属性.
has接受两个参数（trapTarget, key）
Reflect.has返回值是true或者false

    var target = { name: 'Shirley', career: 'FE'};
    var proxy = new Proxy(target, {
        has(trapTarget, key) {
            if(key === 'name') {
                return false;
            }
            return Reflect.has(trapTarget, key)
        }
    })
    
    'name' in proxy // false;
    'career' in proxy // true;
    
#### 代理中的deleteProperty陷阱
delete操作符可以将对象的属性删除，如果操作成功，返回true，否则会返回false。
但是注意的是, delete删除的属性如果是nonconfigurable的属性，那么在严格模式下会报错，在非严格模式下回返回false。

    var object = {
        name: 'Shirley',
        value: 32,
    }
    Object.defineProperty(object, 'name', { configurable: false });
    
    delete object.value; // true;
    delete object.name; // false;

当对代理使用delete操作符时，会触发deleteProperty陷阱。deleteProperty应该传入的参数分别是trapTarget，key。例如：

    var object = {
        name: 'Shirley',
        value: 32,
    };
    
    Object.defineProperty(object, 'name', { configurable: false });
    
    var proxy = new Proxy(object, {
        deleteProperty (trapTarget, key) {
            if (key === 'name') {
                return false;
            }
            return Reflect.deleteProperty(trapTarget, key);
        }
    })


#### 代理中原型陷阱
ES6中，对原型的操作有Object.setPrototypeOf和Object.getPrototypeOf两种不同的方法。对应的原型陷阱也有setProtoypeOf和getProtoype两种。
其中，setPrototypeOf的参数是（trapTarget, proto）。而getPrototypeOf的参数则是（trapTarget）。

###### 原型代理陷阱的运行机制
getPrototypeOf的返回值只能是对象或者null，如果返回的基本类型的值就会报错。
setPrototypeOf的返回值只要不是false就代表设置成功。
因此，可以通过getPrototypeOf和setPrototypeOf将原型隐藏起来。

    var target = {
        name: 'shirley',
    };
    
    var proxy = new Proxy(target, {
        getPrototypeOf(trapTarget) {
            return null;
        },
        setPrototypeOf(trapTarget, proto) {
            return false;
        },
    });
    
    Object.getPrototypeOf(proxy); // null
    Object.setPrototypeOf(proxy, {}); //会报错
    
如果想使用这两个陷阱的默认行为，可以使用Reflect.setPrototypeOf和Reflect.getPrototypeOf这两种方法。

    var proxy = new Proxy(target, {
        getPrototypeOf(trapTarget) {
            return Reflect.getPrototypeOf(trapTarget);
        },
        setPrototypeOf(trapTarget, proto) {
            return Reflect.setPrototypeOf(trapTarget, proto);
        },
    });
###### 为什么有两组方法
Object.setPrototypeOf、 Object.getPrototypeOf 这一组方法和 Reflect.setPrototypeOf、Reflect.getPrototypeOf 这一组方法还是有一定差别的。

例如，如果Object.getPrototypeOf中传入了一个不是对象的值，会首先将这个值转换成一个对象，然后进行后续的操作。但是在Reflect中则会直接报错。

    var numberProto = Object.getPrototypeOf(1);
    numberProto === Number.prototype // true;
    
    Reflect.getPrototypeOf(1); //报错

Object.setPrototypeOf()方法失败的话，会返回错误，如果成功的话会返回第一个参数。Reflect.setPrototypeOf()方法成功会返回true, 失败会返回false。

    var target1 = {};
    var result1 = Object.setPrototypeOf(target1, {});
    console.log(result1 === target1); // true
    
    var target2 = {};
    var result2 = Reflect.setPrototypeOf(target2, {});
    console.log(result2); // true
    
在代理上使用Object/Reflect两个方法都会触发setPrototypeOf和getPrototypeOf两个陷阱





















    