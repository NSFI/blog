---
title: 如何发表文章
date: 2017-11-20 15:41:32
tags: how to use
---

## 快速开始
### 获取源码

代码托管在`NSFI`组下的`sf-blog-source`中

``` bash
$ git clone https://github.com/NSFI/sf-blog-source.git
```
### 安装依赖

``` bash
$ npm install
```
### 创建文章
``` bash
$ hexo new '文章title'
```
### 本地服务预览
``` bash
$ hexo server
```
### 生成静态文件
``` bash
$ hexo g
```
### 部署到服务器
``` bash
$ hexo d
```
为了保证统一，部署之前最好将代码推到github库中去以后再执行`hexo d`。

More info: [hexo文档](https://hexo.io/zh-cn/docs/index.html)
 


