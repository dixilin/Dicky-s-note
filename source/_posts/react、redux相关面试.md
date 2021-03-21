---
title: react、redux相关面试
date: 2020-06-08 22:18:45
tags:
 - 面试
 - react
categories: 
 - 面试
 - react
---

## setState

### **setState是不可变值**

数组的pop、push、splice等一系列返回值会改变原数组都不可，

```jsx
class Test extends React.Component{
 state = {
    num: 0,
    arr: [0,1,2]
  }
  handleClick = () => {
    this.setState(()=>{
      return {
        arr: this.state.arr.push(3)
      }
    },()=>{
      console.log(this.state.arr) //第一次5 第二次报错 
    })
  }
  render(){
    return (
      <div>
        <button onClick={this.handleClick}>+1</button>
      </div>
    )
  }
}
```

可通过克隆新数组进行添加或删除等一系列操作

```jsx
class Test extends React.Component{
 state = {
    num: 0,
    arr: [0,1,2]
  }
  handleClick = () => {
    const newArr = [...this.state.arr]
    newArr.push(3)
    this.setState(()=>{
      return {
        arr: newArr
      }
    },()=>{
      console.log(this.state.arr)
    })
  }
  render(){
    return (
      <div>
        <button onClick={this.handleClick}>+1</button>
      </div>
    )
  }
}
```



### setState可能是同步也可能是异步

普通情况下是异步执行。可在第二个参数获取更新后的值

在setTimeout()情况下或自定义dom事件（不是onXxx这种），setState是同步执行



### setState可能会被合并

```jsx
class Test extends Component {
    state = {
        num : 0
    },
    componentDidMount(){
        this.setState({
            num: this.state.num + 1
        })
        console.log(this.state.num) //0
        this.setState({
            num: this.state.num + 1
        })
        console.log(this.state.num) //0
        setTimeout(()=>{
            this.setState({
                num: this.state.num + 1
            })
            console.log(this.state.num) //2
        })
        setTimeout(()=>{
            this.setState({
                num: this.state.num + 1
            })
            console.log(this.state.num) //3
        })
    }
}
```

输出结果为 0 0 2 3.

由于setState传入的是对象且这两个setState是异步执行，所以前两个this.setState会合并，就是这两次 num+1 只执行了一次，num为1，但是打印的是0。

在setTimeout里的为同步执行，所以第一次setTimeout和第二次setTimeout打印的分别为2，3



**如果想要setState不被合并，只需将第一个参数改为函数即可**

```jsx
class Test extends Component {
    state = {
        num : 0
    },
    componentDidMount(){
        this.setState((prev,props) => {
            return {
                num: prev.num + 1
            }
        })
        console.log(this.state.num) //0
        this.setState((prev,props) => {
            return {
                num: prev.num + 1
            }
        })
        console.log(this.state.num) //0
        setTimeout(()=>{
            this.setState({
                num: this.state.num + 1
            })
            console.log(this.state.num) //3
        })
        setTimeout(()=>{
            this.setState({
                num: this.state.num + 1
            })
            console.log(this.state.num) //4
        })
    }
}
```

打印结果为0 0 3 4



### setState主流程

{%asset_img p1.png%}

### batchUpdates机制（isBatchingUpdates）

```js
class ListDemo extends Component {
    state = {
        count: 1
    }
    increse = () => {
        // 异步
        // 开始： 处于batchUpdate 
        // isBatchingUpdates = true
        this.setState({
            count: this.state.count +1
        },()=>{
            //回调函数中拿到最新state
            console.log('count by cb', this.state.count)
        })
        // 结束
        // isBatchingUpdates = false
        // 异步，拿到的是上一次的值
        console.log('count', this.state.count)
    }
    
    increse2 = () => {
        // setTimeout中 setState是同步
        // 开始： 处于batchUpdate 
        // isBatchingUpdates = true
		setTimeout(()=>{
            // 延迟操作 此时 isBatchingUpdates = false
            this.setState({
                count: this.state.count +1
            })
            //可拿到最新值
            console.log('count in setTimeout', this.state.count)
        })
        // 结束
        // isBatchingUpdates = false
    }
    
    componentDidMount(){
        // 开始： 处于batchUpdate 
        // isBatchingUpdates = true
        document.body.addEventListener('click',()=>{
            // 此时 isBatchingUpdates 为 false
            this.setState({
                count: this.state.count + 1
            })
            // 拿到最新值
            console.log('count in body event',this.state.count)
        })
        // 结束
        // isBatchingUpdates = fale
    }
}
```

