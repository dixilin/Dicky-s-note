---
title: redux基础及流程及示例以及redux-thunk介绍及使用
date: 2020-04-09 17:33:39
categories: 
    - redux
tags: 
    - redux
---

### redux

先上两张图

{%asset_img p1.png%}

{%asset_img p2.png%}

Store:数据仓库，保存数据的地方。

State:是1个对象，数据仓库里的所有数据都放到1个state里。

Action:1个动作，触发数据改变的方法。

Dispatch:将动作触发成方法

Reducer:是1个函数，通过获取动作，改变数据，生成1个新state，从而改变页面

### 示例

不扯别的，直接上代码，使用redux制作一个简单的todoList的demo，ui框架为antd，可进行增删操作。

首先看一眼目录结构

{%asset_img p3.png%}

```js
/*
 * index.js 入口文件
 */
 
import React from "react";
import ReactDOM from "react-dom";
import TodoList from "./TodoList";
import "antd/dist/antd.css";

ReactDOM.render(<TodoList />, document.getElementById("root"));

```

```jsx
/*
 * TodoList.jsx 
 */
import React, { Component } from "react";
import { List, Input, Button } from "antd"; //antd组件

import store from "./store";

import {
  getChangeInputAction,
  getHandleSubmitAction,
  getDeleteItemAction,
} from "./store/actionCreators";

class TodoList extends Component {
  constructor(props) {
    super(props);
    this.state = {};
    store.subscribe(() => {
      this.setState(this.state);
    });
  }
  render() {
    return (
      <div style={{ marginTop: "200px", paddingLeft: "200px" }}>
        <div>
          <Input
            style={{ width: "300px", marginRight: "10px" }}
            value={store.getState().inputValue}
            onChange={this.handleInputChange}
          />
          <Button type="primary" onClick={this.handleSubmit}>
            提交
          </Button>
        </div>
        <List
          style={{ width: "300px", marginTop: "10px" }}
          bordered
          dataSource={store.getState().list}
          renderItem={(item, index) => (
            <List.Item
              onClick={() => {
                this.deleteItem(index);
              }}
            >
              {item}
            </List.Item>
          )}
        />
      </div>
    );
  }

  handleInputChange = (e) => {
    const action = getChangeInputAction(e.target.value);
    store.dispatch(action);
  };
  handleSubmit = () => {
    const action = getHandleSubmitAction(store.getState().inputValue);
    store.dispatch(action);
  };
  deleteItem = (idx) => {
    const action = getDeleteItemAction(idx);
    store.dispatch(action);
  };
}

export default TodoList;

```



```js
/*
 * store/index.js 
 */
import {createStore} from 'redux'
import reducer from './reducer'

const store = createStore(reducer)

export default store
```



```js
/*
 * store/reducer.js 
 */
import {
  CHANGE_INPUT_VAL,
  SUBMIT_LIST,
  DELETE_ITEM,
} from "./actionTypes";

//默认数据
const defaultState = {
  inputValue: "欸",
  list: ["爸爸", "你是我的好爸爸"],
};

const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case CHANGE_INPUT_VAL: {
      const newState = JSON.parse(JSON.stringify(state));
      newState.inputValue = action.val;
      return newState;
    }
    case SUBMIT_LIST: {
      const newState = JSON.parse(JSON.stringify(state));
      newState.list.unshift(action.val);
      newState.inputValue = ''
      return newState;
    }
    case DELETE_ITEM: {
      const newState = JSON.parse(JSON.stringify(state));
      newState.list.splice(action.idx,1);
      return newState;
    }
    default:
      return state;
  }
};

export default reducer;

```



actionTypes.js只是为了防止代码书写错误定义的常量

```js
/*
 * store/actionTypes.js 
 */
export const CHANGE_INPUT_VAL = "change_input_val";
export const SUBMIT_LIST = "submit_list";
export const DELETE_ITEM = "delete_item";
```



actionCreator为统一管理的action，返回值为一个对象

```js
/*
 * store/actionCreator.js 
 */
import {
  CHANGE_INPUT_VAL,
  SUBMIT_LIST,
  DELETE_ITEM,
} from "./actionTypes";

export const getChangeInputAction = (val) => {
  return {
    type: CHANGE_INPUT_VAL,
    val
  };
};

export const getHandleSubmitAction = (val) => {
  return {
    type: SUBMIT_LIST,
    val
  };
};

export const getDeleteItemAction = (idx) => {
  return {
    type: DELETE_ITEM,
    idx
  };
};
```



