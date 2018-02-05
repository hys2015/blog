---
title: GitHub Pages + Hexo 搭建博客记录
date: 2016-03-09 16:12:01
tags: 
- Hexo
- GitHub Pages
category: 
- 日常
---

曾经想用阿里云ecs自己搭一个博客，平常可以写点东西放上去，结果过于麻烦并且不必要。Github Pages + Hexo是在windows上很友好的一个组合。
<!--more-->
好多教程里用的是 jekyll 来生成静态网页，但是在windows上使用并不是官方推荐的，所以我使用了hexo作为生成工具。

# 需要安装的软件
- [node.js][4]
- [git][6]
- [hexo][3]

# 关于Git及GitHub
## Git & GitHub
关于Git的教程网上很多，我推荐我看过的一个
[廖雪峰的Git教程][1]
里面说的很详细，不再赘述
## GitHub Pages
GitHub是一个在线托管仓库，GitHub Pages就是把你的仓库变成一个可以访问的静态网站的功能。

相关介绍(同时也是官方教程):
[GitHub Pages][2]

通过Github Pages你可以为你的Github账户建立一个自己的主页，每一个托管在github上的项目都可以作为一个二级目录来展示你网站的相应的部分或者实现相应的功能。

由于只支持静态网页的访问（毕竟免费），所以如果想要搭建博客，那么就需要一套自动生成静态网页的工具，还要整合一些网络上的服务，以便于让自己的博客功能更加丰富。

## 开启Github Pages
网上各种教程，比如这个:
[Hexo3.3.1搭建博客指南][5]


# 关于Hexo
官网链接：
[Hexo - 快速、简洁且高效的博客框架][3]
需要注意的是安装之前需要先安装node.js，下载请到这里:
[nodejs.org][4]
hexo安装完成之后，参考hexo官网的文档，或者上面那篇[博客][5]内的说明，就可以使用hexo进行博客的更新的管理了。
如果需要美化的话，hexo中提供的有主题，也可以自己根据需要制作。

# 遇到的问题
## git filename/filepath too long
因为想要保存hexo源文件到github，便于换电脑时候也能写博客（参考[这里][7]），使用`git add .` 命令时提示node_modules中的文件有一部分因为名字太长而无法访问，找到一个说法是可以在git内设置变量如下
```git
git config --global core.longpaths true
```
可以让git接收过长的路径名，结果还是不行
google了一下发现是windows api的锅，所以只好把 node_modules文件夹放入 .gitignore文件，解决
## git permission denied
因为 git bash（安装git之后的终端窗口）连接github仓库时需要rsa密钥，我之前用的是github的bash，所以有密钥，就不要生成新的秘钥，再添加到github的白名单里了。
添加秘钥的方法参考:
[Generate a SSH key][8]
[Adding your SSH key to the ssh-agent][9]
## 自动替换crlf为lf
lf是linux系统中使用的换行符，crlf是windows中使用的换行符。
因为windows和linux在换行符上的差异。因为跨平台的特性，git默认会检查换行符，并给出警告。
一下引用自git官方文档:

> `core.autocrlf`
> 假如你正在Windows上写程序，又或者你正在和其他人合作，他们在Windows上编程，而你却在其他系统上，在这些情况下，你可能会遇到行尾结束符问题。这是因为Windows使用回车和换行两个字符来结束一行，而Mac和Linux只使用换行一个字符。虽然这是小问题，但它会极大地扰乱跨平台协作。
> 
> Git可以在你提交时自动地把行结束符CRLF转换成LF，而在签出代码时把LF转换成CRLF。用core.autocrlf来打开此项功能，如果是在Windows系统上，把它设置成true，这样当签出代码时，LF会被转换成CRLF：
> 
> `$git config --global core.autocrlf true` 
> Linux或Mac系统使用LF作为行结束符，因此你不想
> Git
> 在签出文件时进行自动的转换；当一个以CRLF为行结束符的文件不小心被引入时你肯定想进行修正，把core.autocrlf设置成input来告诉
> Git 在提交时把CRLF转换成LF，签出时不转换：
> 
> `$ git config --global core.autocrlf input`

> 这样会在Windows系统上的签出文件中保留CRLF，会在Mac和Linux系统上，包括仓库中保留LF。
> 
> 如果你是Windows程序员，且正在开发仅运行在Windows上的项目，可以设置false取消此功能，把回车符记录在库中：
> 
> `$ git config --global core.autocrlf false`

## 个性化博客
我使用的是[NexT-Muse][10]主题，这个主题提供的插件什么的都挺齐全。
### 为博客添加[多说][11]的评论功能
配置相当简单，在主题 themes/next/_config.yml 文件中找到如下内容
```
duoshuo_info:
  ua_enable: true
  admin_enable: true
  user_id: 
  #admin_nickname: ROOT

```
user_id 改为自己多说的id
ua_enable 是开启获取用户平台功能，开启之后会显示用户所用的操作系统和浏览器版本
admin_enable 是开启标记博主功能

