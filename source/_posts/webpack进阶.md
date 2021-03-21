---

title: webpack进阶
date: 2020-04-21 12:33:07
tags: 
 - webpack
categories: 
 - webpack
---

## meta标签及title标签自动注入html

创建meta.json，用来存放需导入的meta标签

```json
{
  "meta1": {
    "data-vue-meta": true,
    "name": "renderer",
    "content": "webkit"
  },
  "meta2": {
    "data-vue-meta": true,
    "http-equiv": "X-UA-Compatible",
    "content": "IE=edge"
  }
}
```

配合HtmlWebpackPlugin使用

```js
module.exports = {
	plugins: [
		new HtmlWebpackPlugin({
          	template: path.join(__dirname, "src/index.html"), //模板路径
          	filename: "index.html", //文件名称
          	// chunks: ["main"], //使用哪些chunk
          	inject: true, //chunk自动注入
          	meta: require('./src/meta.json'), //注入meta
          	title: 'test-webpack', //注入的标签名字
        }),
	]
}
```

index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <!-- title ，ejs语法-->
  <title><%= htmlWebpackPlugin.options.title %></title> 
</head>
<body>
  <div id="root"></div>
</body>
</html>
```



## 多页面应用打包通用方案

动态获取entry和设置html-webpack-plugin数量

利用glob.sync，约定好所有的页面入口都是对应文件夹下的index.js，模板文件都叫index.html

```
yarn add glob -D
```

核心代码：

```
glob.sync(path.join(__dirname,'./src/views/*/index.js'))
```

接着上全部代码

webpack.prod.js

```js
const glob = require('glob')
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

//设置多页面
const setMPA = () => {
    const entry = {}
    const HtmlWebpackPlugins = []
    const entryFiles = glob.sync(path.join(__dirname,'./src/views/*/index.js')) 
    
    console.log('entryFiles~~~~~~~~~~~~~~~~~~~~~',entryFiles) //为一个数组
    
    entryFiles.forEach((item) => {
    const match = item.match(/\/src\/views\/(.*)\/index\.js/);
    const pageName = match && match[1]; //匹配的页面名称
    entry[pageName] = item;
     
    HtmlWebpackPlugins.push(
        new HtmlWebpackPlugin({
            template: path.join(__dirname, `src/views/${pageName}/index.html`), //模板路径
            filename: `${pageName}.html`, //文件名称
            chunks: [pageName], //使用哪些chunk
            inject: true, //chunk自动注入
        })
    );
    });
    
    return {
        entry,
        htmlWebpackPlugins
    }
}

const {entry , HtmlWebpackPlugins} = setMPA()

module.exports = {
    entry,
    output: {
        path: path.resolve(__dirname, "./dist"),
        filename: "js/[name]_[chunkhash:8].js",
    },
    plugins: [
        ...HtmlWebpackPlugins
    ]
}
```



## source map

作用：通过source map 定位到源代码

开发环境开启，线上环境关闭，线上排查问题的时候可将source map 上传到错误监控系统

### 关键字

eval：使用eval包裹模块代码

source map：产生.map文件

cheap：不包含列信息

inline：将.map作为DataURI嵌入，不单独生成.map文件

module：包含loader的sourcemap

### 类型

{%asset_img p1.png%}

webpack.config.js

```js
module.exports = {
    devtool: 'source-map'
}
```





## 利用SplitChunksPlugin进行公共脚本分离

webpack4内容插件，替代webpack3的CommonsChunkPlugin



沿用之前配置的多页面代码

{%asset_img p2.png%}

```js
// index/index.js
import React from 'react'
import ReactDOM from 'react-dom'

import 'lib-flexible'
import Demo from './demo.jsx'
import common from '../common.js'

const App = ()=>{
  return (
    <div>
      Appaaaa
      {common()}
      <Demo />
    </div>
  )
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
)
```

```js
// search/index.js
import React from 'react'
import ReactDOM from 'react-dom'

import common from '../common.js'

