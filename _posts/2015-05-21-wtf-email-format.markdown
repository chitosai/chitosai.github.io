---
author: TheC
comments: true
date: 2015-05-21 11:11:11+00:00
layout: post
title: 邮件在 OSX Mail 下变成纯文本的奇怪 BUG
categories:
- 技术
---

最近在做公司的推广邮件时遇到了一个奇怪的问题，在网页中显示一切正常的邮件，用 OSX 自带的客户端接收后变成纯文本，完全丢失了所有样式和 HTML 标签。

![](http://thec.u.qiniudn.com/20150521113125.jpg)

以前从来没玩过邮件，百度谷歌了半天也没有找到任何结果，只能自己动手了，先看看我收到的邮件的源码：

```
Content-Type: multipart/alternative; BOUNDARY="=_Part_1569072_1859990196.1431331492470"
Message-ID: <AIUAegB5APOxfxbExZ9034p*.1.1431331492470.Hmail.###@###.com>
To: =?UTF-8?B?6ZmI5Y2O546l?= <###@###.com>
Subject: =?UTF-8?B?dGVzdDU=?=
X-Priority: 3
X-Mailer: HMail Webmail Server V2.0 Copyright (c) 2014-163.com
X-Originating-IP: 1.1.1.1
MIME-Version: 1.0
Received: from ###@###.com( [1.1.1.1) ] by ajax-webmail ( [127.0.0.1] ) ; Mon, 11 May 2015 16:04:52 +0800 (GMT+08:00)
From: =?UTF-8?B?6ZmI5Y2O546l?= <###@###.com>
Date: Mon, 11 May 2015 16:04:52 +0800 (GMT+08:00)

--=_Part_1569072_1859990196.1431331492470
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: base64

[ base64 content ]

--=_Part_1569072_1859990196.1431331492470
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: base64

[ base64 content ]

--=_Part_1569072_1859990196.1431331492470--
```

可以看到这封邮件实际上由两个部分组成，第一个部分的 Content-Type 是 text/html，而第二个部分是 text/plain。Base64解码后发现第一部分的内容是由正常的 HTML 组成的，而第二部分是只包含纯文本的内容，我猜这应该是邮件协议的某种 fallback。

于是查了一下 Content-Type: multipart/alternative。

> The multipart/alternative content type is used when the same information is presented in different body parts in different forms. The body parts are ordered by increasing complexity. For example, a message that consists of a heavily formatted Microsoft® Word 97 document might also be presented in Microsoft Word version 6.0 format, rich text format, and a plain text format. In this case the plain text would be presented as the first alternative body part. The rich text version would follow, then the Word 6.0, then the most complex, Word 97. Placing the plain text version first is the friendliest scheme for users with non-MIME-compliant UAs, because they will see the recognizable version first. The MIME-compliant UAs should present the most complex version that they can recognize or give the user a choice of which version to view. 


这样答案就很清楚了，multipart/alternative 确实是某种 fallback 协议。邮件客户端会先读取最后一个 part 的内容，如果不能够正确显示则尝试上一个 part 直到最基础的纯文本内容。而我收到的这封邮件错误地把 text/plain 放在了 text/html 之后，这使得 OSX Mail 误以为这封邮件原本即是纯文本而显示了错误的内容。

现在问题已经找到了，怎么解决呢？然而貌似并没有办法解决，我们作为邮件服务的使用者根本接触不到这么底层的东西。好在经过反复测试，发现这个 BUG 是在同事将邮件转发给过我的过程中发生的，从我们的系统直接发出的邮件并没有这个 BUG，真是万幸。鉴于我没能搜到任何关于这个 BUG 的解释或讨论，就把自己的经验写下来供大家参考。