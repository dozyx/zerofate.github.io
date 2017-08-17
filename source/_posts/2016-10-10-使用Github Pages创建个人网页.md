---
title:   Github + Jekyll 搭建博客
date:   2016-10-10 22:00:00 +0800
categories: 笔记
tags:
  - github paages
---

* content
{:toc}


## 使用Github Pages创建个人网页

[参考1](https://pages.github.com/)

[参考2](http://www.cnblogs.com/crazyacking/p/4678976.html)

[参考3](http://www.jianshu.com/p/9a6bc31d329d)

直接将站点托管在github仓库

步骤

1\. Create a repository  
命名规则username/username.github.io

2\. Clone the repository  
将项目下载到本地

3\. Create an index file  
添加index.html文件编写html设置站点，也可以直接使用站点主题。站点主题步骤：在个人github pages的repository页面Settings，点击Update site的"Launch automatic page generator"按钮，根据指引选择主题，最后publish page即可。

4\.  Commit  

----------

## Jekyll（本文中为Windows平台）
　　jekyll是一个简单的免费的Blog生成工具，可以生成静态网页，github pages支持Jekyll。
​	
### 安装Ruby和RubyDevKit
[参考](http://jekyll-windows.juthilo.com/1-ruby-and-devkit/)

Ruby是Jeky所使用的编程语言。

[下载Ruby for Windows](http://rubyinstaller.org/downloads/)

安装过程注意勾选“Add Ruby executables to your PATH”，否则需要自己配置环境变量

[下载Ruby DevKit](http://rubyinstaller.org/downloads/)

Ruby v2.0.0对应版本以DevKit-mingw64开头

打开DevKit安装解压(建议使用简单解压路径，如C:\RubyDevKit)完成后，执行以下指令与Ruby关联：

> cd C:\RubyDevKit 
> ruby dk.rb init 
> ruby dk.rb install

### 安装Jekyll

> gem install Jekyll

如果上述指令执行失败，则使用

> gem sources --remove https://rubygems.org/
> gem sources -a http://rubygems.org/ 
> gem install jekyll

执行

> jekyll --version

查看是否安装成功

进入repository目录
执行
> jekyll serve


启动本地服务，即可通过http://localhost:4000访问blog页面。



　　jekyll目录结构

```
├── _config.yml              (配置文件)
├── _drafts                  (drafts（草稿）是未发布的文章)
|   ├── begin-with-the-crazy-ideas.textile
|   └── on-simplicity-in-technology.markdown
├── _includes             (加载这些包含部分到你的布局)
|   ├── footer.html
|   └── header.html
├── _layouts                 (包裹在文章外部的模板)
|   ├── default.html
|   └── post.html
├── _posts                   (这里都是存放文章)
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.textile
|   └── 2009-04-26-barcamp-boston-4-roundup.textile
├── _site                 (生成的页面都会生成在这个目录下)
├── .jekyll-metadata      (该文件帮助 Jekyll 跟踪哪些文件从上次建立站点开始到现在没有被修改，哪些文件需要在下一次站点建立时重新生成。该文件不会被包含在生成的站点中。)
└── index.html            (网站的index)
```

### 添加Jekyll theme
[参考](https://help.github.com/articles/adding-a-jekyll-theme-to-your-github-pages-site/)

　　可以通过下载[jekyll主题](http://jekyllthemes.org/)布置博客的外观。直接将下载下来的主题替换自己repository中文件即可(一般只保留README.md)。具体的使用设置可以对照主题主页的说明。
​	
## 通过MarkDown写文章
　　使用MarkDown工具写好文章，题头加上以下格式：

	---
	layout: post
	title: title
	author: author
	date: 2016.10.10 22:00:00 +0800
	categories: Blog
	tag: Blog
	---

　　保存后缀为.md，放置在_post文件夹中，再通过git上传到服务器，过几分钟就可以看到新的文章。　　　　

## 寄语  
　　拖拖拉拉的，终于算是入门了MarkDown和Github Pages，虽然总感觉自己写的文章连通顺都得反复考量，但却希望能以这里为一个起点，沉下心，一点点得积累，到达自己想到的远方，即使是连自己都不知道哪里是所谓的远方，但既然选择了这条道路，在没有改道之前，我都只能把坚定留给自己。



