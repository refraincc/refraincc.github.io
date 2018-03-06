---
title: NSCache相关
layout: post
date:  2016-07-15 21:09:13
author: "refraincc"
tags:
	- Coder
---

#NSCache的一些简单有点

当系统资源将要耗尽的时候，NSCache具备自动删减缓冲的机制，并且还会先删减==最近未使用==的对象。
NSCache不拷贝键，而是保留键。因为并不是所有的键都遵从拷贝协议（字典的键是必须要遵从拷贝协议的，有局限性）
NSCache是有==线程安全==的：不编写加锁代码的前提下，多个线程可以同时访问NSCache。