const App = ()=>{
  return (
    <div>
      searchPage
      {common()}
    </div>
  )
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
)
```

```js
// common.js
export default () => {
  return '我是你爸爸我真伟大'
}
```

### 配置optimization

将src目录下的使用到的公共模块抽离到commons

将node_modules下使用到的模块抽离到venders

```js
//webpack.prod.js
module.exports = {
    optimization: {
        splitChunks: {
            chunks: "all",
            cacheGroups: {
                commons: { //抽离公共模块
                    chunks: 'initial',
                    minSize: 0, //大于0K即可抽离
                    minChunks: 2 //被使用次数>=2次
                },
                vendors: { //抽离node_modules下的第三方模块
                    test: /[\\/]node_modules[\\/]/,
                    chunks: 'initial',
                    minSize: 0,
                    minChunks: 2,
                    priority: 10 //优先级
                }
            },
        },
    },
}
```



## js懒加载及预加载

动态导入js        import(/* webpackChunkName: 'lazyLoad' */'./lazyload.js')

```jsx
//demo.jsx
import React, { useState } from "react";
export default () => {
  const [ lazyLoad, setLazyLoad ] = useState(null);

  const handleLazyLoad = async () => {
    //webpackChunkName js的名字
    //  预加载：/* webpackChunkName: 'lazyLoad', webpackPrefetch: true */
    const res = await import(/* webpackChunkName: 'lazyLoad' */'./lazyload.js')
    setLazyLoad(res.default)
  };

  return (
    <div className="demo">
      <button onClick={handleLazyLoad}>lazyLoad</button>
      {lazyLoad}
    </div>
  );
};
```

```js
//lazyload.js
export default '<div>懒加载完成</div>'
```

懒加载返回的是一个promise对象，这里我用的是async await语法，浏览器不支持，所以还需安装

```
yarn add @babel/runtime && yarn add @babel/plugin-transform-runtime -D
```



懒加载是动态加载，事件回调后加载

正常加载可认为是并行加载（同时加载多个文件）

预加载是等待其他所有资源加载完毕后再偷偷加载（兼容性较差）





## Dll

使用 dll对某些库（第三方: react、vue、jq...）进行单独打包

需使用webpack内置插件DllPlugin

新建一个webpack.dll.js

```js
const {resolve} = require('path')
const {DllPlugin} = require('webpack')
module.exports = {
	entry: {
        //最终打包生成的name--->react: ['react']--->要打包的库是react 
        react: ['react'], 
        //最终打包生成的name--->reactDOM: ['react']--->要打包的库是react-dom
        reactDOM: ["react-dom"], 
    },
    output: {
        filename: '[name].js',
        path: resolve(__dirname,'dll'),
        library: '[name]_[hash:8]' //打包的库向外暴露出去的内容名
    },
    plugins: [
        //打包生成一个manifest.json  -->提供和react的映射关系
        new Dllplugin({
            name: '[name]_[hash:8]', //映射库暴露的内容名称
            path: resolve(__dirname, 'dll/manifest.json') //输出文件路径
        })
    ],
    mode: 'production'
}
```

package.json增加scripts配置

```json
"dll": "webpack --config webpack.dll.js"
```

执行yarn dll命令后根目录会生成以下文件

{%asset_img p1.png%}



之后在webpack.prod.js引入webpack内置插件DllReferencePlugin

还需安装add-set-html-webpack-plugin

```
yarn add add-asset-html-webpack-plugin -D
```



```js
//webpack.prod.js
const {resolve} = require('path')
const {DllReferencePlugin} = require('webpack')
const AddAssetHtmlWebpackPlugin = require('add add-asset-html-webpack-plugin')

module.exports = {
	plugins: [
        //告诉webpack哪些库是不参与打包
        new DllReferencePlugin({
            manifest: resolve(__dirname,'dll/manifest.json')
        }),
		//将某个文件打包输出出去，并在html中自动引入该资源，多个可动态添加，这里就不细写了
        new AddAssetHtmlWebpackPlugin({
            filepath: resolve(__dirname, 'dll/react.js')
        }),
        new AddAssetHtmlWebpackPlugin({
            filepath: resolve(__dirname, 'dll/react-dom.js')
        })
    ]
}
```



## 注意：

由于本人学识尚浅，暂无不知splitChunks 和 Dll如何共存。