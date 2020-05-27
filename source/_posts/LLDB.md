---
title: LLDB
date: 2019-08-22 11:50:13
tags:
  - DEBUG
categories:
  - 开发
index_img: /images/CoverImage/wallhaven-mdgxky.jpg
---

> Discover advanced techniques, and tips and tricks for enhancing your Xcode debugging workflows. Learn how to take advantage of LLDB and custom breakpoints for more powerful debugging. Get the most out of Xcode's view debugging tools to solve UI issues in your app more efficiently.
>
> 发现先进的技术，技巧和诀窍来提高自己 Xcode 的 debug 工作流程。学习如何利用 LLDB 和定制的断点来获得更有效的调试。充分利用 Xcode 的视图调试工具，更有效地解决应用中的 UI 问题

# LLDB

## 输出方式

#### po - 输出对象

| Command         | Alias For                                       | Steps TO Evaluate                                             |
| --------------- | ----------------------------------------------- | ------------------------------------------------------------- |
| po <expression> | expression --object-description -- <expression> | 1. Expression: evaluate <br/>2. Expression: debug description |

​ po 过程

​ LLDB 先把语句生成一小段代码

![12b150c95af8aeeac8c8fd4e46c334a5](https://images.xiaozhuanlan.com/photo/2019/12b150c95af8aeeac8c8fd4e46c334a5.png)

​ 编译执行，在生成获取结果的代码

![87b66f7b2a9b72bb56a81729f472e659](https://images.xiaozhuanlan.com/photo/2019/87b66f7b2a9b72bb56a81729f472e659.png)

​ 然后再编译执行，拿到对应的结果，显示出来

![18f667b879ceb9568b3eeed341b5af75](https://images.xiaozhuanlan.com/photo/2019/18f667b879ceb9568b3eeed341b5af75.png)

#### p - 输出 LLDB 格式数据

| Command | **Alias For** | Steps TO Evaluate                                                  |
| ------- | ------------- | ------------------------------------------------------------------ |
| p       | expression -- | 1. Expression: evaluate <br/>2. Outputs LLDB-formatted description |

p 过程

![242ff4fcf39db08ab2380c4f58de7f44](https://images.xiaozhuanlan.com/photo/2019/242ff4fcf39db08ab2380c4f58de7f44.png)

一直到取到结果，p 和 po 的行为一模一样。区别在于 p 使用了 dynamic type resolution （动态类型推断）

![7e98e6bc799197d68f8b8ed62a21f7f4](https://images.xiaozhuanlan.com/photo/2019/7e98e6bc799197d68f8b8ed62a21f7f4.png)

在动态类型推断之后，还有一步格式化

![4c88d4ff0d61d22e1ea1143ed47acd68](https://images.xiaozhuanlan.com/photo/2019/4c88d4ff0d61d22e1ea1143ed47acd68.png)

####

#### V

（Xcode 10.2 引入的 alias，之前版本需要使用 frame variable）

frame variable 不同之处的是从当前 frame 调用栈的内存中拿到的值。只接受变量作为参数，不接受表达式。通过`frame variable`命令，可以打印出当前 frame 调用栈的的所有变量。

| Command | **Alias For**  | **Steps TO Evaluate**                                                  |
| ------- | -------------- | ---------------------------------------------------------------------- |
| v       | frame variable | . Reads value of from memory<br/>2. Outputs LLDB-formatted description |

v 并不像`p`和`po`一样，`v`并没有编译执行的能力，但因此速度也更快。它能访问的是当前栈帧能访问到的数据。如果需要一些更复杂的执行代码或是计算一些值，建议还是使用`p`和`po`

当执行`v variable`的时候，会检测当前程序状态，从内存中读出数据，进行（之前说过的）类型推断

![9cf14650b78d34d147fcb987f734bb44](https://images.xiaozhuanlan.com/photo/2019/9cf14650b78d34d147fcb987f734bb44.png)

如果有访问变量的子属性，例如`v variable.field1.field2`，则会不断的重复读内存和类型推断的行为，最后再走到（之前在`p`说过的）格式化。
![img](https://images.xiaozhuanlan.com/photo/2019/89e62a823a1d3c1efc8fca104c6d0df8.png)

如果有访问变量的子属性，例如`v variable.field1.field2`，则会不断的重复读内存和类型推断的行为，最后再走到（之前在`p`说过的）格式化。
![img](https://images.xiaozhuanlan.com/photo/2019/89e62a823a1d3c1efc8fca104c6d0df8.png)

![0737e5483eb3fd9a8b379d6828605cd2](https://images.xiaozhuanlan.com/photo/2019/0737e5483eb3fd9a8b379d6828605cd2.png)

- 只有`po`有描述的过程
- `p`和`v`都有格式化参与
- 因为`po`和`p`有编译执行的能力，所以可以更随意的执行一些逻辑
- 因为`v`访问的是内存中实际的值，类型推断可以不断执行，最终再到格式化逻辑

所以，实际使用还是需要根据情况，需求，选择适合的指令帮助调试

## 定制格式化输出

#### CustomDebugStringConvertible

#### CustomReflectable

![image-20190822104654921](/images/LLDB/CustomReflectable.png)

## 断点

​ `expression`命令可以改变程序当前的各种状态，`e`、`expr` 作为简写也可以实现同样的功能

​ 在 console 和 breakpoint 中都可以使用该命令

​ 在断点中使用，相当于实时插入代码，勾选 option 则执行代码且不在此断点停留

![image-20190822105843514](/images/LLDB/BreakPointExpression.png)

### 伪寄存器

在断点列表界面，选择新增 Symbolic Breakpoint 断点

![B9973C7A-57D6-4E0F-84D5-8677A331CEE7](/images/LLDB/SymbolicBreakpoint.png)

在 symbol 栏目中可以输入任何函数或者方法名称
图示中用的是 OC 表达式，因为 UIKit 是一个 OC 的框架

在确定输入后，在断点下面有一个子行，这是来自调试器的反馈，他告诉我们能够在 UIKit Core 中的一个位置解析这个断点。有可能会解析到多个位置。如果没有看到子条目，表明调试器无法解析你的断点，因此它永远不会命中

![6C0795E4-AA93-437D-94BD-5E805F2AC941](/images/LLDB/SymbolicBreakpointUikit.png)

运行程序后，第一个获取到的断点如图

![32D6D1FA-4AC4-4425-B45F-0814D1090733](/images/LLDB/ResultBreakpoint.png)

arg 为伪寄存器，可用 1，2，3 类似的方法来查看代表着第 1，2，3 个参数的寄存器。
熟悉 objc_msgSend（不熟悉） ,第二个参数应该是选择器。
直接使用 po arg2 时，我们发现并没有看到相对的参数，因为 LLDB 不会自动知道这些参数的类型，所以我们需要做相应的类型转换，（SEL），然后我们就看到了这个消息的选择器 arg3 是传入方法的第一个参数

### 一次性断点

breakpoint set --one-shot true

### 跳过代码

thread jump --by 2 （跳过两行代码）

### watch 断点

在变量视图中，查看所有属性。在属性列表中找到目标属性，右键 "watch "属性""，自动添加断点。在下一次变量的值发生改变的时候暂停调试器

## 修改 UI

### 查找内存地址，仅通过内存地址来操作视图

有属性或输出口的时候，可以通过 po 命令获取到对应的调试描述，但是这个试图控制器下的所有视图成为了问题

利用 Xcode 的可视化视图调试器

使用 UIView 上的调试函数，self.view.recursiveDescription() 。此方法仅用于调试目的，他不是公共 API 的一部分，所以也不会被 Swift 扫描。swift 比较严格，而 OC 代码的运行时特性，可以让调试器使用 OC 语法实现

expression -l objc -o -- [self.view recursiveDescription] 。-l objc 是说给 expression 命令一段 objc 的代码，-o 表示我们调试描述，-- 表示没有更多的选项

但是这段代仍不能实现。这段代码将为 OC 编译创建一个临时表达式上下文，并且不会继承 Swift 框架中的所有变量

expression -l objc -o -- [‘self.view’ recursiveDescription]。将 self.view 放入反引号中，反引号就像预处理器一样，他表示先评估其在当前帧中的内容，并插入结果，然后我们可以评估其余的部分，然后我们就可以得到递归描述了

po 内存地址，并不能得到，应为 swift 不会讲数字视为指针并为你解引用
command alias poc expression -l objc -0 -- 这个可以将`expression -l objc -0 --` 简化为 `poc`命令
poc 内存地址 ，可以查看该对象的调试描述

swift 方法查看调试描述，`po unsafeBitCast(内存地址 ,to: ScoreboardView.self)`，不安全的原因是需要你来提供正确的类型。这个方法的好处是返回了一个经过类型转化的结果，我们可以调用它的函数，例如.frame

修改该对象的中心店位置`po unsafeBitCast(内存地址 ,to: ScoreboardView.self).center.y = 300`

屏幕中并不会立即显示，我们在调试器中暂停了，所以 Core Animation 目前不会有任何视图模块的更改，应用到屏幕的帧缓冲区

但是使用表达式 CAtransaction.flush()，这告诉 Core Animation 更新屏幕的帧缓冲区

#### 步骤

- expression -l objc -O -- [`self.view` recursiveDescription] 得到的是视图结构，在此拿到内存地址

  ![image-20190822112240319](/images/LLDB/image-20190822112240319.png)

- 根据方法 po unsafeBitCast(内存地址 ,to: ScoreboardView.self) ，可以打印或设置相关属性

  ```Swift
  // 打印对象
  (lldb) po unsafeBitCast(0x7fe439d13160, UILabel.self)
  <UILabel: 0x7fe439d13160; frame = (57 141; 42 21); text = 'Label'; opaque = NO; autoresize = RM+BM; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x600003942a30>>

  // 打印中心点坐标
  (lldb) po unsafeBitCast(0x7fe439d13160, UILabel.self).center
  ▿ (78.0, 151.5)
    - x : 78.0
    - y : 151.5

  // 设置中心点坐标
  (lldb) po unsafeBitCast(0x7fe439d13160, UILabel.self).center.y = 300
  ```

- expression CATransaction.flush() 刷新页面

## 自定义 LLDB 命令

例如上面用到的 poc 命令

```
command alias poc expression -l objc -O --
```

用 poc 命令代替 expression -l objc -O --

## 自定义 Python 脚本

nudge 是 python 脚本文件，有苹果官方提供

```swift
command alias poc expression -l objc -O --
command alias 🚽 expression -l objc -- (void)[CATransaction flush]

// 引用 nudge
(lldb) command script import ~/nudge.py
The "nudge" command has been installed, type "help nudge" for detailed help.

// 拿到对象指针
(lldb) po myLabel
▿ Optional<UILabel>
  - some : <UILabel: 0x7fc04a60fff0; frame = (57 141; 42 21); text = 'Label'; opaque = NO; autoresize = RM+BM; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x600001d36c10>>

// Y轴向上偏移5
(lldb) nudge 0 -5 0x7fc04a60fff0
```

参考资料 【1】 Advanced Debugging with Xcode and LLDB -- WWDC 2018 - Session 412 - iOS, macOS, tvOS, watchOS

【2】 WWDC2019 - Session 429 [小专栏](https://xiaozhuanlan.com/topic/2683509174)
