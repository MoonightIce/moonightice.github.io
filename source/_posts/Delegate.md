---
title: Delegate
date: 2018-08-30 14:54:45
tags:
  - 代理
categories:
  - 基础
index_img: /images/CoverImage/wallhaven-eyz82r.jpg
---

# Delegate 使用

- 在被代理的对象中，声明代理和代理方法。在类中弱引用代理对象
  弱引用原因:因为 View 作为 delegate 对 ViewController 是强引用，同时 ViewController 对 View 也是强引用，这就出现了循环引用。ViewController 引用着 View，并且 View 引用着 ViewController，两者的引用计数都不会变成 0，所以它们都不会被销毁，从而造成内存泄露

```示例
//在类之前声明
protocol objectNeedDelegate: class {
    // class关键字用来约束协议，使得只能被引用类型的数据(也就是类)使用
    func objectMethod() -> Void
}
//在类中声明
class className: superClass{
    //弱引用变量，指向引用类型（UIViewController）
    weak var delegate : objectNeedDelegate?
    func useDelegate(){
        delegate.?.objectMethod()
    }
}
```

<!-- more -->

![DelegateImage](/images/delegate.png)

- 在代理对象中，声明实现代理和代理方法。

```示例
//在类之前声明
class  delegateMakeClass: superClass,objectNeedDelegate {
    let object = className()
    object.delegate = self
    func useDelegate(){
        //实现代理方法
    }
}
```