setState无所谓同步还是异步，关键是要是否能命中batchUpdate机制，判断isBatchingUpdates是否为true。

如果isBatchingUpdates为true，则属于异步；如果为false；则属于同步



**哪些能命中batchUpdate机制**：

​	生命周期以及它调用的函数

​	React中注册的事件（和它调用的函数）

​	React可以“管理”的入口



**哪些不能命中batchUpdate机制：**

​	setTimeout、setInterval等（和它调用的函数）

​	自定义dom事件（和它调用的函数）

​	React管理不到的入口



## 受控组件、非受控组件

### 受控组件

假设我们现在有一个表单，表单中有一个input标签，input的value值必须是我们设置在state中的值，然后，通过onChange触发事件来改变state中保存的value值，这样形成一个循环的回路影响。也可以说是React负责渲染表单的组件仍然控制用户后续输入时所发生的变化。

```jsx
class Cpn extends Component {
    state = {
        inputVal: 'aaa'
    }
	handleChange = (e) => {
        this.setState(()=>{
            return {
                inputVal: e.target.value
            }
        })
    }
	render(){
        return (
        	<div>
            	<input value={this.state.inputVal} onChange={this.handleChange} />
            </div>
       	)
    }
}
```

就像上面这样，input中的value值通过state值获取，onChange事件改变state中的value值，input中的value值又从state中获取。。。 



### 非受控组件

非受控也就意味着我可以不需要设置它的state属性，而通过ref来操作真实的DOM。 

通过设置defaultValue来设置input的默认值，通过ref获取节点

```jsx
class Cpn extends React.Component {
  constructor(props){
    super(props)
    this.inputRef = React.createRef()
  }
  btnClick = () => {
    alert(this.inputRef.current.value)
  }
  render() {
    return (
      <div>
        <input defaultValue="啦啦啦" ref={this.inputRef}/>
        <button onClick={this.btnClick}>提交</button>
      </div>
    )
  }
}
```



## Portals

**能将子节点渲染到父组件的 DOM层次之外**

```js
ReactDOM.createPortal(child, container)
```

第一个参数（child）是任何可渲染的 React 子元素，例如一个元素，字符串或 片段(fragment)。第二个参数（container）则是一个 DOM 元素。

对于 portal 的一个典型用例是当父组件有 overflow: hidden 或 z-index 样式，但你需要子组件能够在视觉上 “跳出(break out)” 其容器。例如，对话框以及提示框。

```jsx
class PortalsDemo extends Component {
	render(){
        //不适用Portals
        //return <div className="modal">模态框</div>
        
        //使用Portals使得modal和根组件同级，放在body下
        return ReactDOM.createPortal(
        	<div className="modal">模态框</div>,
            document.body 
        )
    }
}

```



## 异步组件

16.6版本以前可选择loadable，16.6后版本选择React自带的lazy、Suspense

```jsx
const LazyCpn = React.lazy(() => import('./LazyCpn'))

class LazyDemo extends React.Component {
    render(){
        return (
        	<div>
            	<p>异步组件</p>
                <React.Suspense fallback={<div>loading...</div>}>
                	<LazyCpn />
                </React.Suspense>
            </div>
        )
    }
}

```

fallback是异步加载LazyCpn时显示的过渡loading。



## 性能优化

