---
title: promise相关
date: 2020-05-29 14:36:21
categories: 
 - 面试
 - promise
tags: 
 - 面试
 - promise

---



## Promise是什么

Promise是一个构造函数，是ES6中进行异步编程的一种新的解决方案。



## Promise状态改变

pending -> resolved

pending -> rejected

只有这两种状态改变，且一个promise对象只能改变一次，无论成功还是失败，都会有一个结果数据。

成功的结果数据一般称为value，失败的结果数据一般称为reason



## 基本使用

```js
const p = new Promise((resolve,reject)=>{
    setTimeout(()=>{
        const time = Date.now()
        if(time%2===0){
           resolve('resolve,time='+time)
        }else{
           reject('reject,time='+time)
        }
    },1000)
})

p.then(
	(value)=>{
        console.log('success callback!'+value)
    },
    (reason)=>{
        console.log('fail callback!'+reason)
    }
)
/*.catch(
	(reason)=>{
        console.log('fail callback!'+reason)
    }
)*/
```



## Promise的Api

**1.Promise构造函数：Promise(excutor){}**

​	excutor：同步执行 (resolve,reject)=>{}

​	resolve：内部定义成功调用的函数 value=>{}

​	reject：内部定义失败调用的函数 reason=>{}

​	说明：excutor会在Promise内部立即执行同步回调，异步操作在执行器中执行



**2.Promise.prototype.then()：(onResolved,onRejected)=>{}**

​	onResolved：成功的回调 value=>{}

​	onRejected：失败的回调 reason=>{}

​	说明：指定用于得到成功value的成功回调和用于得到失败reason的失败回调，返回一个新的promise对象



**3.Promise.prototype.catch()：(onRejected)=>{}**

​	onRejected：失败的回调 reason=>{}

​	说明：then()的语法糖，相当于 then(undefined,onRejected)



**4.Promise.resolve()：(value)=>{}**

​	value：成功的数据或promise对象

​	说明：返回一个成功或失败的promise对象

```js
new Promise((resolve)=>{
    resolve('success')
})
//语法糖 两种等价
Promise.resolve('success')
```



**5.Promise.reject()：(reason)=>{}**

​	onRejected：失败的原因

​	说明：返回一个失败的promise对象

```js
new Promise((resolve,reject)=>{
    reject('fail')
})
//语法糖 两种等价
Promise.reject('fail')
```



**6.Promise.all()：(promises)=>{}**

​	promises：包含n个promise的数组

​	说明：返回一个新的promise，只有所有promise都成功才成功，有一个失败了就失败



**7.Promise.race()：(promises)=>{}**

​    promises：包含n个promise的数组

​	说明：返回一个新的promise，第一个完成的promise结果状态就是最终结果



```js
const p1 = new Promise(resolve=>{
    resolve(1)
})
const p2 = new Promise(resolve=>{
    resolve(2)
})
const p3 = new Promise((resolve,reject)=>{
    resolve(3)
    //reject('p3 fail')
})

Promise.all([p1,p2,p3]).then(value=>{
    console.log('all success')
},reason=>{
    console.log('someone failed')
})

Promise.race([p1,p2,p3]).then(value=>{
    console.log('first success')
},reason=>{
    console.log('first failed')
})
```



## 一些注意事项

```js
const p1 = new Promise((resolve,reject)=>{
    resolve('666')
    //reject('err')
})

p1.then(value=>{
    console.log('onResolved1 success' ,value)
    //return '666第二次'
    //不写返回值默认return undefined
},reason=>{
    console.log('onResolved1 fail,', reason)
     //throw 'err第二次' 或者 return Promise.reject('err第二次')
    //不写返回值默认return undefined
}).then(value=>{
    console.log('onResolved2 success' ,value)
},reason=>{
    console.log('onResolved2 fail,', reason)
})

//打印结果为
//onResolved1 success 666
//onResolved2 success undefined
```

为什么第二次.then()会得到undefined？

第一次.then()中的onResolved没有写返回值，默认返回一个undefined



**如果将resolve('666')改为reject('err')，会打印出什么？**

```js
//onResolved1 fail, err
//onResolved2 success undefined
```

为什么第二次.then()还是打印onResolved2 success undefined，原因是reject()也需要返回值，不写返回值也是默认走onResolved并且value为undefined，若第二次也需走onRejected失败，则抛出throw 或者 

return Promise.reject()即可。

这里需要注意一点就是throw 抛出，无论抛出什么，都是走onRejected



**Promise异常穿透**

当使用了.then()链式调用，可以在最后再指定失败的回调，前面任何操作出了异常，都会传到最后失败的回调中进行处理。

```js
new Promise((resolve,reject)=>{
    //resolve(1)
    reject('err1')
}).then(value=>{
    console.log('onResolved1...',value)
    return value++
}).then(value=>{
    console.log('onResolved2...',value)
    return value++
}).then(value=>{
    console.log('onResolved3...',value)
}).catch(reason=>{
    console.log('onRejected ',reason)
})
```

