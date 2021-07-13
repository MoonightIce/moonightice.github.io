---
title: Swift-闭包
date: 2020-06-03 09:55:19
tags:
- 闭包
categories:
- Swift
index_img: /images/CoverImage/wallhaven-k716d6.jpg
---

## 尾随闭包

> 官方定义：如果需要将闭包表达式作为函数的最终参数传递给函数，并且闭包表达式很长，则可以将其写为尾随闭包。写法：尾随闭包写在函数调用的圆括号之后，即使它是函数的最终参数。如果尾随闭包作为函数的最终参数，并且定义了相应的参数标签，在使用尾随闭包语法时，不能将闭包的参数标签写为函数调用的一部分。

实际需求中需要的将尾随闭包变成可选，也就是根据参数来判断是否需要执行。
虽然没有用 @escaping 关键词，但是因为是可选值所以就成了逃逸闭包

```swift
func esc(_ pk: Int, block: ((String) -> Void)?) {
    if let bbb = block {
        bbb("block")
    }
}
// 执行闭包，尾随闭包写法
esc(100) { (str) in
    print(1)
    print(str)
}
// 不执行闭包
esc(100, block: nil)
```

因为使用的可选值闭包，所以无法同时满足 **可选值和非逃逸**

1. 可以通过提供一个默认值避免可选值
2. 或者通过重载函数，同时提供一个包含可选值逃逸的函数，和一个不是可选值非逃逸的函数

```swift
func esc(_ pk: Int, block: ((String) -> Void)) {
    if let bbb = block {
        bbb("block")
    }
}
```

当使用 nil（或一个可选值）参数时，会调用逃逸闭包的函数
使用闭包字面量会调用非可选值的函数
