---
title: 在Hexo中使用本地图片
date: 2022-06-13 13:15:19
tags:
- hexo
---

## 问题描述
使用Hexo编写博文时，根据Hexo的官方文档使用起来不方便（无法直观地在本地markdown语法显现）。引用站外图片或图床时则不方便本地化操作。

## 解决方案
使用 hexo-asset-image 插件（hexo5版本以上建议使用 hexo-asset-image-for-hexo5 ）可以解决问题。

> 经测试，hexo-asset-image-for-hexo5 插件在使用 Hexo 的 6.0 版本中有效。

## 操作步骤
① 在hexo主目录安装插件，在bash中输入：npm install hexo-asset-image-for-hexo5 --save
② 在主目录的_config.yml文件中查找并修改post_asset_folder值为true：
``` base
post_asset_folder: true
```
③ 在md文件的同级目录里创建同名文件夹，例：
``` bash
├─article
├──test.png
└─article.md
```
④ 写法：在文章里即可使用`![](./article/test.png)`在本地显示图片test.png，同时在静态网页中也可以正常显示。

>注意事项：
- 1、创建文件夹名称要和md文件名称保持一致；
- 2、设置post_asset_folder: true后，若使用 hexo new 命令创建新博文时会自动创建同名文件夹；
