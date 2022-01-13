---
title: Stack
date: 2019-09-18 15:51:21
tags:
  - stack
categories:
  - 算法
index_img: /images/CoverImage/63ed7dbf6d7794eb050460d44ca972fe.jpg
---

[Swift - 栈](https://www.raywenderlich.com/800-swift-algorithm-club-swift-stack-data-structure)

文章时间 Dec 16 2016

翻译编写时间 Sep 18 2019

leetcode 相关 ： 20,155,232,844,224,682,496.

栈，和数组有相似的地方，但是有着有限的功能。你只能 push 在栈顶来增加一个新的元素， pop 来移出栈顶元素，peek 查看栈顶元素而不做其余的操作。

在许多的算法中，你需要在某些时刻将对象添加到临时列表，一段时间后再把他们从列表中删除。很多时候，添加和删除的顺序很重要。

栈使用的是 LIFO （last-in first-out order）后进先出的顺序。

利用 array 来实现栈

```swift
struct Stack {
  fileprivate var array: [String] = []
}
```

### Push

在数组里往后添加元素，而非将新的元素插入到第一个的位置。这样的目的是减少时间复杂度，前者只需要 O(1)，后者需要 O(n)

```swift
// 1
mutating func push(_ element: String) {
  // 2
  array.append(element)
}
```

### CustomStringConvertible

`joined(separator:)` 将所有的对象凭借到一起，每个对象之间插入方法的参数

`["3D Games by Tutorials", "tvOS Apprentice"]` -> `"3D Games by Tutorials\ntvOS Apprentice"`

```swift
// 1
extension Stack: CustomStringConvertible {
  // 2
  var description: String {
    // 3
    let topDivider = "---Stack---\n"
    let bottomDivider = "\n-----------\n"

    // 4
    let stackElements = array.reversed().joined(separator: "\n")
    // 5
    return topDivider + stackElements + bottomDivider
  }
}
```

### Pop

```swift
// 1
mutating func pop() -> String? {
  // 2
  return array.popLast()
}
```

### 泛型

```swift
struct Stack<Element> {
  // ...
}
```

```swift
struct Stack<Element> {
  fileprivate var array: [Element] = []

  mutating func push(_ element: Element) {
    array.append(element)
  }

  mutating func pop() -> Element? {
    return array.popLast()
  }

  func peek() -> Element? {
    return array.last
  }
}
```

```swift
// previous
let stackElements = array.reversed().joined(separator: "\n")

// now
let stackElements = array.map { "\($0)" }.reversed().joined(separator: "\n")
```

### 总结

```swift
public struct Stack<T> {
  fileprivate var array = [T]()

  public var isEmpty: Bool {
    return array.isEmpty
  }

  public var count: Int {
    return array.count
  }

  public mutating func push(_ element: T) {
    array.append(element)
  }

  public mutating func pop() -> T? {
    return array.popLast()
  }

  public var top: T? {
    return array.last
  }
}
```
