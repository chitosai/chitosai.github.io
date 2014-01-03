---
author: TheC
comments: true
date: 2012-01-08 14:16:23+00:00
layout: default
slug: link
title: 友情链接
---

迁移博客的时候弄丢了以前的友链表...我有罪（土下座。

<ul class="posts">
{% for link in site.data.links %}
  <li><span>{{ link.desc }}</span> &raquo; <a href="{{ link.url }}">{{ link.name }}</a></li>
{% endfor %}
</ul>