---
title: webpack基础用法
date: 2020-04-20 23:28:37
tags: 
 - webpack
categories: 
 - webpack
---

## 核心概念

### entry

单入口：

```js
module.exports = {	
    entry: './src/index.js'
}
```

多入口

```js
module.exports = {	
    entry: {    	
        app: './src/app.js',        
        adminApp： './src/adminApp.js'    
    }	
}
```

### output

webpack编译后的文件输出到磁盘

```js
const path = require('path')
module.exports = {    
	entry: {    	
		app: './src/app.js',        
		adminApp： './src/adminApp.js'    },    
	output: {        
		path: path.join(__dirname,'dist'),        
		filename: '[name].js'    //[name]占位符   
    }
}
```

webpack初始只支持js和json两种文件类型，通过loader去支持其他文件类型并把它们转化成有效的模块，并且添加到依赖图中。

常见的loader有：

| 名称                    | 描述                           |
| ----------------------- | ------------------------------ |
| babel-loader            | 转换es6、7等js新特性语法       |
| css-loader              | 支持.css后缀文件的加载和解析   |
| less-loader/sass-loader | 将.less/.scss后缀文件转化成css |
| ts-loader               | 将ts转换成js                   |
| file-loader/url-loader  | 进行图片、字体等的打包         |
| raw-loader              | 将文件以字符串的形式导入       |
| thread-loader           | 多进程打包js和css              |

```js
module.exports = {    
	module:{        
		rules:[            
			{                
				test: /\.txt$/, //test指定匹配规则                
				use: 'raw-loader' //use指定使用的loader名称           
            }        
        ]    
    }
}
```

### plugin

常见的plugin:

| 名称                     | 描述                                       |
| ------------------------ | ------------------------------------------ |
| CommonsChunkPlugin       | 将chunks相同的模块代码提取成公共js         |
| CleanWebpackPlugin       | 清理构建目录                               |
| ExtractTextWebpackPlugin | 将css从bundle文件里提取成一个独立的css文件 |
| CopyWebpackPlugin        | 将文件或文件夹拷贝到构建的输出目录         |
| HtmlWebpackPlugin        | 内存中创建html文件去承载输出的bundle       |
| zipWebpackPlugin         | 将打包厨的资源生成一个zip包                |

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')

//插件全都放入plugins数组里
module.exports = {	
	plugins: [        
		new HtmlWebpackPlugin(            
			{                
				template: './src/index.html'            
			}        
		)    
	]
}
```

### mode

用来指定当前的构建环境： production、development、none

| 选项        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| development | 设置process.env.NODE_ENV的值为development，开启NamedChunksPlugin和NamedModulesPlugin |
| production  | 设置process.env.NODE_ENV的值为production，开启FlagDependencyUsagePlugin ，FlagIncludedChunksPlugin ,ModuleConcatenationPlugin , NoemitOnErrorsPlugin, OccurrenceOrderPlugin , SideEffectFlagPlugin, TerserPlugin |
| none        | 不开启任何优化选项                                           |

```js
module.exports = {	
	mode: development // production/none
}
```

## 解析ES6和jsx

### 解析es6

命令：

```
yarn add @babel/core @babel/preset-env babel-loader -D
```

在根目录创建.babelrc文件

```
{	
	"presets": [		
		"@babel/preset-env"	
	]
}
```

webpack.config.js

```js
module.exports = {	
	module: {        
		rules: [            
			{                
                test: /\.js$/,                
                use: 'babel-loader'            
            }        
		]    
	}
}
```

### 解析jsx

```
yarn add react react-dom @babel/preset-react -D
```

.babelrc增加@babel/preset-react

```
{
	"presets": [
		"@babel/preset-env",
		"@babel/preset-react"
	]
}
```

webpack.config.js

```js
module.exports = {
 module: {
    rules: [
      {
        test: /\.(js|jsx)$/, //增加匹配后缀名jsx即可使用.jsx文件
        use: "babel-loader"
      }
    ]
  }
}
```



## 解析css、less/scss

### 解析css

```
`yarn add style-loader css-loader -D`
```

顺序必须是先写style-loader，再写css-loader。代码是从右向左执行，先执行css-loader用来解析.css文件，然后再将解析好的css传递给style-loader。

### 解析less/sass，以less为例

```
yarn add less less-loader -D
```

先将.less文件转化为.css文件，之后同上

webpack.config.js

```js
module.exports = {
	module: {
		rules: [
			//解析css
            {
                test: /\.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            }, 
            {
                test: /\.less$/,
                use: [
                    'style-loader',
                    'css-loader'，
                    'less-loader'
                ]
            }
        ]
	}
}

