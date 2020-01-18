---
type: redis哨兵机制
title: Gallery Post
date: 2014-11-18 15:45:20
category: Photo
photos:
- http://ww1.sinaimg.cn/mw690/81b78497jw1emfgwkasznj21hc0u0qb7.jpg
- http://ww3.sinaimg.cn/mw690/81b78497jw1emfgwjrh2pj21hc0u01g3.jpg
- http://ww2.sinaimg.cn/mw690/81b78497jw1emfgwil5xkj21hc0u0tpm.jpg
- http://ww3.sinaimg.cn/mw690/81b78497jw1emfgvcdn25j21hc0u0qpa.jpg
tags:
- consectetur
description: Gallery Post Test. 测试图片类文章的显示。
---

首先，每个哨兵节点每秒都会向主从服务器，其它的哨兵节点发送心跳包（检测是否在线），如果在配置时间内主节点都返回无效的回复，则该哨兵会认为主节点客观下线。为了确定主服务器是否真的下线，该哨兵会向其它哨兵节点询问，如果收到确认主服务器下线回复超过了规定值，则该哨兵节点会认为主服务器客观下线。 所有判断主服务器客观下线的哨兵都有机会成为负责下线主服务器的故障处理的主哨兵节点。它们在确定主服务器下线后，后发起选举，它会向其它哨兵节点发消息告诉它们自己要当主哨兵，让其它哨兵投票，在一次故障处理过程中每个哨兵只能投票一次，如果发起投票的哨兵的选票达到了半数以上，那么它就会成为主哨兵节点。所以说先到先得，如果某个哨兵已经投票给其它的哨兵，那么再有请求发来也不能再进行投票。 你提到的3/2+1=2.5这个半数就是1.5，大于1.5票就可以，比如一共三个哨兵，哨兵A判断主服务器客观下线，它发起投票，自己投自己一票，哨兵B投了它一票，这时候它有两票大于半数以上。当哨兵数量是4个时，确实是得有3票才算是半数以上。所以倾向于哨兵总数是奇数个，但不绝对。

<!-- more -->

![Wallbase - dgnfly (wallbase.cc/wallpaper/1384450)](http://ww1.sinaimg.cn/large/81b78497jw1emfgts2pt4j21hc0u0k1c.jpg)

Etiam luctus mauris at mi sollicitudin quis malesuada nibh porttitor. Vestibulum non dapibus magna. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Proin feugiat hendrerit viverra. Phasellus sit amet nunc mauris, eu ultricies tellus. Sed a mi tortor, eleifend varius erat. Proin consectetur molestie tortor eu gravida. Cras placerat orci id arcu tristique ut rutrum justo pulvinar. Maecenas lacinia fringilla diam non bibendum. Aenean vel viverra turpis. Integer ut leo nisi. Pellentesque vehicula quam ut sapien convallis consequat. Aliquam ut arcu purus, eget tempor purus. Integer eu tellus quis erat tristique gravida eu vel lorem.
