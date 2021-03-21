---
title: 使用Gitee+Hexo搭建个人博客
date: 2020-04-04 19:30:00
categories: 
    - 搭建
tags: 
    - 博客搭建

---

 [Gitee](https://gitee.com/): 由于github在国外，被墙了，于是把目光投向了国产Github，Gitee，算是支持国产吧。
 Hexo：一个快速、简洁高效的博客框架，使用Markdown解析文本。 

## 搭建

### 环境要求

- Git
- NodeJs
  默认你已经安装了Git 和 NodeJS (本文使用yarn进行包管理)

### 安装hexo

``` bash
yarn add global hexo
```

### 初始化hexo

 初始化后，文件夹的目录如下： 

``` bash
.
├── .deploy       #需要部署的文件
├── node_modules  #Hexo插件
├── public        #生成的静态网页文件
├── scaffolds     #模板
├── source        #博客正文和其他源文件等都应该放在这里
|   ├── _drafts   #草稿
|   └── _posts    #文章
├── themes        #主题
├── _config.yml   #全局配置文件
└── package.json

```

### 获取博客主题

本文使用的是hexo-theme-diaspora，如需其他主题请自行百度或 https://hexo.io/themes/ 

#### 安装主题

``` bash
git clone https://github.com/Fechin/hexo-theme-diaspora.git themes/diaspora
```

#### 启用主题

 修改Hexo配置文件 `_config.yml` 主题项设置为diaspora 

``` bash
...
theme: diaspora
...
```

#### 更新主题

 注意：请在更时主题时备份`_config.yml`配置文件 

```
cd themes/diaspora
git pull
```

#### 新建文章模板

```
---
title: My awesome title
date: 2016-10-12 18:38:45
categories: 
    - 分类1
    - 分类2
tags: 
    - 标签1
    - 标签2
mp3: http://domain.com/awesome.mp3
cover: http://domain.com/awesome.jpg
---
```

#### 创建分类页

 1 新建一个页面，命名为 categories 。命令如下： 

```
hexo new page categories
```

 2 编辑刚新建的页面，将页面的类型设置为 categories 

```
title: categories
date: 2014-12-22 12:39:04
type: "categories"
---
```

 主题将自动为这个页面显示所有分类。 

#### 创建标签页

1 新建一个页面，命名为 tags 。命令如下：

```
hexo new page tags

```

2 编辑刚新建的页面，将页面的类型设置为 tags

```
title: tags
date: 2014-12-22 12:39:04
type: "tags"
---

```

主题将自动为这个页面显示所有标签。

#### 创建搜索页

1 需要安装hexo的搜索插件

```
npm install hexo-generator-searchdb --save

```

2 配置hexo全局配置文件（请将生成的索引文件放在网站根目录或修改主题js文件的path值）

```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000

```

3 新建一个页面，命名为 search 。命令如下：

```
hexo new page search

```

4 编辑刚新建的页面，将页面的类型设置为 search

```
title: search
date: 2014-12-22 12:39:04
type: "search"
---

```

5 在主题配置文件启用本地搜索

```
#本地搜索,请将索引文件放在网站根目录
local_search:
    #是否启用
    enable: true

```

主题将自动为这个页面显示搜索功能。

#### 主题配置

```
# 头部菜单，title: link
menu:
  首页: /
  分类: /categories
  标签: /tags
  归档: /archives  
  关于: /about

# 是否显示目录
TOC: false

# 是否自动播放音乐
autoplay: false

# 默认音乐（随机播放）
mp3: 
    - http://link.hhtjim.com/163/425570952.mp3
    - http://link.hhtjim.com/163/425570952.mp3

# 首页封面图, 为空时取文章的cover作为封面(注意跨域问题,建议使用同源图片)
welcome_cover: /img/welcome-cover.jpg


# 默认文章封面图（随机调用,支持外链）
cover: 
  - /img/cover.jpg
  - /img/welcome-cover.jpg

 
# 是否关闭默认滚动条
scrollbar: true

# 本地搜索,请将索引文件放在网站根目录,或修改主题js文件的path值
local_search:
    # 是否启用
    enable: false

# 是否显示 一言(hitokoto)
hitokoto: true

# 链接(可选:facebook,twitter,github,wechat,email)
links:
    facebook: /
    twitter: /
    github: /
    wechat: /img/logo.png
    email: mailto:xxxx@gmail.com
  
# 备案
beian: 
    # 是否显示备案信息
    enable: true
    # 是否在主页面最底下显示备案信息(虽然丑，但是完全满足规定要求)
    enableFooter: false
    # 备案号
    beianInfo: 冀ICP备xxxxxxx号
    # 链接地址
    link: http://www.beian.miit.gov.cn

# 是否使用mathjax
mathjax: false

# Gitalk 评论插件（https://github.com/gitalk/gitalk）
gitalk:
    # 是否启用评论功能
    enable: false
    # 是否自动展开评论框
    autoExpand: false
    # 应用编号
    clientID: ''
    # 应用秘钥
    clientSecret: ''
    # issue仓库名
    repo: ''
    # Github名
    owner: ''
    # Github名
    admin: ['']
    # Ensure uniqueness and length less than 50
    id: location.pathname
    # Facebook-like distraction free mode
    distractionFreeMode: false

# 网站关键字
keywords: Fechin

# 要使用google_analytics进行统计的话，这里需要配置ID
google_analytics: 

# 网站ico
favicon: /img/favicon.png

# rss文件
rss: atom.xml

```

### 本地预览

编译项目，输入命令：`hexo g`
运行项目，输入命令：`hexo s`   在浏览器中输入`http://localhost:4000/``就可以看到效果啦 

### 部署博客到Gitee上

#### 新建仓库

{%asset_img p1.png%}

一定要选择javascript语言，否则样式不生效

#### 在_config.yml中配置Git

 注意：冒号后面一定要有空格，否则不能正确识别。 

```
deploy:  
  type: git  
  repo: https://gitee.com/Dicky_Lin/blog.git
  branch: master

```

#### 发布到Gitee

输入命令`yarn add hexo-deployer-git 安装自动部署发布工具
输入命令`hexo clean && hexo g && hexo d` 发布博客，首次发布需要在shell中输入账号和密码。 

#### Gitee  Pages设置

{%asset_img p2.png%}

稍等一会儿博客就发布成功啦，访问博客地址： http://dicky_lin.gitee.io/blog ，就可预览在线博客啦！！！
如果博客的样式不对，则需要在_config.yml中配置下博客地址和路径： 

```
url: https://xiuxiuing.gitee.io/blog/
root: /blog

```

再执行命令`hexo clean && hexo g && hexo d` 就可以啦。 

至此，我们的博客就搭建完成啦！！！
在`/Hexo/source/_posts`目录下就可以写我们的博客啦！！！ 

个人博客效果参考：http://dicky_lin.gitee.io/blog 

