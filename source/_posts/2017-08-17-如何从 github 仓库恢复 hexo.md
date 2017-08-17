---
title: 如何从github仓库恢复hexo
tags:
  - hexo
date: 2017-08-17 15:37:22
categories: 笔记
---

首先，将 github 上 hexo 网页所在仓库 clone 下来，如果使用了多分支管理，需要切换到相应分支。

clone 后的仓库里是没有 hexo 依赖的，所以需要重新配置。

重新安装 hexo（执行完这个指令后，生成、部署都可以使用了，不需要重新安装 git 的部署器，暂时也没发现有什么问题）

> $ npm install hexo --save

安装 hexo-server（否则，`hexo s` 无效）

> npm install hexo-server --save