异常并不是直接到catch里。这里每个.then()只写了成功的回调onResolved，并没有写失败的回调onRejected，其实失败的回调相当与已经在失败的回调中加入reason=>{throw reason}，一层一层传到了catch里

```js
new Promise((resolve,reject)=>{
    //resolve(1)
    reject('err1')
}).then(value=>{
    console.log('onResolved1...',value)
    return value++
},reason=>{throw reason}).then(value=>{
    console.log('onResolved2...',value)
    return value++
},reason=>{throw reason}).then(value=>{
    console.log('onResolved3...',value)
},reason=>{throw reason}).catch(reason=>{
    console.log('onRejected ',reason)
})
```



**中断promise链**

当使用promise.then()链式调用时，在中间中断，不再调用后面的回调函数

方法：在回调函数中返回一个pending状态的promise，return new Promise(()=>{})

```js
new Promise((resolve,reject)=>{
    reject('reject1')
}).then(value=>{
    console.log('onResolved1...',value)
}).catch(reason=>{
    console.log(reason)
    return new Promise(()=>{})
}).then(value=>{
    console.log('onResolved2...',value)
},reason=>{
    console.log('onRejected2...',reason)
})

```



## 自定义Promise

```js
//MyPromise.js

const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

function MyPromise(){
   	this.status = PENDING //给MyPromise对象指定status属性，初始值为pending
	this.value = null //成功回调的数据
	this.reason = null //失败回调的数据
	this.onFulfilledCallbacks = [] //存放成功回调的数组
	this.onRejectedCallbacks = [] //存放失败回调的数组
    
    const resolve = (value) => {
		this.status = FULFILLED
		this.value = value
        // 一旦resolve执行，调用成功数组的函数
		this.onFulfilledCallbacks.forEach(item=>item(value))
	}
	const reject = (reason) => {
		this.status = REJECTED
		this.reason = reason
        // 一旦reject执行，调用失败数组的函数
		this.onRejectedCallbacks.forEach(item=>item(reason))
	}

	try{
        //立即同步执行excutor 
		excutor(resolve,reject)
	}catch(err){
		reject(err)
	}
}
//原型对象的then()方法，指定成功和失败的回调，返回一个新的MyPromise对象
MyPromise.prototype.then = function(onFulfilled,onRejected){
    //成功穿透
	onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value=>value
    //失败穿透
	onRejected = typeof onRejected === 'function' ? onRejected : reason=>{throw reason}
	const promise2 = new MyPromise((resolve,reject)=>{
		if(this.status === PENDING){
			this.onFulfilledCallbacks.push(value=>{
				setTimeout(()=>{
					try{
						const x = onFulfilled(value)
						resolvePromise(promise2,x,resolve,reject)
					}catch(err){
						reject(err)
					}
					
				})
			})
			this.onRejectedCallbacks.push(reason=>{
				setTimeout(()=>{
					try{
						const x = onRejected(reason)
						resolvePromise(promise2,x,resolve,reject)
					}catch(err){
						reject(err)
					}
				})
			})
		}else if(this.status === FULFILLED){
			setTimeout(()=>{
				try{
					const x = onFulfilled(this.value)
					resolvePromise(promise2,x,resolve,reject)
				}catch(err){
					reject(err)
				}
				

			})
		}else if(this.status === REJECTED){
			setTimeout(()=>{
				try{
					const x = onRejected(this.reason)
					resolvePromise(promise2,x,resolve,reject)
				}catch(err){
					reject(err)
				}
			})
		}
	})
	return promise2
}
/*
 1.如果抛出异常，return的promise就会失败，reason就是err
 2.如果回调函数返回的不是promise，return的promise就会成功，value就是返回值
 3.如果回调函数返回的是promise，return的promise的结果就是这个promise的结果
 */
function resolvePromise(promise2,x,resolve,reject){
    //不可返回同一个promise
	if(x === promise2){
		return reject(new TypeError('循环调用'))
	}
	let called = false
	if(typeof x === 'function' ||( typeof x === 'object' && x !== null)){
		try{
			const then = x.then	
			if(typeof then === 'function'){
				then.call(x,y=>{
					if(called) return 
					called = true
					resolvePromise(promise2,y,resolve,reject)
				},r=>{
					if(called) return 
					called = true
					reject(r)
				})
			}else {
				if(called) return 
				called = true
				resolve(x)
			}
		}catch(err){
			if(called) return 
			called = true
			reject(err)
		}
	}else{
		resolve(x)
	}
}


```

**安装promises-aplus-tests验证自定义promise**

```
yarn add promises-aplus-tests -D

```

```js
//MyPromise.js
...

MyPromise.deferred  = function() {
  const defer = {}
  defer.promise = new MyPromise((resolve, reject) => {
    defer.resolve = resolve
    defer.reject = reject
  })
  return defer
}
module.exports = MyPromise


```

输入命令

```
npx promises-aplus-tests MyPromise.js

```

无报错即成功