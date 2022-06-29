---
title: 部署Hexo显示Permission Denied的解决方法
date: 2022-06-11 22:41:27
categories:
- 构建
tags:
- hexo

cover: https://s2.loli.net/2022/06/29/4C2vtNAhBYdrs8k.jpg
---

## 第一步

### 查看_config.yml 中git的配置,和GitHub的ssh配置

 _config.yml 中git的配置如下:

``` bash
deploy:
  type: git
  repo: git@github.com:start-spt/start-spt.github.io.git
  branch: master
```

## 第二步

### 第一步检查无误后,仍然存在问题,删除deploy_git,重新以下,操作

``` bash
$ hexo clean
$ hexo generate
$ hexo deploy
```


