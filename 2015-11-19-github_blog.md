---
title: "初步学习github page 和Jekyll来构建博客"
date: 2015-11-19 00:00:00
category: 笔记
tags: [github,Jekyll]
---

# 首先最基本的博客搭建
## 出现的问题
> 1.文本一定要以UTF-8 无BOM格式存储啊！！！不然显示不出

## 引言：一些问题的解答：
1. 想找服务器挂载一个自己的个人主页或博客，去哪找呢？
	一般像自己当掌握一门技术，当希望有一个属于自己空间，然后把自己的技术心得进行阐述的时候，就会想起记工作笔记，之前自己尝试过用Latex模板来记录学习情况，但是渐渐的。。。。发现Latex在消耗自己的时间成本上代价太大了，然后转转悠悠，偶然的机会我接触到了Mardown语法，突然间自己对此简直爱不释手，语法非常简单，虽然在版面的优化上比不上Latex模板的强大，但是语法之简单，而且可以生成Html的格式。此时我又发现一个问题，Markdown生成的Html页面整理起来不方便。于是自己又通过马克飞象这个工具，离线的记录然后生成PDF版本的笔记，这样就能很好的做一个工作笔记了。但是简简是自己查看感觉这个文档不是在线的查看而是离线查看不太方便，于是自己开始酝酿搞一个自己的博客。一开始自己去了CSDN上写博客，总感觉自己好不爽，爽然东西都是自己的，但是感觉东西是放在人家那里的，整个目录的内容不能离线的处理同步，逐渐自己发现了github能开设自己的博客，于是一下子自己找到了*组织*了。下面三个阶段很能描述自己纠结的三个阶段。
	> 第一阶段，刚接触Blog，觉得很新鲜，试着选择一个免费空间来写。
	第二阶段，发现免费空间限制太多，就自己购买域名和空间，搭建独立博客。
	第三阶段，觉得独立博客的管理太麻烦，最好在保留控制权的前提下，让别人来管，自己只负责写文章。


2. 为什么来到github上挂载自己的博客？什么情况下github.io又不适合了？
	一开始自己还考虑过挂载到阿里云上去，但是在查看阿里云上面的价格后自己就退却了，自己还真没有这个实力去支持这个，当然了主要的还是要看自己的需求了，如果自己需要所有的网页内容都动态的更新加载，所有的操作都在云端实现，这个时候就需要自己花钱买云服务器了，应为在github上挂载的主页都是些静态的网页，如果有大量的交互操作的时候，显然github page是不适合你的。

3. 关于区分个人博客的创建和项目主页的创建
	github page主要有两种创建方式
	1. 一种是在github根目录上创建一个名字为：username.github.io的项目，并且定义成master属性。此时访问的网址则为：username.github.io 就是主页的网址。
	2. 另一个是在任意定义的projectName项目下，通过以下命令在项目的根目录上创建分支节点
    
	```bash
		~$ git checkout --orphan gh-pages
	```
	因为github规定，只有该分支中的页面，才会生成网页文件
