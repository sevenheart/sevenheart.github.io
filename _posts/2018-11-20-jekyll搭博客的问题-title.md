---
title:  jekyll搭博客的问题
date: 2018-12-24 23:31:06
categories:
- jekyll
tags:
 - jekyll
 - ruby
typora-root-url: ..\sources
---



#### jekyll遇到的问题记录

#### 本地环境搭建

按照github中搭建教程：

1.Check whether you have `Ruby 2.1.0` or higher installed:

​    检查ruby版本是不是大于2.1.0

2.Install `Bundler`:

​	安装Bundler，可以通过安装ruby是自己的管理工具gem去安装。

```
gem install bundler
```

3.Install Jekyll and other dependencies from the GitHub Pages gem:

​    通过gem安装jekyll以及其他的相关依赖。在项目根路径下运行这条命令自动安装依赖。

```
bundle install
```

这里出现了很多问题，他里边的许多东西版本都有要求，最后我选择的是rubyinstaller-devkit-2.5.7-1-x64.exe，选2.3以上的集成了devkit的这个版本不要看老的教程了很坑人。。。（亲测）。

https://rubyinstaller.org/downloads/地址这是ruby的下载地址。

问题：还是版本问题按照他给的解决方案安装对应的版本即可。

![](/img\bundlerError.png)

4.Run your Jekyll site locally: 

  通过jekyll启动服务就可以在本地调试了端口4000，可配。

```
bundle exec jekyll server
```

错误：

![](/img\jekyllserveError.png)

这里的问题是在配置文件中找不到仓库的配置，去config文件配一下即可。

在config.yml文件中添加`repository: henry/henrydf.github.io`.根据自己的改变。

具体来源：https://github.com/jekyll/jekyll/issues/4705

**详细配置参考github官方的才是最权威的：**

https://help.github.com/en/github/working-with-github-pages/testing-your-github-pages-site-locally-with-jekyll

**主题模板选择：**http://jekyllthemes.org/

