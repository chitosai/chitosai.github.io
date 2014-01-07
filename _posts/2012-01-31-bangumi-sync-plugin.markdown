---
author: TheC
comments: true
date: 2012-01-31 17:47:00+00:00
layout: post
slug: bangumi-sync-plugin
title: wp-bangumi同步发文插件
wordpress_id: 352
categories:
- 杂物屋
tags:
- bangumi
- wordpress插件
- 同步
---

前几天写圣诞之吻的吐槽时想着发在自己博客上的与动漫相关的东西如果能自动在bgm上也发一份就好了，于是今天就自己动手写了个。据我观察，bgm上用wp的人貌似还挺多的，不知道有没有人有同样的需求呢~

总而言之 <a href="http://wordpress.org/extend/plugins/wp-bangumi-sync-plugin/" target="_blank">戳我下载</a>

只有7kb的小插件，使用方法也很简单，安装后在"设置->bangumi同步插件"里填入邮箱和密码，点保存后插件就会向bangumi娘求证，如果通过验证就会保存这个账号

![](http://thec.u.qiniudn.com/tMs8n.jpg)

账号通过验证后，在发布文章时就会多出这样一个选项：

![](http://thec.u.qiniudn.com/4wVmG.jpg)

勾选的话就会同时发布到bangumi上，不勾选的话就什么也不会发生

至于html/ubb转换，是按照bgm的ubb帮助页面写的，所以没有对应ubb的html标签一律替换为空白。大致来说，像这样的一段wp的MCE编辑器自动生成的html代码

![](http://thec.u.qiniudn.com/pgIAk.jpg)

发到bgm会是这个样子

![](http://thec.u.qiniudn.com/YW8N4.jpg)

因为bgm的ubb不支持16位颜色代码的缘故，所有带颜色的字一律转为红色，所有带背景的字一律转为[mask]

顺带一提，这篇文章也是插件自动发的~
