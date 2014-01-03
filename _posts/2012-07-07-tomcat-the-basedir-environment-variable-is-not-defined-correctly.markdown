---
author: TheC
comments: true
date: 2012-07-07 00:58:31+00:00
layout: post
slug: tomcat-the-basedir-environment-variable-is-not-defined-correctly
title: Tomcat启动报The BASEDIR environment variable is not defined correctly
wordpress_id: 515
categories:
- ACFUN
---

由centOS转向了Ubuntu，期间有各种各样的小麻烦，比如java的安装就不能用rpm包了，需要手动去折腾环境变量什么的。
<!-- more -->
当tomcat装好后，遇到了一个比较诡异的问题：tomcat无法启动，报错：
The BASEDIR environment variable is not defined correctly
This environment variable is needed to run this program 

当时傻乎乎的去找BASEDIR变量，翻看了 catalina.sh，没有发现奇怪的地方，不过想到catalina里面很多include，想了想是不是其他的文件没有权限导致无法运行，于是试验：
chmod +x /usr/local/tomcat6/bin/\*.sh
尝试启动tomcat：
root@ubuntu:/usr/local/tomcat6/bin# ./startup.sh
Using CATALINA_BASE:   /usr/local/tomcat6
Using CATALINA_HOME:   /usr/local/tomcat6
Using CATALINA_TMPDIR: /usr/local/tomcat6/temp
Using JRE_HOME:        /usr/lib/jvm/java/jdk1.6.0_33
Using CLASSPATH:       /usr/local/tomcat6/bin/bootstrap.jar
tomcat启动成功。

