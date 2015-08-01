---
layout: post
title:  "iOS进阶读书笔记"
date:   2015-07-02 22:45:36
categories: jekyll update
---

最近读了下[唐巧](www.devtang.com)写的《iOS进阶》， 补全了很多iOS开发的盲点，总结下自己在开发过程中认知不足的地方。

- GET请求(为何涉及加密传输比如登陆注册不可以用GET请求):GET请求会在浏览器地址栏暴漏网址，然而在客户端不存在这个问题，所以可能有人觉得用不用GET都无所谓，现在知道GET请求会出现在服务器的access log，这是相当危险的。
- 
