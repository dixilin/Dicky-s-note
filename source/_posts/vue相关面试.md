---
title: vue相关面试
date: 2020-05-22 18:12:03
categories: 
 - 面试
 - vue
tags: 
 - 面试
 - vue
---

## 编译模板

模板不是html，有指令、插值、js表达式，能实现判断、循环。

html是标签语言，只有js才能实现判断、循环

因此，模板一定是转换为某种js代码，即编译模板

```js
const compiler = require('vue-template-compiler')

const template = '<p>{{message}}</p>'
//with(this){return _c('p',[_v(_s(message))])}
//_c (h函数 h('p',{},[...])): createElement, _v:createTextNode ，_s:toString(message)
// 返回的是一个v-node
const res = compiler.compile(template)
console.log(res.render)
```

使用vue-loader，会在开发环境下编译模板



## Vue程序编译过程

1.使用runtime-compiler

template -> ast -> render -> virtual dom -> 真实dom



2.使用runtime-only （性能高，代码量少）

render -> virdual dom -> 真实dom

```js
import cpn from './cpn'
new Vue({
    el: '#app',
    render(createElement){
        //1.普通用法 createElement('标签',{标签属性},[''])
        //return createElement('h2',
        //	{class: 'box'},
        //	['hello world',createElement('button',['按钮'])]
        //)
        
        //2.传入组件对象
        return createElement(cpn)
    }
})
```

```js
new Vue({
    el: '#app',
    render:(h)=>createElement(h)
})
```

所有的template 都被webpack当中的vue-template-compiler处理成render函数，即vue-loader。



## vue父子组件生命周期执行顺序

created: 父 -> 子

mounted: 子  -> 父

beforeUpdate:  父 -> 子

updated: 子  -> 父

beforeDestory: 父 -> 子

destoryed: 子  ->  父



## 作用域插槽

父组件替换插槽的标签，但是内容由子组件提供

```html
<div id="app">
    <cpn>
    	<template slot-scope="slot">
        	<span v-for="item in slot.data">{{item}} - </span>
        </template>
    </cpn>
</div>

<template>
	<slot :data="pLanguages">
    	<ul>
            <li v-for="item in pLanguages">{{item}}</li>
        </ul>
    </slot>
</template>

<script>
	new Vue({
        el:'#app',
        components: {
            cpn: {
                template: '#cpn',
                data(){
                    return {
                        pLanguages: ['js','java','python','c','c#']
                    }
                }
            }
        }
    })
</script>
```

## 路由懒加载

```js
import Home from './components/Home'
Vue.use(VueRouter)
const routes = [
    {
        path: '/home',
        component: Home
    },
    {
        path: '/about',
        component: () => import(
            //路由懒加载模块分出来的js的名字设置
            /* webpackChunkName 'aboutChunk' */
            './components/About'
        )
    }
]
```

**懒加载写法**

```js
//amd
const About = resolve => require(['./components/About.vue'], resolve)
//es6
const About = () => import('./components/About')
```



## hash路由

hash变化会触发网页跳转，即浏览器前进、后退，不会刷新页面，永远不会提交到server端（完全前端控制）

```js
//监听hash变化
window.onhashChange = (e) => {
    console.log(e.oldURL,e.newURL)
    console.log('hash:',location.hash)
}
//首页页面加载获取hash
document.addEventListener('DOMContentLoaded',()=>{
    console.log('hash:',location.hash)
})

//js手动修改hash
btn.addEventListener('click',()=>{
    location.href = '#/hashTest'
})

```



## h5 history路由

用url规范的路由，但跳转时不刷新页面

history.pushState 

window.onpopstate

```js
//首页页面加载，获取path
document.addEventListener('DOMContentLoaded',()=>{
    console.log('path:',location.pathname)
})

//进入一个新路由
btn.addEventListener('click',()=>{
    const state = {name:'testPage'}
    history.pushState(state,'','testPage')
})

//监听浏览器前进后退
window.onpopstate = (e) => {
    console.log('onpopstate:',e.state,location.pathname)
}
```



## 异步组件

类似路由懒加载，将路由改成组件

```vue
<template>
	<div>
        <async-cpn v-if="isShow"/>
        <button @click="isShow = true"></button>
    </div>
</template>
<script>
	export default {
        components: {
            AsyncCpn: () => import('./components/AsyncCpn')
        },
        data(){
            return {
                isShow : false
            }
        }
    }
</script>
```