### shouldComponentUpdate

react默认父组件有更新，子组件则无条件也更新。可使用shouldComponentUpdate来阻止子组件重新渲染。

**基本用法**：

```jsx
shouldComponentUpdate(nextProps, nextState){
	if(nextState.count !== this.state.count){
		return true //重新渲染
	}
	return false //不重新渲染
}

```

**总结**：

默认返回true，即react默认重新渲染所以子组件

必须配合不可变值一起使用

写项目时可先不用，需性能优化再考虑使用



### PureComponent、React.memo

就是shouldComponentUpdate实现了浅比较（只比较第一层）

类组件中使用PureComponent

```jsx
class PureCpn extends React.PureComponent {
    
}
export default PureCpn

```

函数组件中使用React.memo

```jsx
const MemoCpn = React.memo(()=>{
	
})
export default MemoCpn

```



### 不可变值 immutable.js

基于共享数据（不是深拷贝），性能较好，速度较快

immutable对象是不可直接赋值的对象，它可以有效的避免错误赋值的问题 

安装

```
yarn add immutable

```

使用：

将js对象转成immutable对象 

```js
import { fromJS } from 'immutable';
const defaultState = fromJS({
  todoList: []
});

```

 获取属性 

```js
state.get('todoList'); // 获取store中的todoList
state.get(['Main', 'todoList']); // 获取Main组件中store的todoList

```

改变属性

```js
state.set('todoList', action.value);  // 设置单个属性值
// 设置多个属性
state.merge({
  todoList: fromJS(action.value), // 由于action.value是js对象所以要转成immutable对象
});

```

将immutable对象转成js对象

```js
state.get('todoList').toJS(); // 把todoList转成js数组

```



## 组件公共逻辑抽离

### 高阶组件 HOC

高阶组件，Higher Order Component，不是一种功能，而是一种模式

高阶组件是对一个组件进行包装，返回一个新的组件，本质上就是一个函数。

为什么使用？因为有的组件需要复用，但有少部分区别，我们可以将公用的写在高阶组件内，而通过往高阶组件（这个函数）传额外的参数去动态改变这个组件在不同场景使用下所的一些小差别。

过多高阶组件的问题？

高阶组件地狱，可使用hooks减少地狱

```
<A>
	<B>
		<C>
			<D/>
		</C>
    </B>
</B>

```



### Render Props

通过一个函数将class组件的state作为props传递给纯函数组件

```jsx
const App = () => {
    <Factory render={
        (props) => <p>{props.a}...</p>	    
    } />
}
class Factory extends Component {
    constructor(props){
        this.state = {
            //多个组件公共逻辑的数据
        }
    }
    render(){
        return <div>{this.props. (this.state)}</div>
    }
}

```



## redux单项数据流

dispatch(action)

reducer -> newState

subscribe 触发通知



## action如何处理异步

使用redux-thunk

```js
//同步
export const addTodo = text => {
	return {
		type: 'ADD_TODO',
        text
	}
}

//异步
export const addTodoAsync = text => {
    //返回函数，其中有dispatch参数
	return (dispatch) => {
        setTimeout(()=>{
            //异步执行 action
            dispatch(addTodo('text'))
		},500)
    }
}

```



## redux中间件

正常情况下，redux的执行过程是

state值发生改变，action触发dispatch，然后reducer接收到新数据返回给store，视图更新

中间件就是在派发dispatch之前的一系列操作

**实现logger**

```js
const next = store.dispatch
store.dispatch = function dispatchLog(action){
    console.log('dispatching',action)
    next(action)
    console.log('next state',store.getState())
}

```



## 路由

