---
title: Swift 数组遍历
date: 2020-04-10 21:54:55
tags:
- 数组
categories: 
- Swift
index_img: /images/CoverImage/wallhaven-q6qyyd.jpg
---
## sort - sorted

sorted 方法是返回一个结果数组，而 sort 是对原数据进行修改（in place）
两个方法除去名字和返回的对象不同，其余的基本一致

```swift
    @inlinable public func sorted(by areInIncreasingOrder: (Element, Element) throws -> Bool) rethrows -> [Element]

    @inlinable public mutating func sort(by areInIncreasingOrder: (Element, Element) throws -> Bool) rethrows

```

传入的参数一般写法。sort 或 sorted 后面的闭包中主要返回的是排序的循序，true 为正确，false 则是错误。
简单的写法就是 `sort(by: >)` 或者是 `sort(by: <)` 。如果不传入参数默认就是从小到大排序也就是 `sort(by: <)`

```s
    ///     enum HTTPResponse {
    ///         case ok
    ///         case error(Int)
    ///     }
    ///
    ///     let responses: [HTTPResponse] = [.error(500), .ok, .ok, .error(404), .error(403)]
    ///     let sortedResponses = responses.sorted {
    ///         switch ($0, $1) {
    ///         // Order errors by code
    ///         case let (.error(aCode), .error(bCode)):
    ///             return aCode < bCode
    ///
    ///         // All successes are equivalent, so none is before any other
    ///         case (.ok, .ok): return false
    ///
    ///         // Order errors before successes
    ///         case (.error, .ok): return true
    ///         case (.ok, .error): return false
    ///         }
    ///     }
    ///     print(sortedResponses)
    ///     // Prints "[.error(403), .error(404), .error(500), .ok, .ok]"

```

## enumerated