## 动态组件

用来动态挂载不同的组件 :is

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>示例</title>

</head>
<body>
    <div id="app">
        <component :is="currentView"></component>
        <button @click="handleChangView('A')">切换到A</button>
        <button @click="handleChangView('B')">切换到B</button>
        <button @click="handleChangView('C')">切换到C</button>
    </div>
<script src="https://unpkg.com/vue/dist/vue.min.js"></script>
<script>
   	new Vue({
        el: '#app',
        components: {
            comA: {
                template: '<div>组件A</div>'
            },
            comB: {
                template: '<div>组件B</div>'
            },
            comC: {
                template: '<div>组件C</div>'
            }
        },
        data: {
            currentView: 'comA'
        },
        methods: {
            handleChangView: function (component) {
                this.currentView = 'com' + component;
            }
        }
    })
</script>
</body>
</html>
```



## 路由导航守卫

```js
//动态生成document.title
const routes = [
    {
        path:'/',
        component: Home,
        meta: {
            title：'首页'
        },
        //局部导航守卫
        beforeEnter(to,from,next){
            next()
        }
    },
    {
        path:'/about',
        component: About,
        meta: {
            title：'关于'
        }
    },
]

const router = new VueRouter {
    routes
}
//前置钩子（hook） 路由跳转前
router.beforeEach((to,from,next)=>{
    //从from 跳转到 to
    document.title = to.matched[0].meta.title //嵌套路由使用
    next()
})

//后置钩子 跳转后
router.afterEach((to,from)=>{
    
})
```

## keep-alive

keep-alive是Vue内置的一个组件，可以使组件避免重新渲染。

​	两个属性：include - 字符串或正则，只有匹配的组件会被缓存

​						exclude - 字符串或正则，任何匹配的组件都不会被缓存

router-view也是一个组件，如果直接被包在keep-alive里面，所有路径匹配到的视图组件都会被缓存。

```vue
//若存在keep-alive,则多出两个生命周期函数activated(活跃)/deactivated(不活跃)
<keep-alive exclude="About,Home">
	<router-view/>
</keep-alive>

//About为该组件的name属性,Home为该组件的home属性
```

也可使用路由元信息替代exclude和include，在路由配置中添加meta: {isAlive: true} （meta: {isAlive: false}）

```vue
<router-view v-if="!$route.meta.isAlive"/>  //不缓存
<keep-alive >
	<router-view v-else/> //缓存
</keep-alive>
```



## mixin

多个组件有相同的逻辑，可使用mixin抽离出来。mixin内的data、methods、生命周期等所有方法都可供引入者使用

```vue
<template>
	<div>
        <p>{{this.name}}</p>
        <p>{{this.formatName}}</p>
        <button @click="this.showName">点我</button>
    </div>
</template>
<script>
	import mixinDemo from './mixinDemo.js'
    export default {
        mixins: [mixinDemo]
    }
