---
title: js高级
date: 2020-06-15 23:38:21
tags:
 - js高级
categories: 
 - js高级
---

## 面向对象

三要素：

​	封装：数据权限和保密（public完全开发、protected对子类开发、private对自己开放。es6尚不支持，但是ts支持），减少耦合，不该外露的数据不外露，利于数据、接口的权限管理。

​	继承：子类继承父类

​	多态：同一接口不同实现，js应用极少，不要深究



## 单例设计模式

英文叫singleton pattern	

**表现形式：**就是一个对象

```js
var obj = {
	xxx:xxx
	xx:xx
}
```

在单例设计模式中，obj不仅仅是对象名，又被称为命名空间【NameSpace】，把描述事务的属性存放到命名空间中，多个命名空间是独立分开的，互不冲突



**作用：**

把描述同一件事物的属性和特征进行分组、归类，即存储在同一个堆内存中，避免了全局变量之间的冲突和污染。



**单例设计模式明明的由来：**

每一个命名空间都是js中Object这个内置基类的实例，而实例之间是互相独立互不干扰，所以我们称它为单例。可以把它理解为“单独的实例”



**高级单例模式：**

在给命名空间赋值的时候，不是直接赋值一个对象，而是先执行匿名函数，形成一个私有作用域（不销毁的栈内存），在这个私有作用域创建一个堆内存，把堆内存地址赋值给命名空间

这种模式的好处是我们可以在该私有作用域中创建很多内容，哪些需要供外面调取使用的，我们才暴露到返回的对象中，这也是实现模块化的一种思想

```js
var nameSpace = (function(){
	var n = 12
    function fn (){
		//..  
        console.log(n)
    }
    return {
        fn:fn
    }
})()

//改造
(function(window){
	var n = 12
    function fn (){
		//..  
        console.log(n)
    }
    window.nameSpace = {
        fn:fn
    }
})(window)
```

来一道题：

```js
var n = 2 
var obj = {
    n:3,
    fn: (function(n){
        n *= 2
        this.n += 2
        var n = 5
        return function(m){
        this.n *= 2
        console.log(m + (++n )) 
    }
    })(n)
}
var fn = obj.fn
fn(3) //9
obj.fn(3) //10
console.log(n,obj.n) // 8 6
```



## 构造函数模式

和单例模式差不多，把对象用构造函数改写

```js
function Fun(){
    this.xx = xx
    this.xxx = xxx
}
var fn1 = new Fun()
var fn2 = new Fun()
//fn1和fn2都是Fun的实例，但是是独立分开互不影响的
```

构造函数执行和普通函数执行一样，new Fun()也是形成一个私有作用域（栈内存）。之后也是形参赋值、变量提升，私有变量。

构造函数独有：在js代码自上而下执行之前，首先在当前形成的私有作用域中创建一个对象（创建一个堆内存，暂时不存储任何东西），并且让函数中的执行主体（this）指向这个新的堆内存（this指向实例）

代码自上而下执行

构造函数独有：代码执行完成，把之前创建的堆内存地址返回（浏览器默认返回）



**构造函数执行，不写return，浏览器会默认返回之前创建的实例，如果自己写了return呢?**

​	如果return的是一个基本数据类型，返回的结果依然是实例，不会受到影响。

​	如果return的是一个引用数据类型，则会把默认返回的实例覆盖，此时接收到的数据就不再是当前类的实例了。

所以构造函数执行时，尽量减少return防止覆盖实例

```js
function Fn(){
    this.name = 'AA'
    //覆盖实例
    return {
        name: 哈哈
    }
}
```



## 工厂模式

将new操作单独封装，遇到new时，就要考虑是否该使用工厂模式

示例：

我去麦当劳点餐，并不用自己亲手做，而是麦当劳直接将做好的餐给到我



简化后的uml类图：

{%asset_img p1.png%}

```js
class Product {
    constructor(name){
        this.name = name
    }
    init(){
        console.log('init')
    }
    fn1(){
        console.log('fn1')
    }
    fn2(){
        console.log('fn2')
    }
}
class Factory {
    create(name){
        return new Product(name)
    }
}

const factory = new Factory()
const p = factory.create('汉堡')
p.init()
```



**JQuery中的使用**

```js
class JQuery {
    constructor(selector){
        const dom = [...document.querySelectorAll(selector)]
        const length = dom ? dom.length : 0
    }
    append(node){
        //...
    }
    addClass(node){
        //...
    }
    html(data){
        //...
    }
    //省略多个api
}
window.$ = (selector) => {
    return new JQuery(selector)
}
$('p').append('<input value="666"/>')
```



## 观察者模式

发布、订阅；一对多



示例：

我去一点点买奶茶，买了之后排号，叫到我的号了我再过去取

简化后的uml类图：

{%asset_img p2.png%}

```js
//主题，保存状态，状态变化之后触发所有观察者对象
class Subject {
    constructor(){
    	this.state = 0
        this.observers = []
	}
    getState(){
        return this.state
    }
    setState(newState){
        this.state = newState
        this.notifyAllObservers()
    }
    notifyAllObservers(){
        this.observers.forEach(observer=>{
            observer.update()
        })
    }
    attach(observer){
        this.observers.push(observer)
    }
}

//观察者
class Observer {
    constructor(name,subject){
        this.name = name
        this.subject = subject
        this.subject.attach(this)
    }
    update(){
        console.log(`${this.name} update, state: ${this.subject.getState()}`)
    }
}

//test
const sub = new Subject()
const o1 = new Observer('o1',sub)
const o2 = new Observer('o2',sub)
const o3 = new Observer('o3',sub)
sub.setState(1)

//o1 update, state: 1S
//o2 update, state: 1
//o3 update, state: 1
```

