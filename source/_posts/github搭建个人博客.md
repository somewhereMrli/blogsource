---
layout: hexo
title: github搭建个人博客
date: 2017-06-06 17:39:40
tags: tools
---

小白一枚，突然想自己搭建一个博客，偶然间发现了hexo这个东东，结合github的免费建站功能，然后就…说干就干，一下记录了搭建的步骤。(ps:适合需要有一点了解git的小伙伴)

首先 我是window10 安装 git nodejs， 什么不会？ 家常问度娘，专业问google，你懂得，这个就不多解释了

## Quick Start

### 安装hexo

``` bash
$ npm install hexo-cli -g
$ hexo init blog
$ cd blog
$ npm install
$ hexo server
```

太慢？npm设置一个淘宝的源吧 看看这个 https://npm.taobao.org/

执行完毕之后，控制台应该会输出

``` bash
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```

简单介绍一下hexo的命令，应该都能看懂

``` bash
D:\blog\somewhere>hexo -help
Usage: hexo <command>
Commands:
  clean     Removed generated files and cache.
  config    Get or set configurations.
  deploy    Deploy your website. #简写 hexo d
  generate  Generate static files. #简写 hexo g
  help      Get help on a command.
  init      Create a new Hexo folder.
  list      List the information of the site
  migrate   Migrate your site from other system to Hexo.
  new       Create a new post.
  publish   Moves a draft post from _drafts to _posts folder.
  render    Render files with renderer plugins.
  server    Start the server. #简写 hexo s
  version   Display version information.
Global Options:
  --config  Specify config file instead of using _config.yml
  --cwd     Specify the CWD
  --debug   Display all verbose messages in the terminal
  --draft   Display draft posts
  --safe    Disable all plugins and scripts
  --silent  Hide output on console
```
More info: [hexo](https://hexo.io/)

### 主题安装

来自知乎大神 https://www.zhihu.com/question/24422335 笔者毅然决然的选择了next 以下是简单的安装步骤
``` bash
$ cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```
接着找到 _config.yml 修改theme为next
``` bash
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```
然后找到位于\themes\next [_config.yml] 这里面有这个主题相关配置
选择 Scheme
Muse - 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
Mist - Muse 的紧凑版本，整洁有序的单栏外观
Pisces - 双栏 Scheme，小家碧玉似的清新
Scheme 的切换通过更改 主题配置文件，搜索 scheme 关键字。 你会看到有三行 scheme 的配置，将你需用启用的 scheme 前面注释 # 去除即可。
``` bash
#scheme: Muse
#scheme: Mist
scheme: Pisces
```
### 发布到github
我们写好博客，不仅想自己能看，也想分享到浩瀚的网络世界，那么github就是我们一个不错的选择，免费，免费，免费，不限流量，缺点静态网页，不过写写博客足够了
首先在git上面创建 yourname.github.io 格式一定要这样，
打开 _config.yml 找到 deploy
``` bash
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/yourname/yourname.github.io.git
  branch: master
```
然后
``` bash
$ hexo d
```
地址栏输入：yourname.github.io.git
什么发布报错？
``` bash
npm install hexo-deployer-git --save
```
