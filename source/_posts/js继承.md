---
title: js继承
date: 2020-04-28 22:54:27
tags: 
 - 基础
categories: 
 - 基础
---

## 原型链继承

记住一句话：父类的实例是子类的原型对象

```js
function Parent (name) {
  this.name = name
  this.say = function(msg){
    alert(msg)
  }
}
var p = new Parent('爹爹')

function Son (name){
  this.name = name
}

Son.prototype = p //父类的实例是子类的原型对象

var s = new Son('儿儿')
s.say('爸爸爸爸我是你的好儿子啊')
```

缺点：

1.无法向父类构造函数传参（例子中无法向Parent实例中的name属性传参）

2.原型链中引用类型的属性会被所有实例共享，即所有实例对象使用的是同一份数据，会互相影响（引用类型：例如数组的删减）



## 构造函数继承

 在子级构造函数中调用父级构造函数 

```js
function Parent (name){
    this.name = name
    this.age = 38
    this.say = function(msg){
        alert(msg)
    }
}
Parent.prototype.hobby = '吃屎'
var p = new Parent('爹爹')

function Son (){
    Parent.call(this,'儿儿')
}

var s = new Son()
```

缺点： 

1. 只继承了父类构造函数的属性，无法继承父类原型上的的属性  

2. 无法实现构造函数的复用。（每次用每次都要重新调用） 

3. 每个新实例都有父类构造函数的副本，臃肿 



## 组合继承

组合继承，即原型链继承+构造函数继承

```js
function Parent (name){
    this.name = name
    this.age = 38
    this.say = function(msg){
        alert(msg)
    }
}
Parent.prototype.hobby = '吃屎'
var p = new Parent('爹爹')  //p:{name: "爹爹", age: 38, say: ƒ}

function Son (){
    Parent.call(this,'儿儿')
}

var s = new Son()  //s:{name: "儿儿", age: 38, say: ƒ}

Son.prototype = p  //Son.prototype: {name: "爹爹", age: 38, say: ƒ}
```

推荐使用，但性能消耗较大，因为调用了两次父类。



### 优化、完美写法(解决调用两次父类问题、prototype.constructor指向问题)

```js
function Parent (name){
    this.name = name
    this.age = 38
    this.say = function(msg){
    	alert(msg)
    }
}
Parent.prototype.hobby = '吃屎'

function Son (){
	Parent.call(this,'儿儿')
}

//将两个对象区分开，并且直接调用s.hobby即可得到'吃屎'，而不需要调用Son.prototype.hobby
Son.prototype = Object.create(Parent.prototype)
//给Son原型对象写一个自己的constructor
Son.prototype.constructor = Son
var s = new Son() 

```



## 原型式继承

用一个函数包装一个对象，然后返回这个函数的调用

```js
function Parent (name){
    this.name = name
    this.age = 38
    this.say = function(msg){
    	alert(msg)
    }
}
Parent.prototype.hobby = '吃屎'
var p = new Parent('爹爹')

function Son (obj){
    function F(){}
    F.prototype = obj
    return new F()
}

var s = new Son(Parent)

Son.prototype = p
```

 类似于复制一个对象，用函数来包装 

 缺点：所有实例都会继承原型上的属性。浅拷贝



## es6 class extends

```js
class Parent {
    constructor(name){
        this.name = name
        this.age = 38
    }
    say(msg){
        alert(`我是${this.name},今年${this.age}岁`)
    }
}

var p = new Parent('爹爹')

class Son extends Parent {
    constructor(name,age){
        super()
        this.name = name
        this.age = age
    }
}
var s = new Son('儿儿',10)

p.say() //我是爹爹,今年38岁
s.say() //我是儿儿,今年10岁
```