```

## 解析图片

使用file-loader或url-loader，本文以url-loader，可配置limit，若文件小于限制大小则返回base64格式

```
yarn add url-loader -D
```

webpack.config.js

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 10240 //小于10K则使用base64
            }
          }
        ]
      }
    ]
  }
}
```

## webpack中文件的监听

文件监听是在发现源码发生变化时，自动重新构建出新的输出文件。

webpack开启监听模式有两种

1. 启动webpack命令时，带上–watch
2. 在配置webpack.config.js中设置watch:true

缺陷是需要手动刷新浏览器

package.json

```json
{
    "scripts": {
        "build": "webpack",
        "watch": "webpack --watch"
    },
}
```

### 监听原理分析

 轮询判断文件的最后编辑事件是否变化，某个文件发生了变化，并不会立刻告诉监听者，而是先缓存起来，等aggregateTimeout 

```
module.export = {
	watch: true,
    //只有开启监听模式时，watchOptions才有意义
	watchOptions: {
		ignored: /node_modules/, //不监听的文件或文件夹
		aggregateTimeout: 300, //监听到变化发生后等300ms再去执行，默认300ms
        //判断文件是否发生变化是通过不停询问系统指定文件有没有变化实现的，默认每秒访问1000次
        poll: 1000 
	}
}
```



## 热更新 webpack-dev-server

wds不需要刷新浏览器，也不输出文件，而是放在内存中

```
yarn add webpack-dev-server -D
```

### 方法一，需配合内置的HotModuleReplacementPlugin插件

packgage.json

```json
{
	"scripts": {
		"start": "webpack-dev-server"
	}
}
```

webpack.config.js

```js
const {HotModuleReplacementPlugin} = require('webpack')

module.exports = {
	plugins: [
        new HotModuleReplacementPlugin()
    ],
    devServer: {
        contentBase: './dist', //项目根路径
        port: 8080, //端口，默认8080
        hot: true //热重载
        open: true //自动打开
    }
}
```

### 方法二

直接修改package.json ，无需使用HotModuleReplacementPlugin

```json
{
	"scripts": {
		"start": "webpack-dev-server --open --port 8080 --contentBase ./dist --hot"
	}
}
```

### 热更新原理

Webpack Compile : 将js源代码编译成Bundle

HMR Server: 将热更新的文件输出给 HMR Runtime

Bundle server: 提供文件在浏览器的访问

HMR Runtime: 会被注入到浏览器，更新文件的变化

bundle.js: 构建输出的文件

{%asset_img p1.png%}



## 文件指纹

即打包输出的文件名的后缀

Hash：和整个项目的构建相关，只要有项目文件修改，整个项目构建的hash值就会更改

Chunkhash：和webpack打包的chunk有关，不同的entry会生成不同的chunkhash

Contenthash：根据文件内容来定义hash，文件内容不变则contenthash不变

### js的文件指纹设置

设置output的filename，使用[chunkhash]

```js
module.exports = {
	output: {
		path: './dist',
		filename: '[name]_[chunkhash:8].js'
	}
}
```

### css文件指纹设置

使用style-loader是将css代码打包进js文件，所以我们使用一个插件mini-css-extract-plugin，可以将css提取成一个独立的文件打包到dist目录，无法和style-loader一起使用，替换style-loader为MiniCssExtractPlugin.loader

