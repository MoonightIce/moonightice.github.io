---
title: SwiftLint 规则-中文
date: 2019-05-21 09:56:44
tags:
  - 静态编译
categories:
  - 开发
index_img: /images/CoverImage/wallhaven-xldwp3.jpg
---

> 对于规则的简单描述，需要使用对应标识符以及更具体功能参考官网
> [Rules](https://github.com/realm/SwiftLint/blob/master/Rules.md#force-cast)

## AnyObject Protocol

> Prefer using AnyObject over class for class-only protocols.

对于类的代理，使用 AnyObject 比使用 class 作为继承类更好
