---
title: LeetCode --- 字符串
date: 2018-08-28 15:24:10
tags:
- 字符串
categories: 算法
index_img: /images/CoverImage/wallhaven-j5xlow.jpg
---

# 字符串倒叙输出

## 原题是在Codewars上要求是reverse（“hello world”）==“world hello ”   reverse（“hi there”）==“there hi”

```Head
#import <Foundation / Foundation.h>
NSString * reverse（NSString * text）{
    //字符串变成数据，根据引号里对象分离
    NSArray * words = [text componentsSeparatedByString：@“”];
    //数据倒序遍历，所有对象
    NSArray * reversed = [[words reverseObjectEnumerator] allObjects];
    //数组变成字符串，根据引号内对象隔开
    return [reversed componentsJoinedByString：@“”];
}
```

<!-- more -->

下面的是题目中一开始给好的方法结构

```Head
＃import<Foundation/ Foundation.h>
NSString*   reverse(NSString *text){
    return text;
}
```

在上述的解决方案中，存在一个隐患。如果存在多个空格符，不影响输出结果，但是在内存地址上有问题

如“I am a boy”中有三个空格符，当使用下面的方法中

```Head
NSArray *words = [text componentsSeparatedByString:@" "];
```

三个空位符的内存是同一个，如果要根据空格符做出操作会出现跳过其中两个空格符的情况。
[参考](https://blog.csdn.net/leikezhu1981/article/details/16802635)
