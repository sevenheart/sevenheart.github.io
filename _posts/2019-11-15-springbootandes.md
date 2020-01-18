---
title: Springboot整合Elasticsearch遇到的坑'elasticsearchClient', AvailableProcessors is already set to [4]
date: 2019-11-15 21:33:26
tags:
- es
---

## 解决:Error creating bean with name 'elasticsearchClient', AvailableProcessors is already set to [4]
***解决方案：***
遇到这个问题的时候快去关注关注你们自己有没有和**Redis**做过整合，如果做过那么在pom.xml中看看有没有给引入的依赖加上版本号（真的是坑我啊！！）

报错时的样子：

```
   <dependency>
                <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

```
·
    改正后的样子（版本号根据自己的依赖弄）
```bash
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <version>2.1.10.RELEASE</version>
        </dependency>
```
补救办法：如果还是没有解决看下面这篇博客把应该可以！！
[最全的这个问题解决方法点这里](https://blog.csdn.net/q258523454/article/details/82387130)