#### 获取user_id的方法
找到自己的留言，在用户名上右键->检查（资源审查），然后如图
![获取user_id](http://7xrsid.com1.z0.glb.clouddn.com/bloguser_id.png "获取user_id")
将拿到的user_id放到，上面那段代码的user_id: 后面，注意**冒号之后**一定要有**空格**
然后评论后面就会有相应的标记了

#### 多说后台添加自定义css
一张图说明一切
![添加自定义css](http://7xrsid.com1.z0.glb.clouddn.com/blogcustom_css.png "添加自定义css")

### 添加统计功能
使用[不蒜子][12]，功能介绍详细，使用方便

### --2016-3-12更新--
### 添加 RSS 订阅
使用hexo的插件 hexo-generator-feed
项目地址: [hexo-generator-feed][16]
安装方法
```
npm install hexo-generator-feed --save
```
然后在 hexo/_config.yml 中添加
```
plugin:
- hexo-generator-feed
type: atom
path: atom.xml
limit: 20
```
注意冒号后的空格

然后，再使用 `hexo g` 命令，就能生成rss文件了

在NexT主题的 _config.xml 中，在rss后面添加 /atom.xml，就会在头像下方出现RSS图标，点击就是刚刚生成的xml文件 

### 添加 自动生成 sitemap(网站地图)

使用 hexo 的插件 hexo-generator-sitemap
项目地址：Github:[hexo-generator-sitemap][15]
安装方法
```
npm install hexo-generator-sitemap --save
```
在 hexo/_config.xml 中添加一行
```
sitemap: sitemap.xml
```
然后再执行 `hexo g` 之后，访问 yoururl/sitemap.xml 就能看到生成的sitemap文件了。
然后把sitemap文件url添加到[百度站长][13]或者[Google Analytics][14]上，就能够被搜索引擎收录了。

## 消除hexo转义符对mathjax公式的影响
参考问题中修改marked.js的回答，目前还没有发现什么问题。
https://segmentfault.com/q/1010000003987383/a-1020000003987577

## github和coding.net(原gitcafe)同时部署
github的pages服务用百度没办法检索到，只好双备份，github和gitcafe都有。
在coding.net上使用pages服务跟github上类似。
**需要绑定域名时，一定要先在运营商那边添加CNAME条目再在coding.net上绑定相应的域名，否则会不起作用（绑定的时候coding.net也会提醒）**
想要同时部署，在hexo的blog配置文件中deploy部分改成如下片段就好：
```
deploy: 
- type: git
  repo: git@github.com:hys2015/blog.git
  branch: gh-pages
- type: git
  repo: git@git.coding.net:markheng/blog.git
  branch: coding-pages
```
**需要注意branch要对应到pages服务需要的分支才有用。**


  [1]: http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000 "廖雪峰的Git教程"
  [2]: https://pages.github.com/ "GitHub Pages 官方网站"
  [3]: https://hexo.io/zh-cn/%20hexo-%E5%BF%AB%E9%80%9F%E3%80%81%E7%AE%80%E6%B4%81%E4%B8%94%E9%AB%98%E6%95%88%E7%9A%84%E5%8D%9A%E5%AE%A2%E6%A1%86%E6%9E%B6
  [4]: https://nodejs.org/en/ "node.js"
  [5]: http://lovenight.github.io/2015/11/10/Hexo-3-1-1-%E9%9D%99%E6%80%81%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E6%8C%87%E5%8D%97/ "loveNight hexo3.3.1搭建博客指南"
  [6]: https://git-scm.com/downloads "Git Download Page"
  [7]: http://zhihu.com/question/21193762/answer/79109280?utm_campaign=webshare&amp;utm_source=weibo&amp;utm_medium=zhihu "CrazyMilk的回答"
  [8]: https://help.github.com/articles/generating-an-ssh-key/ "Generating an SSH key"
  [9]: https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#adding-your-ssh-key-to-the-ssh-agent "Add your SSH key to your ssh-agent"
  [10]: http://theme-next.iissnan.com/ "NexT文档"
  [11]: http://duoshuo.com/ "多说"
  [12]: http://ibruce.info/2015/04/04/busuanzi/ "不蒜子简易计数"
  [13]: http://zhanzhang.baidu.com "百度站长"
  [14]: https://analytics.google.com/analytics/web/ "Google Analytics"
  [15]: https://github.com/hexojs/hexo-generator-sitemap "hexo-sitemap"
  [16]: https://github.com/hexojs/hexo-generator-feed "hexo-feed"