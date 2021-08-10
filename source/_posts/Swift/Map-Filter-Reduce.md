---
title: Map Filter Reduce
date: 2019-11-05 16:45:11
tags:
categories:
index_img: /images/CoverImage/woman-in-black-bikini-standing-on-shore-825904.jpg
---

# Map Filter Reduce

- map 函数返回一个包含了对原集合中每一个元素经过映射后的 Array。

- filter 函数返回一个包含原集合中满足筛选条件的元素的 Array。

- reduce 函数返回一个初始参数与原集合元素经过组合后的非集合类型的值。

## Map

> 遍历集合中每一个元素进行相同的操作

- 使用 for-in 循环来计算每一个元素的平方

```示例
let values = [2.0,4.0,5.0,7.0]
var squares: [Double] = []
    for value in values {
  squares.append(value*value)
}
```

- 使用 map 函数
- map 函数有一个唯一的闭包类型参数，遍历整个集合。这个闭包会取得其中的每一个元素作为参数并返回一个经过处理的值
- map 将经过处理的值以 Array 形式返回

```示例
let values = [2.0,4.0,5.0,7.0]
let squares = values.map {$0 * $0}
```

- 这种方式编写 map 函数可以让我们更容易的看出发生了什么

```示例
let values = [2.0,4.0,5.0,7.0]
let squares2 = values.map({
    (value: Double) -> Double in
    return value * value
})
```

- 这个闭包有一个参数：(value: Double)，Swift 可以推断出闭包的返回值为 Double 类型。
- 因为 map 函数只有一个闭包参数，因此我们可以省略掉 ( 和 ) ，同时省略掉 return：

```示例
let squares2 = values.map {value in value * value}
```

- in 关键字的作用是将闭包的参数与函数体分开。你可以使用参数编号将代码简化得更彻底

```示例
let squares = values.map { $0 * $0 }
```

- 返回值的类型不会受限于原数组中元素的类型。_下面是一个从整型数组映射到字符串数组的例子_

```示例
let scores = [0,28,124]
let words = scores.map { NSNumberFormatter.localizedStringFromNumber($0,
    numberStyle: .SpellOutStyle) }
    // ["zero", "twenty-eight", "one hundred twenty-four"]
```

- map 操作不局限于数组，只要是集合类型你都可以使用 map。
- 举个例子，对 Dictionary 或 Set 类型使用，返回值一般会是 Array 类型。
- 下面是一个应用在 Dictionary 上的例子
- 在遍历这个集合的时候，我们的闭包有 String 类型 和 Double 类型的两个参数，它们的类型是由组成这个字典的元素中的 key 与 value 的类型决定

```示例
let milesToPoint = ["point1":120.0,"point2":50.0,"point3":70.0]
let kmToPoint = milesToPoint.map { name,miles in miles * 1.6093 }
```

- 最后一个例子，关于 Set 集合

```示例
let lengthInMeters: Set = [4.0,6.2,8.9]
let lengthInFeet = lengthInMeters.map {meters in meters * 3.2808}
```

## Filter

> filter 函数会遍历一个集合，并返回一个 Array,其中包含了集合中满足过滤条件的元素
> filter 函数中有一个参数指定了过滤条件，它是一个闭包，
> 它会从集合中取得一个元素作为闭包的参数，经过闭包处理后返回一个 Bool 类型的值，
> 这个值指示了这个元素是否满足过滤条件

```示例
let digits = [1,4,10,15]
let even = digits.filter { $0 % 2 == 0 }
// [4, 10]
```

## Reduce

> 使用 reduce 来组合集合中的所有元素并返回一个非集合类型的值
> reduce 函数有两个参数，一个为初始值，另一个为组合闭包

```s
  @inlinable public func reduce<Result>(_ initialResult: Result, _ nextPartialResult: (Result, (offset: Int, element: Base.Element)) throws -> Result) rethrows -> Result
```

```示例
let items = [2.0,4.0,5.0,7.0]
let total = items.reduce(10.0,+)
// 28.0
```

```示例
let codes = ["abc","def","ghi"]
let text = codes.reduce("",+)
// "abcdefghi"
```

