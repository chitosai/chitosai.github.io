---
author: TheC
comments: true
date: 2011-12-09 15:52:28+00:00
layout: post
slug: some-updates
title: 更新了点装备
wordpress_id: 69
categories:
- 杂物屋
---

预感到再看DSP我就要悲剧了，于是放弃了一个晚上做些自己喜欢的事情。

这次给图片增加了thickbox，因为BGM用的这个所以感觉比较亲切，而且刚好WP也自带了，直接 wp_enqueue_script('thickbox'); 就可以了，轻松愉快~

效果是这样的

<p class="image-placeholder">[原图已丢失]</p>



然后给WP增加了minify和gzip，WP好像是自带gzip的只要在index.php里添加ob_start('ob_gzhandler');就可以了，相当方便

但是minify他真是个愁人的货啊！！

WP Minify，这个看起来好像是为了简化而专门为WP做的插件，老纸就是玩不转啊！！


> How Does it Work?

> Upon page load, WP Minify intercepts scripts and style printing at the ‘wp_print_scripts’ and ‘wp_print_styles’ hook. WP Minify grabs these files in the proper order (minding dependencies) and passes that list to the Minify engine. The Minify engine then returns a consolidated, minified, and compressed script or style. WP Minify then references this compressed script or style in the WordPress header instead of each individual scripts/styles.


WP Minify会查找wp_print_scripts钩子，按照里头写的文件关系压缩后输出，查了WP文档之后，貌似这样写：


    function add_scripts(){
       wp_enqueue_script('jQuery');
       wp_enqueue_script('thickbox',$deps = array('jQuery'));
    }
    add_action('wp_print_scripts','add_scripts');


因为wp_head会去调用wp_print_scripts钩子，所以会在wp_head时输出这两个js。

然后呢？js是输出了，说好的压缩呢？！老纸的压缩在哪里？！

还不如一开始就直接下个普通的minify用呀！ = =

说起来，因为minify直接访问时的页面太简单易懂了，一直没认真看过底下的英文说明，原来minify的真谛group以前一直忽略了...

于是经过重新整理，现在的首页只有1个JS请求+1个CSS请求，感觉舒服多了~XD

顺带，做了个新的404页面w

一直记不住她名字的长野原美绪~跟佑子在一起你的名字复杂到爆了，完全记不住啊！

<p class="image-placeholder">[原图已丢失]</p>
