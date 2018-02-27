---
layout: hexo
title: npm淘宝镜像使用方法
date: 2018-02-27 15:39:40
tags: tools
---

淘宝 npm 地址： http://npm.taobao.org/

## 1.临时使用


``` bash
$ npm --registry https://registry.npm.taobao.org install express
```


## 2.持久使用


``` bash
$ npm config set registry https://registry.npm.taobao.org
```
配置后可通过下面方式来验证是否成功 

``` bash
$ npm config get registry
```
或 
``` bash
$ npm info express
```

## 3.通过cnpm使用


``` bash
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```

使用 

``` bash
$ cnpm install express
```
