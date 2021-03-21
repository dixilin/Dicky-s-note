---
title: React_组件传值、事件
date: 2020-04-08 16:49:26
categories: 
    - react
tags: 
    - react
---

## 组件传值

### 父传子

在父组件添加msg属性,子组件通过props.msg拿到父组件传递过来的值并显示

```jsx
const Parent = () => {
	return (
		<div>
			<Child msg={'我是你爸爸'} />
		</div>
	)
}

//子
const Child = (props) => {
	return (
		<div>
			{props.msg}
		</div>
	)
}
```

### 子传父

子组件需要将msg传递给父组件并显示，先在子组件绑定一个点击事件handleClick，在父组件传递一个函数getChildMsg给子组件，子组件的点击事件handleClick调用父组件传递过来的函数getChildMsg，并将msg作为getChildMsg的形参传递给夫组件，父组件的getMsg方法即可得到子组件传递过来的msg值

```jsx
class Parent extends Component {
  constructor(props){
    super(props)
    this.state = {
      childMsg : ''
    }
  }
  render() {
    return (
      <div style={style}>
        {this.state.childMsg}
        <Child getChildMsg={this.getMsg} />
      </div>
    )
  }
  getMsg = (msg) => {
    console.log(msg)
    this.setState({
      childMsg: msg
    })
  }
}

class Child extends Component {
  constructor(props) {
    super(props)
    this.state = {
      msg: '爸爸，爸爸，我是你儿子呀'
    }
  }
  render() {
    return (
      <div>
        <button onClick={this.handleClick}>我要告诉我爸爸</button>
      </div>
    )
  }
  handleClick = () => {
    this.props.getChildMsg(this.state.msg)
  }
}
```

## 事件

特点：

1React事件，绑定事件的命名，驼峰命名法。

2{},传入1个函数，而不是字符串

事件对象：React返回的事件对象是代理的原生的事件对象，如果想要查看事件对象的具体值，必须之间输出事件对象的属性。

注意：

原生，阻止默认行为时，可以直接返回return false；

React中，阻止默认必须e.preventDefault();

React事件传参数：

```
<button  onClick={(e)=>{this.parentEvent1('msg:helloworld',e)}}>提交</button>
```