```jsx
import {
	HashRouter as Router, //hash
    //BrowserRouter as Router, //history
    Switch,
    Route,
    Link,
    useParams
} from 'react-router-dom'

const Project = () => {
    const {id} = useParams()
    return (
    	<div>
        	<p>{id}</p>
            <link to="/">回到首页</link>
        </div>
    )
}

const RouterComponent = () => {
    return (
    	<Router>
        	<Switch>
            	<Route exact path="/">
                	<Home />
                </Route>
                <Route path="/about">
                	<About />
                </Route>
                <Route exact path="/project/:id">
                	<About />
                </Route>
            </Switch>
        </Router>
    )
}

```

路由懒加载配置：

和异步组件一样都需用到lazy和Suspense

```jsx
import {
	HashRouter as Router,
    Switch,
    Route
} from 'react-router-dom'
import React,{Lazy,Suspense} from 'react'
const Home = import('./Home')
//懒加载
const About = lazy(() => import('./About'))
const App = () => (
    <Router>
    	<Suspense fallback={<div>loading...</div>}>
        	<Switch>
            	<Route exact path="/" Component={Home}></Route>
                <Route path="/about" Component={About}></Route>
            </Switch>
        </Suspense>
    </Router>
)

```



## JSX

jxs等同于于Vue的模板，Vue的模板不是html，jxs也不是js。

jsx由babel提供编译，template由vue-loader提供编译

```jsx
const App1 = <div id="root">
	<P class="p">我是P</P>
</div>
//等价于
const App2 = React.createElement(
    'div',{id:'root'},React.createElement(
        'p',{class:'p'},'我是p')
)

```

```jsx
const List1 = <ul>
	{
     	this.state.list.map(item)=>{
            return <li key={item.id}>名字是：{item.name}</li>
        }         
    }      
</ul>
//等价于
const List2 = React.createElement('ul',null,(void0).state.list.map(function(item){
    return React.createElement('li',{key:item.id},'名字是'+item.name)
}))

```

所以jsx本质就是将jsx语法编译成一堆React.createElement...



## 合成事件

所有事件挂载到document上

event事件源对象不是原生的，是SyntheticEvent合成事件对象

和Vue的事件不同，和dom事件也不同

{% asset_img p2.png %}

原生事件对象可在event.nativeEvent拿到



**为何要使用合成事件机制？**

更好的兼容性和跨平台

挂载到document上，减少内存消耗，避免频繁解绑（事件委托）

方便事件的统一管理（如事物机制）



## transaction事务机制

先执行开始的逻辑，再执行函数体，最后执行结束的逻辑

```js
transaction.initialize = () => {
    console.log('开始')
}
transaction.close = () => {
    console.log('结束')
}
const method = () => {
    console.log('一系列相关操作')
}

transaction.perform(method)
//输出： 开始 -> 一系列相关操作 -> 结束

```

只是一种机制，也服务于batchUpdate

```js
class Demo extends Component {
    handle = () => {
        // 开始 处于 batchUpdate
        // isBatchingUpdates = true
        
        // 其他任何操作
        ...
        
        // 结束
        // isBatchingUpdates = false
    }
}

```



## 组件渲染和更新过程

**渲染**

jsx、即createElement函数执行，生成Vnode。然后通过patch(elem,vnode)触发组件渲染

**更新**

setState(newState) --> dirtyComponents（可能有子组件）

render()生成newVnode

patch(vnode,newVnode)



## fiber

组件更新时，patch被拆分为两个阶段：

 1.reconciliation阶段：执行diff算法、纯js计算

 2.commit阶段：将diff结果渲染到dom上

关于fiber，是react内部运行机制，开发者体会不到

js是单线程，且和dom渲染公用一个线程。那么当组件过于复杂，组件更新时计算和渲染压力都很大，同时再有Dom操作需求（动画、滚动条滑动、鼠标拖拽等），浏览器很容易卡顿

**解决方案**

将reconciliation阶段进行任务拆分（commit无法拆分）

Dom需要渲染时暂停，空闲时恢复

通过window.requestIdleCallback这个api（如果浏览器不支持则不使用）



## 虚拟dom及diff算法

 https://www.jianshu.com/p/af0b398602bc 