## 第一步：在github创建一个项目
首先到GitHub上创建一个项目（ [create a new repository](https://github.com/new)）,并且命名为：username.github.io，username是你注册github账号时的用户名。
![github上创建自己的项目](http://ww2.sinaimg.cn/bmiddle/9d2c4511gw1ey6ntlgonej20mg0ku77s.jpg)
## 第二步：clone到本地
	~$ git clone https://github.com/username/username.github.io


## 第三步：编写Helloworld的主程序
	~$ cd username.github.io
	~$ echo "Hello World" > index.html


## 第四步：上传主程序，查看效果
	~$ git add --all
	~$ git commit -m "Initial commit"
	~$ git push -u origin master


# 然后是结合Jekyll搭建博客
## Jekyll是什么？
[Jekyll](http://jekyllrb.com/)（发音/'d?i?k ?l/，"杰克尔"）是一个静态站点生成器，它会根据网页源码生成静态文件。它提供了模板、变量、插件等功能，所以实际上可以用来编写整个网站。
![jekyll标志](http://ww3.sinaimg.cn/bmiddle/9d2c4511gw1ey6odz1o5pj20hj0abt8y.jpg)

整个思路到这里就很明显了。你先在本地编写符合Jekyll规范的网站源码，然后上传到github，由github生成并托管整个网站。

**这种做法的好处是：**
- 免费，无限流量。
- 享受git的版本管理功能，不用担心文章遗失。
- 你只要用自己喜欢的编辑器写文章就可以了，其他事情一概不用操心，都由github处理

**它的缺点是：**
- 有一定技术门槛，你必须要懂一点git和网页开发。
-  它生成的是静态网页，添加动态功能必须使用外部服务，比如评论功能就只能用disqus。
- 它不适合大型网站，因为没有用到数据库，每运行一次都必须遍历全部的文本文件，网站越大，生成时间越长。
## 如何快速完成一个基于Jekyll的个人博客？
### 第一步
在你的电脑上，建立一个目录，作为项目的主目录。我们假定，它的名称为jekyll_demo。

```bash
$ mkdir jekyll_demo
```
对该目录进行git初始化

```bash
$ cd jekyll_demo
$ git init
```
然后，创建一个没有父节点的分支gh-pages。因为github规定，只有该分支中的页面，才会生成网页文件。

```bash
$ git checkout --orphan gh-pages

```
### 第二步
在项目根目录下，建立一个名为_config.yml的文本文件。它是jekyll的设置文件，我们在里面填入如下内容，其他设置都可以用默认选项，具体解释参见[官方网页](https://github.com/mojombo/jekyll/wiki/Configuration)。

```bash
baseurl: /jekyll_demo
```
目录结构变成：

```
/jekyll_demo
　　　　|--　_config.yml
```
### 第三步
在项目根目录下，创建一个_layouts目录，用于存放模板文件。

```bash
$ mkdir _layouts
```
进入该目录，创建一个default.html文件，作为Blog的默认模板。并在该文件中填入以下内容。

```html
<!DOCTYPE html>
　　<html>
　　<head>
　　　　<meta http-equiv="content-type" content="text/html; charset=utf-8" />
　　　　<title>{{ page.title }}</title>
　　</head>
　　<body>

　　　　{{ content }}

　　</body>
　　</html>
```
Jekyll使用[Liquid模板语言](https://github.com/shopify/liquid/wiki/liquid-for-designers)，{{ page.title }}表示文章标题，{{ content }}表示文章内容，更多模板变量请参考[官方文档](https://github.com/mojombo/jekyll/wiki/Template-Data)。
目录结构变成：

```
/jekyll_demo
　　　　|--　_config.yml
　　　　|--　_layouts
　　　　|　　　|--　default.html
```
### 第四步
回到项目根目录，创建一个_posts目录，用于存放blog文章。

```bash
$ mkdir _posts
```
进入该目录，创建第一篇文章。文章就是普通的文本文件，文件名假定为2012-08-25-hello-world.html。(注意，文件名必须为"年-月-日-文章标题.后缀名"的格式。如果网页代码采用html格式，后缀名为html；如果采用markdown格式，后缀名为md。）
在该文件中，填入以下内容：（注意，行首不能有空格）

```html
---
layout: default
title: 你好，世界
---
<h2>{{ page.title }}</h2>
<p>我的第一篇文章</p>
<p>{{ page.date | date_to_string }}</p>
```
每篇文章的头部，必须有一个yaml文件头，用来设置一些元数据。它用三根短划线"---"，标记开始和结束，里面每一行设置一种元数据。"layout:default"，表示该文章的模板使用_layouts目录下的default.html文件；"title: 你好，世界"，表示该文章的标题是"你好，世界"，如果不设置这个值，默认使用嵌入文件名的标题，即"hello world"。
在yaml文件头后面，就是文章的正式内容，里面可以使用模板变量。{{ page.title }}就是文件头中设置的"你好，世界"，{{ page.date }}则是嵌入文件名的日期（也可以在文件头重新定义date变量），"| date_to_string"表示将page.date变量转化成人类可读的格式。
目录结构变成：

```
/jekyll_demo
　　　　|--　_config.yml
　　　　|--　_layouts
　　　　|　　　|--　default.html 
　　　　|--　_posts
　　　　|　　　|--　2012-08-25-hello-world.html
```
### 第五步:创建首页
有了文章以后，还需要有一个首页。
回到根目录，创建一个index.html文件，填入以下内容。

```html
---
layout: default
title: 我的Blog
---
　　<h2>{{ page.title }}</h2>
　　<p>最新文章</p>
　　<ul>
　　　　{% for post in site.posts %}
　　　　　　<li>{{ post.date | date_to_string }} <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></li>
　　　　{% endfor %}
　　</ul>
```
它的Yaml文件头表示，首页使用default模板，标题为"我的Blog"。然后，首页使用了 {$% for post in site.posts %$} '，表示对所有帖子进行一个遍历。这里要注意的是，Liquid模板语言规定，输出内容使用两层大括号，单纯的命令使用一层大括号。至于{{site.baseurl}}就是_config.yml中设置的baseurl变量。
目录结构变成：

```
/jekyll_demo
　　　　|--　_config.yml
　　　　|--　_layouts
　　　　|　　　|--　default.html 
　　　　|--　_posts
　　　　|　　　|--　2012-08-25-hello-world.html
　　　　|--　index.html
```
### 第六步，发布内容。
现在，这个简单的Blog就可以发布了。先把所有内容加入本地git库。

```bash
$ git add .
$ git commit -m "first post"
```
然后，前往github的网站，在网站上创建一个名为jekyll_demo的库。接着，再将本地内容推送到github上你刚创建的库。注意，下面命令中的username，要替换成你的username。

```bash
$ git remote add origin https://github.com/username/jekyll_demo.git
$ git push origin gh-pages
```
上传成功之后，等10分钟左右，访问http://username.github.com/jekyll_demo/就可以看到Blog已经生成了（将username换成你的用户名）。


**参考文章**:
1. [阮一峰的搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)