store文件夹下的index.js为入口文件,创建一个store并导出,  组件派发一个dispatch事件告知store，传递的内容为一个action。reducer接收到从store传递过来的action，通过action.type区分，将state拷贝一份newState，将其所需要改变的值改变，并将整个newState返回，store接收到后将自动更新state。这时redux中的state值就已经改变。但是视图并未更新，需要重新render一次组件，store里有一个subscribe的方法用作监听，当监听到state改变时则触发，所以可以在store.subscribe( () =>{this.setState(this.state)})重新render。



### 注意：

dispatch只能处理同步，如果想要进行异步操作，可以用到redux-thunk 。redux-thunk中间件使我们能够编写异步 actionCreator，它返回的是函数，而不是对象 。也可使用redux-saga进行改造，redux-saga是基于generator函数，对异步操作进行统一管理。本文将采取redux-thunk进行异步操作



安装redux-thunk

```
yarn add redux-thunk
```

改写store/index.js，引入 redux的applyMiddleware以及引入

```js
/*
 * store/index.js 引入redux-thunk以及引入redux下的applyMiddleware
 */
import {applyMiddleware,createStore} from 'redux'
import reducer from './reducer'
import thunk from 'redux-thunk'

const store = createStore(
  reducer,
  applyMiddleware(thunk)
)

export default store
```

actionCreator添加异步操作

```js
/*
 * store/actionCreator.js 使用redux-thunk编写异步actionCreator
 */
import {
  CHANGE_INPUT_VAL,
  SUBMIT_LIST,
  DELETE_ITEM,
} from "./actionTypes";

export const getChangeInputAction = (val) => {
  return {
    type: CHANGE_INPUT_VAL,
    val
  };
};

export const getHandleSubmitAction = (val) => {
  return {
    type: SUBMIT_LIST,
    val
  };
};

export const getDeleteItemAction = (idx) => {
  return {
    type: DELETE_ITEM,
    idx
  };
};

//添加一个异步的actionCreator，点击后延迟1s添加进入列表
export const getHandleSubmitActionAsync = (val) => (dispatch) => {
  setTimeout(()=>{
      dispatch(getHandleSubmitAction)
  },1000)
};
```

TodoList.jsx添加一个按钮

```jsx
/*
 * TodoList.jsx 
 */
import React, { Component } from "react";
import { List, Input, Button } from "antd"; 

import store from "./store";

import {
  getChangeInputAction,
  getHandleSubmitAction,
  getDeleteItemAction,
  getHandleSubmitActionAsync //导入异步action
} from "./store/actionCreators";

class TodoList extends Component {
  constructor(props) {
    super(props);
    this.state = {};
    store.subscribe(() => {
      this.setState(this.state);
    });
  }
  render() {
    return (
      <div style={{ marginTop: "200px", paddingLeft: "200px" }}>
        <div>
          <Input
            style={{ width: "300px", marginRight: "10px" }}
            value={store.getState().inputValue}
            onChange={this.handleInputChange}
          />
          <Button type="primary" onClick={this.handleSubmit}>
            提交
          </Button>
          {//增加延迟点击按钮}
          <Button type="primary" onClick={this.handleSubmitAsync}>
            延迟一秒后提交
          </Button>
        </div>
        <List
          style={{ width: "300px", marginTop: "10px" }}
          bordered
          dataSource={store.getState().list}
          renderItem={(item, index) => (
            <List.Item
              onClick={() => {
                this.deleteItem(index);
              }}
            >
              {item}
            </List.Item>
          )}
        />
      </div>
    );
  }

  handleInputChange = (e) => {
    const action = getChangeInputAction(e.target.value);
    store.dispatch(action);
  };
  handleSubmit = () => {
    const action = getHandleSubmitAction(store.getState().inputValue);
    store.dispatch(action);
  };
  deleteItem = (idx) => {
    const action = getDeleteItemAction(idx);
    store.dispatch(action);
  };
  //添加点击事件
  handleSubmitAsync = () => {
    const action = getHandleSubmitActionAsync(store.getState().inputValue);
    //该action为一个函数而不是一个对象
    store.dispatch(action);
  };
}

export default TodoList;
```



### redux-thunk和redux DevTools冲突解决

改写store/index.js，啥也不说，复制粘贴即可

```js
/*
 * store/index.js 
 */
import { applyMiddleware, createStore, compose } from "redux";
import reducer from "./reducer";

import thunk from "redux-thunk";

//Redux Devtools配置
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__
  ? window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({})
  : compose;

//将Redux Devtools和redux-thunk合并
const enhancer = composeEnhancers(applyMiddleware(thunk));

const store = createStore(reducer, enhancer);

export default store;

```

