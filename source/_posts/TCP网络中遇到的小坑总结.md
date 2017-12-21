---
title: TCP网络
layout: post
date:  
author: "refraincc"
tags:
- Coder
---


#TCP网络中遇到的小坑总结
最近公司项目需要搭建一个TCP网络，因为iOS系统中是采用小端序，而后端是采用的大端序所以在截取Dada文件的时候，截取大小对应不上，查阅各方资料终于截取成功，代码如下：

```dash

- (void)socket:(GCDAsyncSocket *)sock didReadData:(NSData *)data withTag:(long)tag{
    
    NSInteger le;
    
    [data getBytes:&le length:2];
        
    uint16_t len = ntohs(le);

    NSData *subData = [data subdataWithRange:NSMakeRange(2, len)];
    
    NSString *s = [[NSString alloc]initWithData:subData encoding:NSUTF8StringEncoding];
    
    NSLog(@"s = %@",s );
    
}

```

核心代码是用`ntohs()`方法将截取的`headData`中的`cotentData`的`lenght`转换成小端序，这样就转换出了正确的contentData长度，从而正确解析contentData数据

