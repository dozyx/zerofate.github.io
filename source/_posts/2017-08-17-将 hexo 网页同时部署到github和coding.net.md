---
title: 同时部署hexo到github和conding.net
tags:
  - hexo
date: 2017-08-17 15:37:22
categories: 笔记
---

### 配置公钥

如果已经在 Github上创建了 ssh key，那么可以直接使用。

+ 复制 `~/.ssh/id_rsa.pub` 内容，在 coding.net 部署公钥里进行添加
+ 执行 `ssh -T git@git.coding.net`，如果返回「Hello zerofate, You've connected to Coding.net via SSH. This is a deploy key.」，表示公钥添加成功



### 部署到 coding.net

打开 hexo 配置文件 `_config.yml`，按以下格式添加部署服务器，如：

> deploy:
>   type: git
>   message: [message]
>   repo:
> ​    github: \<repository url>,[branch]
> ​    gitcafe: \<repository url>,[branch]

执行

> hexo clean
>
> hexo d -g 

就可以部署到 coding.net



> 本来还想将 hexo 迁移到 coding.net 作为私有仓库存放，而 github 只存放部署后的内容，但不清除以后再迁移会 github 时的 commit 记录是否会有问题，所以暂时没弄，以后有时间再研究。