</script>
```

```js
//mixinDemo.js
export default {
    data(){
        return {
            name: '小林'
        }
    },
    methods: {
        showName(){
            alert(this.name)
        }
    },
    mounted(){
        console.log('mixin mounted')
    },
    computed: {
        formatName(){
            return `我叫${this.name}`
		}
    }
}
```



## vue组件中的data为什么是函数形式

组件是一个可复用的vue实例，这也就意味着如何data是一个普通对象，那么所有复用这个实例的组件都将引用同一份数据，这就造成了数据污染。

这个时候如果我们将data封装成一个函数，我们在实例化组件的时候只是调用了这个函数生成的数据副本，这就避免了数据污染 



**根实例为什么可以是一个对象？**

根实例只有一个，不用担心造成多实例复用产生的状态污染。

 

## vue-cli(3.0以上) proxy代理配置

根目录新建vue.config.js

```js
module.exports = {
    devServer: {
		proxy: {
            //如果地址以/api开头，就会请求到http://127.0.0.1:3000
			'/api': {
                target: 'http://127.0.0.1:3000',
                changeOrigin: true, //设置为true时候，本地虚拟一个服务端接收请求并代理转发
                pathRewrite: {
                    '^/api': '' //发送请求时，将/api替换成''
                }
            }
        }
    }
}
```



## watch深度监听

```js
export default {
	data(){
        return {
            a: {
                b:{
                    name:'汪汪'
                }
            }
        }
    },
    //第一种：
    watch: {
        //监听的是a.b.name，深度监听
        'a.b.name':(newV,oldV)=>{
            
        }
    }
    //第二种
     watch: {
        a:{
    		deep: true, //开启深度监听 a对象里面任何数据变化都会触发
    		handler(newV,oldV){
                
            }
		}
    }
}
```



## 前端鉴权

1.有些请求需要token，可以在axios请求拦截中添加token

2.有些页面需登录才能查看，可以使用路由导航守卫判断是否存在token

3.后台管理系统侧边栏查看权限，根据权限渲染路由

​	登录后拿到token和对应账号的路由数据

```js
router.beforeEach((to,from,next)=>{
	if(token){
        const asyncRouter = [{path:'/..',role:'主管',...}]
        router.addRoutes(asyncRouter)
    }
})
```

### 前端权限控制思路

#### 菜单控制

在登录请求中，会得到权限数据（需后端给到），前端根据权限展示对应菜单

#### 界面控制

如果用户没有登录，手动在地址栏输入界面地址，则跳转到登录界面

如果用户已经登录，手动输入非权限内的地址，提示无权限或404

#### 按钮控制

在某个菜单界面中，还得感觉用户权限，展示出是否可进行操作的按钮

使用自定义指令

#### 请求和响应控制

如果用户通过非常规操作将某些禁用按钮变成可用，此时发的请求也应被前端拦截

请求拦截



## vue数据流（react数据流）

面试问vue数据流 不是问双向绑定

vue（react）中数据流是单向的，由父节点流向子节点，如果父节点的props发生了改变，react会递归遍历整个组件



## vue服务端渲染（ssr）

**为什么要做服务端渲染？**

vue官网说的狠明白，要做服务端渲染首先必须是有对应的需求，即对页面访问时间的绝对需求，如果只是一个简单的管理系统，区区几百ms的优化显得十分小题大做

服务端渲染在vue中有一个成熟的框架（nuxt.js），react对应的有next.js。



vue单页面应用渲染是从服务器获取js，在客户端将其解析生成html挂载于id为app的DOM元素上，这样做会存在两大问题:

1.由于资源请求量大，造成首屏加载缓慢，不利于用户体验（可利用懒加载优化）

2.由于页面内容通过js插入，对于内容性网站来说，搜索引擎无法抓取网站内容，不利于seo。

nuxt.js是一个基于vue。js的通用应用框架，预设了利用vue.js开发服务端渲染的应用所需要的各种配置。可以将html在服务端渲染，合成完整的html文件再输出给浏览器。



nuxt可以使渲染内容完全服务端化，解决seo不够友好、以及首屏渲染速度不够迅速的问题。



安装步骤（无需安装脚手架，node自带）：

```
npx create-nuxt-app <项目名>
```

启动

```
yarn dev
```

打包

```
yarn build
```



**nuxt和vue的一些区别**:

1.路由

​	nuxt按照pages文件夹的目录结构生成路由，localhost:3000/user/reg 相当于访问pages下的user中的reg.vue

​	vue需手动配置路由

2.入口页面

​	nuxt页面入口为layouts/default.vue

​	vue页面入口为src/App.vue

3.nuxt对应router-view，nuxt-link对应router-link

等等......



**nuxt中的asyncData方法**

asyncData里发送ajax，asyncData和生命周期这些是平级的，asyncData在执行时，其实是在服务端完成的，这个数据是在服务端渲染好了的。任何的获取数据的操作都会变成页面的跳转。（类似express中res.render()）



**使用axios需安装@nuxtjs/axios**

```
yarn add @nuxtjs/axios
```

此外还需其他配置自行百度，小林就不做过多阐述



## v-for和v-if哪个优先级高

v-for优先级高于v-if（源码中可查看，先执行的genFor优先于genIf）

如果同时出现v-for和v-if，每次渲染都会先循环后判断，无论v-if是否为true都无法避免循环，浪费了性能

如何避免？外层嵌套tempalte，在这层进行v-if判断，内部v-for



## key

```vue
<template>
	<div>
        <p v-for="item in items">{{item}}</p>
    </div>	
</template>
<script>
	export default {
        data() {
            return {
            	items: ['A','B','C','D','E']
        	}
        },
        mounted(){
            setTimeout(()=>{
                //2秒后在C前面插入F
                this.items.splice(2,0,'F')
            },2000)
        }
    }
