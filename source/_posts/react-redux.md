---
title: react-redux、改写TodoList
date: 2020-04-10 17:43:58
categories: 
    - redux
tags: 
    - redux
---

安装命令

```
yarn add react-redux
```

react-redux里的几个核心api



Provider：组件，自动将store里的state和组件进行关联。

MapStatetoProps：将store的state映射到组件的里props

mapdispatchToProps: 将store中的dispatch映射到组件的props里，实现了方法的共享。

Connect：方法，将组件和数据（方法）进行连接



OK，废话不多说，上代码，store里的文件无需修改，改变的部分只有index.js以及Todolist.jsx。

首先index.js，从react-redux引入一个 Provider组件包裹住TodoList，并将store传入这个组件内

```js
//index.js
import React from "react";
import ReactDOM from "react-dom";
import TodoList from "./TodoList";
import "antd/dist/antd.css";

import { Provider } from "react-redux";
import store from "./store";

const App = () => {
  return (
    <Provider store={store}>
      <TodoList />
    </Provider>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```



TodoList.jsx,无需再导入store，并且无需再进行subscribe监听。需从react-redux导入connect，导出connect()(TodoList)，该组件是通过connect方法自动生成的容器组件。

connect方法接受两个参数：mapStateToProps和mapDispatchToProps。它们定义了 UI 组件的业务逻辑。前者负责输入逻辑，即将state映射到 UI 组件的参数（props），后者负责输出逻辑，即将用户对 UI 组件的操作映射成 Action。

```jsx
import React, { Component } from "react";
import { List, Input, Button } from "antd";

// import store from "./store";

import { connect } from "react-redux";

import {
  getChangeInputAction,
  getHandleSubmitAction,
  getDeleteItemAction,
} from "./store/actionCreators";

class TodoList extends Component {
  render() {
    return (
      <div style={{ marginTop: "200px", paddingLeft: "200px" }}>
        <div>
          <Input
            style={{ width: "300px", marginRight: "10px" }}
            value={this.props.inputValue}
            onChange={this.props.handleInputChange}
          />
          <Button
            type="primary"
            onClick={() => {
              this.props.handleSubmit(this.props.inputValue);
            }}
          >
            提交
          </Button>
        </div>
        <List
          style={{ width: "300px", marginTop: "10px" }}
          bordered
          dataSource={this.props.list}
          renderItem={(item, index) => (
            <List.Item
              onClick={() => {
                this.props.deleteItem(index);
              }}
            >
              {item}
            </List.Item>
          )}
        />
      </div>
    );
  }
}

const mapStateToProps = (state) => {
  return {
    inputValue: state.inputValue,
    list: state.list
  };
};

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    handleInputChange(e) {
      const action = getChangeInputAction(e.target.value);
      dispatch(action);
    },
    handleSubmit(val) {
      const action = getHandleSubmitAction(val);
      dispatch(action);
    },
    deleteItem(idx) {
      const action = getDeleteItemAction(idx);
      dispatch(action);
    },
  };
};

export default connect(mapStateToProps, mapDispatchToProps)(TodoList);

```

