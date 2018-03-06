
---
title: KOV的实现原理
layout: post
date:  2017-6-05 11:39:55
author: "refraincc"
tags:
	- Coder
---

#KOV的实现原理


当你观察一个对象时，一个新的类会被动态创建。这个类继承自该对象原本的类，并重写了被观察属性的setter方法。重写setter方法会负责在调用原setter方法之前和之后，通知所有的观察对象：值的更改。最后通过isa混写(isa-swzzling)把这个对象的isa指针指向这个新创建的子类，对象就神奇的变成了新创建的子类的实例。


####苹果使用了isa(isa-swizzling)来实现KVO


键值观察通知依赖于NSObject的两个方法：`willChangeValueForKey:` 和 `didChangeValueForKey:`。在一个被观察属性发生改变之前，`willChangeValueForKey:`一定会被调用，这就会记录旧的值。而当改变发生后，`observerValueForKey:ofObject:change:context:`会被调用，继而`didChangeValueForKey:`也会被调用。可以手动实现这些调用，蛋很少有人这么做。一般我们只在希望能控制回调的调用时机时才这么做。大部分情况下，改变通知会自动调用。

比如调用`setNow:`时，系统还会以某种方式在中间插入`willChangeValueForKey:`、`didChangeValueForKey:`和`obserValueForKeyPath:ofObject:change:context:`调用。大家可能以为这是因为`setShow`是合成方法，有时候我们也能看到有人这么写代码:
```dash
- (void)setNow:(NSData *)aData{
	[self willChangeValueForKey:@"now"];
	_now = aData;
	[self didChangeValueForKey:@"now"];
}
```
这完全没有必要，不要这么做，这样的话，KOV代码会被调用两次。KVO在调用存取方法之前总是调用`willChangeValueForKey:`， 之后总是调用`didChangeValueForKey:`。怎么做到的呢？答案是通过`isa`混写(isa-swuzzling)。第一次对一个对象调用`addObserver:forKeyPath:options:context:`时，框架会创建这个类的信的KVO子类，并被观察对象转换成新的子类的对象。在这个KVO特殊的子类中，Cocoa创建观察属性的setter，大致工作如下：
```dash 
	- (void)setNow:(NSDate *)aData{
		[self willChangeValueForKey:@"now"];
		[super setValue:aDate forKey:@"now"];
		[self didChangeValueForKey:@"now"];
	}
```
这种继承和方法注入是在运行时而不是编译时实现的。这就是==正确命名==如此重要的原因。==只有在使用KVC命名约定时，KVO才能做到这一点。==

KVO在实现中通过`isa`混写(`isa-swizzling`)把这个对象的`isa`指针(isa指针告诉Runtime系统这个对象的类是什么)指向这个新创建的子类，对象就神奇的变成了新创建的子类的实例。

然而KVO在实现中使用了`isa`混写（`isa-swizzling`），这个的确不容易被发现：Apple 还重写了，覆盖了 `-class`方法并返回原本的类。企图欺骗我们：这个类没有变，就是原本的那个类。。。

但是，假设==被监听的对象==的类对象是`MYClass`， 有时候我们能看到对`NSKOVNotifying_MYClass`的引用而不是对`MYClass`的引用。借此我们得以知道Apple使用了`isa`混写了(`isa-swizzling`)。

那么`willChangeValueForKey:`、`didChangeValueForKey:`和`observeValueForKeyPath:ofObject:change:Context:`这三个方法的执行顺序是怎么样的呢?

`willChangeValueForKey:`、`didChangeValueForKey:`很好理解，`observeValueForKeyPath:ofObject:change:Context:`的执行实际是什么时候呢？

例子：

```dash
- (void)viewDidLoad {
   [super viewDidLoad];
   [self addObserver:self forKeyPath:@"now" options:NSKeyValueObservingOptionNew context:nil];
   NSLog(@"1");
   [self willChangeValueForKey:@"now"]; // “手动触发self.now的KVO”，必写。
   NSLog(@"2");
   [self didChangeValueForKey:@"now"]; // “手动触发self.now的KVO”，必写。
   NSLog(@"4");
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context {
   NSLog(@"3");
}
```


如果单单从下面这个例子的打印上，
	顺序似乎是 `willChangeValueForKey:`、`observeValueForKeyPath:ofObject:change:Context:`、`didChangeValueForKey:`。
	
其实不然，这里有一个`observeValueForKeyPath:ofObject:change:Context:`，和`didChangeValueForKey:`到底谁先调用的问题：如果`observeValueForKeyPath:ofObject:change:Context:`是在`didChangeValueForKey:`内部处罚的操作呢？那么顺序就是`willChangeValueForKey:`、`didChangeValueForKey:`、`observeValueForKeyPath:ofObject:change:Context:`

不信你把`didChangeValueForKey:`注释掉，看下`observeValueForKeyPath:ofObject:change:Context:`会不会执行。