</script>
```

{%asset_img p1.png%}

**如果不使用key**

{%asset_img p2.png%}

A->A，B->B，C->F，D->C，E->D，插入E。三次更新一次创建插入操作



**如果使用key**

```
//首尾假猜

//第一次循环patch A
A B C D E 
A B F C D E 

//第二次循环patch B
B C D E 
B F C D E 

//第三次循环patch E
C D E 
F C D E 

//第四次循环patch D
C D 
F C D 

//第五次循环patch C
C 
F C 
//老数组中全部处理完成为空，新数组剩下F，创建F并插入到C前面
```

没有进行更行操作，值进行一次创建插入操作



主要作用是为了高效更新虚拟dom，原理是vue在patch过程中(patch.js)通过key可以精准判断两个节点是否为同一个，从而避免频繁更新，使得整个patch过程更加高效，减少dom操作量，提高性能

另外不设置key可能会触发一些bug，例如splice删除操作



## vdom

由于想减少计算次数，把计算转移为js计算，js执行速度快。用js模拟dom结构，计算出最小的变更，再操作dom。

**用js模拟dom结构**

```html
<div id="div1" class="container">
    <p>vdom</p>
    <ul style="font-size:20px">
        <li>aaa</li>
    </ul>
</div>
```

```js
let vdom = {
    tag: 'div',
    props: {
        className: 'container',
        id: 'div1'
    },
    children: [
        {
            tag: 'p',
            children: 'vdom'
        },
        {
            tag: 'ul',
            props: {
                style: 'font-size:20px'
            },
            children: [
                {
                    tag: 'li',
                	children: 'aaa'
                }
                //...
            ]
        }
    ]
}
```



## diff算法

diff算法是vdom技术的对应产物，通过新旧vdom作对比（即diff），将变化的地方更新在真实dom上；另外，也需要diff高效的执行对比过程，从而降低时间复杂度为O(n)。

两棵树做diff，第一遍历tree1，第二遍历tree2，第三排序，1000个节点就需计算1亿次，时间复杂度为O(n3)，算法不可用。

**优化时间到O(n)**:

​	之比较同一层级，不跨级比较

​	tag不相同，则直接删掉重建，不再深度比较

​	tag和key，两者都相同，则认为是相同节点，不再深度比较



vue2.x中为了降低watcher粒度，每个组件只有一个watcher与之对应，只有引入了diff才能精确找到发生变化的地方

整体过程是遵循深度优先、同层比较的策略。两个节点之间比较会根据他们是否拥有子节点或者文本节点做不同操作。比较两组子节点是算法的重点，首先假设首尾节点可能相同做4次对比，如果没有找到相同节点才按照普通方式进行遍历查询，查询结束后再按情况处理剩下的节点；借助key可以精确找到相同点，使得整个patch过程非常高效

## 对MVC、MVP、MVVM的理解

### **web1.0时代**

在这时，并没有前端的概念，开发一个web应用多数采用asp.net/java/php编写，项目通常由aspx/jsp/php文件构成，每个文件中同时包含了html、css、js、以及后端代码，系统架构可能是这样的：

{%asset_img p3.png%}

这种架构简单快捷，但是jsp代码难以维护



为了让开发更加快捷，代码更容易维护，前后端职责更清晰。便衍生出MVC的开发模式和框架，前端展示以模板的形式出现。典型的框架就是Spring...

{%asset_img p4.png%}

使用这种分层架构，职责清晰，代码更容易维护。单这里的MVC仅限于后端，前后端形成了一定的分离，前端只完成了后端开发中的view层（切图仔出现）。

但是这种模式仍存在一些问题：

​	前端页面开发效率不高，前后端职责仍然不清



### **web2.0时代**

随着ajax技术的出现，前后端工程师的职责更加清晰。因为前端可以通过ajax与后端进行数据交互，因此，整体的架构图也变成了这样：

{%asset_img p5.png%}

通过ajax与服务器进行数据交换，前端只需要开发页面这部分内容，后端提供api接口。而且ajax可以使页面实现部分刷新，减少了服务端负载和流量的消耗，用户体验也更佳。这时，才开始有专职的前端工程师（终于能叫工程师了）。同时前端类库也慢慢开始发展，jQuery。

当然，此架构也存在一些问题：缺乏可行的开发模式承载更复杂的业务需求，页面内容全部糅杂在一起，一旦应用规模变大，就导致难以维护。因此，前端的mvc也随之而来



### 前端MVC

前端MVC与后端类似，具备View、Controller和Model

Model：负责保存应用数据，与后端数据进行同步

Controller：负责业务逻辑，根据用户行为对Model数据进行修改

View：负责视图展示，将Model中的数据展示出来

三者形成了这样的一个模型：

{%asset_img p5.png%}

这样的模型理论是可行，但是实际开发往往不会这么做，因为开发过程并不灵活。例如一个小小的时间操作，都必须经过这样一个流程，开发繁琐。

实际场景中，会看到另外易中模式：

{%asset_img p7.png%}

这种模式在开发中更加的灵活，backbone.js框架就是这种模式，但是这种灵活会造成更严重的问题：

​	1.数据流混乱：如下

 {%asset_img p8.png%}

​	2.View比较庞大而Controller比较单薄。很多开发者都会在view中写一些逻辑代码，逐渐导致view中的内容越来越庞大，而controller变得越来越单薄



既然有缺陷，就会有变革。前端变化中，似乎少了MVP这种模式，因为AngularJS早早将MVVM架构模式带入了前端。MVP虽然前端开发不常见，但是在安卓等原生开发中还是会使用到

### MVP

与MVC很接近，P指的是Presenter。Presenter可以理解为一个中间人，它负责View和Model之间的数据流动，防止View和Model之间直接交流

 {%asset_img p9.png%}

Presenter负责和Model和View进行双向交互。这种方式相比MVC少了一些灵活，使得View变成被动视图，并且本身变得很小。虽然分离了Model和View，但是应用规模变大后导致Presenter体积增大难以维护。

### MVVM

MVVM（Model-View-ViewModel），ViewModel可以理解为Presentor的进阶。如图：

{%asset_img p10.png%}

ViewModel通过实现一套数据响应机制自动响应Model中的数据变化。同时ViewModel会实现一套更新策略自动将数据变化转化为视图更新。

通过事件监听响应View中用户交互修改Model数据。

这样在ViewModel中就减少了大量DOM操作。

MVVM在保持View和Model松耦合的同时，还减少了维护它们的关系，使我们专注于业务逻辑，兼顾开发效率和可维护性

### 总结

三者都是框架模式，设计目的都是为了解决Model和View的耦合

MVC出现较早，主要使用在服务端，如Spring MVC，在前端领域早期也有应用，如bockbone.js。优点是分层清晰，缺点是数据流混乱

MVP是MVC的进化形式，Presenter作为中间层负责Model和View的通信，解决了两者耦合为题，但P层过于臃肿会导致维护困难

MVVM在前端领域广泛应用，不仅解决Model和View耦合问题，还同时解决了维护两者映射关系的大量繁杂代码和DOM操作代码，在提高开发效率、可读性同时还保持了优越的性能



## 实现vue中的mvvm

**实现功能：**

```
v-model、v-text、v-html、{{}}、v-on:click...、computed、beforeCreate、created、beforeMount、mounted、beforeUpdate、updated。
```

代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <div id="app">
    <p>
      <input type="text" v-model="personalMsg.name">
    </p>
    <h1>{{personalMsg.age}}-{{personalMsg.name}}</h1>
    <i v-text="personalMsg.name"></i>
    <p v-html="html"></p>
    {{getMsg}}
    <button v-on:click="changeValue">click me</button>
  </div>
  <script src="mvvm.js"></script>
  <script>
    const vm = new MyVue({
      el: '#app',
      data: {
        personalMsg: {
          name: 'lin',
          age: 18
        },
        msg:'textMsg',
        html:'<b>htmlMsg</b>'
      },
      methods: {
        changeValue(e){
          this.personalMsg.name = 999
        }
      },
      computed: {
        getMsg(){
          return `我叫${this.personalMsg.name},今年${this.personalMsg.age}岁`
        }
      },
      beforeCreate(){
      },
      created(){
      },
      beforeMount(){
      },
      mounted(){
      },
      beforeUpdate(){
        console.log('更新前')
      },
      updated(){
        console.log('更新了')
      }

    })
  </script>
</body>

</html>
```

