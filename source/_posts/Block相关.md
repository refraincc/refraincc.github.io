---
title: Block相关
layout: post
date:  2017-04-3 22:43:59
author: "refraincc"
tags:
	- Coder
---

#Block的分类

首先iOS中的Block一共分为三种
```dash
NSConcreteGlobalBlock,  	 全局静态
NSConcreteStackBlock,		 保存在栈中，出函数作用域就销毁
NSConcreteMallocBlock,		 保存在堆中，retainCount == 0销毁
```

Block的copy、retain、release操作不同于NSObjec的copy、retain、release操作：

*	Block_copy与copy等效，Block_release与release等效；
*	对Block不管是retain、copy、release都不会改变引用计数retainCount，retainCount始终是1；
*	NSGlobalBlock：retain、copy、release操作都无效；
*	NSStackBlock：retain、release操作无效，必须注意的是，NSStackBlock在函数返回后，Block内存将被回收。即使retain也没用。容易犯的错误是[[mutableArray addObject:stackBlock]，（补：在arc中不用担心此问题，因为arc中会默认将实例化的block拷贝到堆上）在函数出栈后，从mutableArray中取到的stackBlock已经被回收，变成了野指针。正确的做法是先将stackBlock copy到堆上，然后加入数组：[mutableAarry addObject:[[stackBlock copy] autorelease]]。支持copy，copy之后生成新的NSMallocBlock类型对象。NSMallocBlock支持retain、release，虽然retainCount始终是1，但内存管理器中仍然会增加、减少计数。copy之后不会生成新的对象，只是增加了一次引用，类似retain；
*	尽量不要对Block使用retain操作。

#block要用copy修饰,还是用strong

block本身是像对象一样可以retain，和release。但是，block在创建的时候，它的内存是分配在栈(stack)上，而不是在堆(heap)上。他本身的作于域是属于创建时候的作用域，一旦在创建时候的作用域外面调用block将导致程序崩溃。 使用retain也可以，但是block的retain行为默认是用copy的行为实现的，因为block变量默认是声明为栈变量的，为了能够在block的声明域外使用，所以要把block拷贝（copy）到堆，所以说为了block属性声明和实际的操作一致，最好声明为copy。


#__block和__weak有什么区别？

* __block不管是ARC还是MRC模式下都可以使用，可以修饰对象，还可以修饰基本数据类型。
* __weak只能在ARC模式下使用，也只能修饰对象（NSString），不能修饰基本数据类型（int)
* __block对象可以在block中被重新赋值，__weak不可以。