- 因为组合参数是一个闭包，所以你可以使用尾随闭包语法来编写 reduce

```示例
let names = ["alan","brian","charlie"]
let csv = names.reduce("===") {text, name in "\(text),\(name)"}
// "===,alan,brian,charlie"
```

### 进阶

```s
  Apple 文档中的完整方法
  @inlinable public func reduce<Result>(into initialResult: Result, _ updateAccumulatingResult: (inout Result, (offset: Int, element: Base.Element)) throws -> ()) rethrows -> Result
  示例 1
    ///     let letters = "abracadabra"
    ///     let letterCount = letters.reduce(into: [:]) { counts, letter in
    ///         // letter(Character)
    ///         counts[letter, default: 0] += 1
    ///     }
    ///     // letterCount == ["a": 5, "b": 2, "r": 2, "c": 1, "d": 1]
  示例 2
    ///     let order: [Int: Int] = arr2.enumerated().reduce(into: [:]) { result, next in
    ///        // next(offset: Int, element: Int)
    ///        // 下列也可写成一行 result[next.element] = next.offset
    ///          let (i, n) = next
    ///          result[n] = i
    ///     }


```



## FlatMap

> 最简单的用法是如同它的名字所描述的那样将一个二维数组拆开展平

```示例
let collections = [[5,2,7],[4,8],[9,1,3]]
let flat = collections.flatMap { $0 }
// [5, 2, 7, 4, 8, 9, 1, 3]
```

> 它可以判断集合中的不可选值，并将不可选值移出集合

```示例
let people: [String?] = ["Tom",nil,"Peter",nil,"Harry"]
let valid = people.flatMap {$0}
// ["Tom", "Peter", "Harry"]
```

> flatMap 的魔力在于你可以拆开展平一个多维数组，并且这个多维数组的子数组可以经过处理，最终返回一个包含所有结果的一维数组

- 遍历一个二维数组并返回一个元素都为偶数的一维数组

```示例
let collections = [[5,2,7],[4,8],[9,1,3]]
let onlyEven = collections.flatMap {
    intArray in intArray.filter { $0 % 2 == 0 }
}
// [2, 4, 8]
```

- 因为 flatMap 遍历的是子数组为整型数组的二维数组，所以它的闭包参数的参数 intArray 是 [Int]类型。
- 这也是为什么你可以将上面的代码利用简洁的闭包语法变成以下更简洁的方式，但我认为这样做会降低代码的可读性

```示例
let onlyEven = collections.flatMap { $0.filter { $0 % 2 == 0 } }
```

- 利用 flatMap 与 map 函数求二维数组的每一个 Int 元素的平方，返回一个 Array

```示例
let allSquared = collections.flatMap { $0.map { $0 * $0 } }

//非简写方式
let allSquared = collections.flatMap {
  intArray in intArray.map { $0 * $0 }
}
```

- 利用 flatMap 与 reduce 函数求出二维数组中每个子数组的元素之和，返回一个包含结果的 Array

```示例
let sums = collections.flatMap { $0.reduce(0, combine: +) }

//Xcode 8.0中Swift语法更为简洁
let sums = collections.flatMap { $0.reduce(0,+) }

//最后一个例子用map函数就可以办到，因为reduce的返回值是Int而不是Array
let sums = collections.map { $0.reduce(0, combine: +) }

//Xcode 8.0中Swift语法更为简洁
let sums = collections.flatMap { $0.reduce(0,+) }
```

## Chaining

> 这些函数链接起来使用
> 例如求数组中大于等于 7 的元素之和，我们可以先用 filter 筛选，其次再用 reduce 求和

```示例
let marks = [4,5,8,2,9,7]
let totalPass = marks.filter{$0 >= 7}.reduce(0,combine: +)
// 24

//Xcode 8.0中Swift语法更为简洁
let marks = [4,5,8,2,9,7]
let totalPass = marks.filter{$0 >= 7}.reduce(0,combine: +)
// 24
```

> 先求数组中的元素的平方，再筛选出偶数

```示例
let numbers = [20,17,35,4,12]
let evenSquares = numbers.map{$0 * $0}.filter{$0 % 2 == 0}
// [400, 16, 144]
```