```js
//mvvm.js
class MyVue {
  constructor(options) {
    //生命周期函数
    const { beforeCreate, created, beforeMount, mounted ,beforeUpdate,updated} = options;

    //beforeCreate
    if (typeof beforeCreate === "function") {
      beforeCreate.call(this);
    }
	
    this.$data = options.data;

    //created
    if (typeof created === "function") {
      created.call(this);
    }
    this.$el = document.querySelector(options.el);
    this.$methods = options.methods;
    this.$computed = options.computed;
      
    /*
     * 如果实例上有beforeUpdate和updated生命周期函数，则给实例绑定$beforeUpdate和$updated
     */
    //beforeUpdate
    if (typeof beforeUpdate === "function") {
      this.$beforeUpdate = beforeUpdate
    }
    //updated
    if (typeof updated === "function") {
      this.$updated = updated
    }

    if (this.$data) {
      //把数据全部转化成用Object.defineProperty来定义
      new Observer(this.$data);
	  //代理computed
      this.getComputed(this.$computed);
      //把数据操作代理到vm实例上 vm.$data
      this.proxy(this.$data);
      //把method方法代理到vm实例上 vm.$methods
      this.proxy(this.$methods);

      //beforeMount
      if (typeof beforeMount === "function") {
        beforeMount.call(this);
      }
	  
      //编译
      new Compiler(this.$el, this);

      //mounted
      if (typeof mounted === "function") {
        mounted.call(this);
      }
    }
  }

  proxy(data) {
    for (let key in data) {
      Object.defineProperty(this, key, {
        set(newVal) {
          data[key] = newVal;
        },
        get() {
          return data[key];
        },
      });
    }
  }

  getComputed(computed) {
    for (let key in computed) {
      Object.defineProperty(this.$data, key, {
        get() {
          return computed[key].call(this);
        },
      });
    }
  }
}

//编译模板
class Compiler {
  constructor(el, vm) {
    this.el = el;
    this.vm = vm;
	
    //得到内存节点fragment
    const fragment = this.node2Fragment(el);
	//编译节点
    this.compile(fragment);
    //将编译好的节点插入到el下
    this.el.appendChild(fragment);
  }
  //把节点移动到内存中  
  node2Fragment(node) {
    //创建一个文档碎片
    let fragment = document.createDocumentFragment();
    let firstChild;
    // 将原生节点拷贝到fragment
    while ((firstChild = node.firstChild)) {
      fragment.appendChild(firstChild);
    }
    return fragment;
  }
  //判断节点是否为元素节点
  isElementNode(node) {
    return node.nodeType === 1;
  }
  //编译内存中的dom
  compile(node) {
    const childNodes = [...node.childNodes];
    childNodes.forEach((child) => {
      //判断子节点是否为元素
      if (this.isElementNode(child)) {
        this.compileElement(child);
        //子节点为元素递归编译
        this.compile(child);
      } else {
        //子节点为文本
        this.compileText(child);
      }
    });
  }
  //属性名开头是否为v-
  isDirective(attrName) {
    return attrName.substr(0, 2) === "v-";
  }
  //编译元素
  compileElement(node) {
    //获取元素属性
    const attrs = [...node.attributes];
    attrs.forEach((attr) => {
      //name:属性名 value:属性值,expr即表达式 可能为a 也可能为a.b
      const { name, value: expr } = attr; 
      if (this.isDirective(name)) {
        const [, directive] = name.split("-");
        const [directiveName, eventName] = directive.split(":");
        //需调用不同指令来处理  CompileUtil.model()、CompileUtil.html() ...
        CompileUtil[directiveName](expr, node, this.vm, eventName);
      }
    });
  }
  //编译文本
  compileText(node) {
    const { textContent: expr } = node;
    //判断当前文本内容是否包含 {{ }} 表达式
    if (/\{\{(.*?)\}\}/.test(expr)) {
      CompileUtil.text(expr, node, this.vm);
    }
  }
}
//编译通用方法
CompileUtil = {
  getVal(expr, vm) {
    //vm:实例 expr:表达式 personalData.name
    let data = vm.$data;
    const exprs = expr.split(".");
    exprs.forEach((key) => {
      data = data[key];
    });
    return data;
  },
  //输入框设置value
  setVal(expr, vm, newVal) {
    let data = vm.$data;
    const exprs = expr.split(".");
    exprs.forEach((key, idx) => {
      if (exprs.length === idx + 1) {
        return (data[key] = newVal);
      }
      data = data[key];
    });
  },
  //是否为v-text指令，而不是{{}}
  isHas_vText_directive(node) {
    if (node.nodeType !== 1) return false;
    const attrs = [...node.attributes];
    return attrs.find((attr) => attr.name === "v-text");
  },
  //v-html设置
  html(expr, node, vm) {
    const val = this.getVal(expr, vm);
    const fn = this.updater.htmlUpdater;
    //添加watcher
    new Watcher(vm, expr, (newVal) => {
      fn(node, newVal);
    });
    fn(node, val);
  },
  //重新
  getTextVal(expr, vm) {
    return expr.replace(/\{\{(.*?)\}\}/g, (...args) => {
      return this.getVal(args[1], vm);
    });
  },
  //v-text和{{}}设置
  text(expr, node, vm) {
    let val;
    if (this.isHas_vText_directive(node)) {
      //v-text
      val = this.getVal(expr, vm);
      new Watcher(vm, expr, (newVal) => {
        fn(node, newVal);
      });
    } else {
      //{{}}  //替换匹配到的一个或多个{{}}内的内容
      val = expr.replace(/\{\{(.*?)\}\}/g, (...args) => {
        new Watcher(vm, args[1], () => {
          fn(node, this.getTextVal(expr, vm));
        });
        return this.getVal(args[1], vm);
      });
    }
    const fn = this.updater.textUpdater;
    fn(node, val);
  },
  model(expr, node, vm) {
    const val = this.getVal(expr, vm);
    const fn = this.updater.modelUpdater;
    //给输入框加一个观察者，如果数据更新会触发，拿新值赋予输入框
    new Watcher(vm, expr, (newVal) => {
      fn(node, newVal);
    });
    //监听input框输入事件，修改value
    node.addEventListener("input", (e) => {
      this.setVal(expr, vm, e.target.value);
    });

    fn(node, val);
  },
  //v-on
  on(expr, node, vm, eventName) {
    node.addEventListener(eventName, (e) => {
      vm.$methods[expr].call(vm, e);
    });
  },
  updater: {
    //数据插入到节点中
    modelUpdater(node, val) {
      node.value = val;
    },
    textUpdater(node, val) {
      node.textContent = val;
    },
    htmlUpdater(node, val) {
      node.innerHTML = val;
    },
  },
};
// 数据监听器
class Observer {
  constructor(data) {
    this.observe(data);
  }

  observe(data) {
    //不是对象直接return
    if (typeof data !== "object" || !data) return;
    // 取出所有属性遍历
    for (let key in data) {
      this.defineReactive(data, key, data[key]);
    }
  }
  //数据劫持
  defineReactive(obj, key, val) {
    const dep = new Dep(); //给每一个属性都加上发布订阅的功能
    this.observe(val); //递归调用
    Object.defineProperty(obj, key, {
      enumerable: true, // 可枚举
      configurable: false, // 不能再define
      get() {
        //创建watcher时会取到对应内容，并把watcher放到全局上
        Dep.target && dep.addSub(Dep.target);
        return val;
      },
      set: (newVal) => {
        if (newVal === val) return;
        this.observe(newVal); //如果newVal是对象，重新赋值会没有get set，需重新赋予get set
         //先赋值
        val = newVal;
        //后通知
        dep.notify();
      },
    });
  }
}
//观察者（发布订阅） vm.$watch(vm,expr,(newVal)=>{})
class Watcher {
  constructor(vm, expr, cb) {
    this.vm = vm;
    this.expr = expr;
    this.cb = cb;
	//默认先存放一个老值
    this.oldVal = this.get();
  }

  get() {
    //把自己放在Dep.target上，也就是Dep上有了watcher
    Dep.target = this; 
    //取值，把观察者和数据关联起来
    const val = CompileUtil.getVal(this.expr, this.vm);
    Dep.target = null;
    return val;
  }
  //数据变化后会调用观察者的update
  update() {
    const newVal = CompileUtil.getVal(this.expr, this.vm);
    if (newVal === this.oldVal) return;
    
    //生命周期 更新前 beforeUpdate
    if(vm.$beforeUpdate && typeof vm.$beforeUpdate === 'function'){
      vm.$beforeUpdate()
    }

    this.cb(newVal);

    //生命周期 更新后 updated
    if(vm.$updated && typeof vm.$updated === 'function'){
      vm.$updated()
    }
  }
}
//订阅者
class Dep {
  constructor() {
    //存放所有的watcher
    this.subs = [];
  }
  //添加watcher
  addSub(watcher) {
    this.subs.push(watcher);
  }
  //通知发布
  notify() {
    this.subs.forEach((watcher) => watcher.update());
  }
}

```