```
yarn add mini-css-extract-plugin -D
```

webpack.config.js

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [MiniCssExtractPlugin.loader, "css-loader"],
            },
            {
                test: /\.less$/,
                use: [MiniCssExtractPlugin.loader, "css-loader", "less-loader"],
            }
        ]
	},
    plugins: [
        new MiniCssExtractPlugin({
          	filename: "[name]_[contenthash:8].css",
        }),
  	]
}
```

### 图片的文件指纹设置

设置file-loader(url-loader)的name,使用[hash]

```js
module.exports = {
	module: {
		rules: [
			test: /\.(png|jpg|svg)$/,
			use: [{
				loader: 'file-loader',
				options: {
					name: '[name]_[hash:8].[ext]'
				}
			}]
		]
	}
}
```

| 占位符名称    | 含义                          |
| ------------- | ----------------------------- |
| [ext]         | 后缀名                        |
| [name]        | 文件名                        |
| [path]        | 相对路径                      |
| [folder]      | 文件所在文件夹                |
| [contenthash] | 文件的内容hash，默认是MD5     |
| [hash]        | 文件的内容hash，默认是MD5     |
| [emoji]       | 一个随机的指代文件内容的emoji |



## 代码压缩

webpack4内置js压缩，所以无需安装js压缩插件

### css压缩

使用optimize-css-assets-webpack-plugin，依赖cssnano

```
yarn add optimize-css-assets-webpack-plugin cssnano -D
```

webpack.prod.js

```js
const OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin')

module.exports = {	
    plugins: [		
        new OptimizeCssAssetsPlugin({			
            assetNameRegExp: /\.css$/g,			
            cssProcessor: require('cssnano')  //依赖cssnano		
        })	
    ]
}
```

### html压缩

```
yarn add html-webpack-plugin -D
```

webpack.prod.js

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
	plugins: [
		new HtmlWebpackPlugin({
            template: path.join(__dirname,'src/index.html'), //模板路径
            filename: 'index.html', //文件名称
            //chunks: ['main'], //包含哪些chunk,
            inject: true//chunk自动注入
        })
	]
}
```



## 清理output的dist目录

默认会删除output的输出目录

```
yarn add clean-webpack-plugin -D
```

webpack.prod.js

```js
const {CleanWebpackPlugin} = require('clean-webpack-plugin')
module.exports = {
	plugins: [
		new CleanWebpackPlugin()
	]
}
```

## 自动补齐css3前缀

使用autoprefixer插件，后置处理器

```
yarn add postcss-loader autoprefixer -D
```

package.json增加兼容浏览器

```json
{	
    "browserslist": [       
        "defaults",        
        "not ie < 11",        
        "last 2 versions",        
        "> 1%",        
        "iOS 7",        
        "last 3 iOS versions"    
    ]
}
```

webpack.config.js增加postcss-loader，开发环境生产环境都需增加。

```js
module.exports = {
	module: {
		rules: [{
        	test: /\.less$/,
        	use: [
          		MiniCssExtractPlugin.loader,
          		"css-loader",
          		"less-loader",
          		"postcss-loader", //postcss-loader
        	]
      	}]
	}
}
```

 根目录新建文件postcss.config.js 

```js
module.exports = {  
    plugins: [      
        require('autoprefixer')()  
    ]
}
```



## px自动转rem

使用px2rem-loader

页面渲染时计算根元素的font-size值，可使用手机淘宝的lib-flexible



```
yarn add px2rem-loader -D
```

```
yarn add lib-flexible
```

webpack.config.js

```js
module.exports = {
    module: {
        rules: [
            test: /\.less$/,
        	use: [
          		MiniCssExtractPlugin.loader,
          		"css-loader",
          		"less-loader",
          		"postcss-loader",
            	{
            		loader: 'px2rem-loader',
           			options: {
            			remUnit: 75,
            			remPrecession: 8 //px转rem小数点位数
            		}
            	}
        	]
        ]
    }
}
```

在main.js导入 lib-flexible

