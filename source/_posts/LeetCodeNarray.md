---
title: LeetCode --- 数组类算法
date: 2018-08-28 09:24:31
tags:
- 数组
categories: 算法
index_img: /images/CoverImage/wallhaven-yj6ep7.jpg
---

# LeetCode ---  数组算法

## 从排序数组中删除重复项

> 不使用额外的数组空间，空间复杂度为O(1)
> 排序数组，不考虑非连续重复
> 原地修改输入，修改输入数组，返回数组长度

```示例
class Solution {
    func removeDuplicates(_ nums: inout [Int]) -> Int {
        if nums.count == 0 || nums.count == 1{
            return nums.count
        }
        var position = 0 //第二个指针，指向当前不重复位置
            for i in 0..<nums.count { //第一个指针，指向对比位置
                if nums[position] != nums[i]{//遍历比较到第一个不相等的元素
                    position += 1 //移动第二个指针，准备修改数组
                    nums[position] = nums[i]
                }
            }
        return position + 1//末位修改元素下角标
    }
}

let solution = Solution()

var numsArray = [0,0,1,2,3,3,5,5]

let len = solution.removeDuplicates(&numsArray)

print("\(len)")
```

<!-- more -->

## 买卖股票的最佳时机

>前一个数比后一个数小的时候，买入股票
>不可连续买入，即连续增大时时候不做操作
>前一个数比后一个数大的时候，卖出股票
>连续减下时，如若已经买入则最后一个值收益最大，未买入则一直不操作

```示例
class Solution {
    func maxProfit(_ prices: [Int]) -> Int {
        if prices.count <= 1 {
            return 0
        }
        var profit = 0 //收益
        var price = -1 //价格,特殊情况 -- 数组为零时会出现漏算
        for j in 0..<prices.count{
            if j  == (prices.count - 1) {
                if price != -1{
                    profit += prices[j] - price
                }
            }else{
                if prices[j+1] > prices[j]{
                    if price == -1 {
                        price = prices[j]
                    }
                }else{
                if price != -1 {
                    profit += prices[j] - price
                    price = -1
                    }
                }
            }
        }
        return profit
    }
}
```

## 旋转数组

> 思考：即最后一位数插入到第一个位置上
> 旋转几步，即是插入几次
> 要求是 使用空间复杂度为 O(1) 的原地算法 ，尽可能多解

### 解法1

- 此方法适用于循环次数较少

```解答
class Solution {
    func rotate(_ nums: inout [Int], _ k: Int) {
        if k > = nums.count{
            k = k%nums.count
        }
        for _ in 0..<k{
            //移除最后一个元素并返回此对象
            let lastnum = nums.removeLast()
            nums.insert(lastnum,at: 0)
        }
    }
}
```

### 解法2

- 此方法适用于循环次数较大，根据下角标直接修改数值

```解答
class Solution {
    func rotate(_ nums: inout [Int], _ k: Int) {
        // 此方法中，应该是在底层额外保存了一份数组，所以不会出现数据错误
        for (index,value) in nums.enumerated(){
            let curIndex = (index + k)%nums.count
            nums[curIndex] = value
        }
    }
}
```

# 存在重复

>当数组过大时，时间复杂度过大

```解答
class Solution {
    func containsDuplicate(_ nums: [Int]) -> Bool {
        for i in 0..<nums.count{
            for j in i..<nums.count{
                if i != j{
                    if nums[i] == nums[j] {
                        return true
                    }
                }
            }
        }
        return false
    }
}
```

- 网上优秀答案思路
    1. 先排序，然后对比前后两个元素的值
    2. 利用 Set 集合去重，若和数组长度不一致则是有相同元素
    3. 利用字典，元素为key，无重复则添加到字典里

# 只出现一次的数字

>先进行了衣蛾大小排序，题目已确定有且仅有，所以目标元素会在偶数下角标

```解答
class Solution {
    func singleNumber(_ nums: [Int]) -> Int {
        if nums.count == 1{
            return nums[0]
        }
        var numArr =  nums
        numArr.sort()
        print("\(numArr)")
        for i in 0..<numArr.count-1{
            print("\(i)")
            if i%2 == 0{
                if numArr[i] != numArr[i+1]{
                    return numArr[i]
                }
            }
            if i == numArr.count-2{
                    return numArr[i+1]
                }
        }
        return 999
    }
}
```

# 两个数组的交集

> 数组转化成 Set 集合求交集
> 判断交集中每个元素的出现次数，选取在两个数组中相等或较小的次数

```解答
class Solution {
    func intersect(_ nums1: [Int], _ nums2: [Int]) -> [Int] {
        let set1 = Set.init(nums1)
        let set2 = Set(nums2)
        let setIn = set1.intersection(set2)
        var tagetArr = Array(setIn)
        for temp in setIn{
        var i = 0
        var j = 0
            for  numone in nums1{
                if temp == numone{
                    i += 1
                }
            }
            for numtwo in nums2{
                if temp == numtwo{
                    j += 1
                }
            }
            var minNum = 0
            if j >= i {
                minNum = i
            } else{
                minNum = j
            }

            if minNum >= 2 {
                for _ in 0..<(minNum-1){
                    tagetArr.append(temp)
                }
            }
        }
        return tagetArr
    }
}
```

# 加一

> 数组中每个元素都是单个正整数
> 特殊情况是在当前数为 9 ，需要进位时改变高位数字

```解答
class Solution {
    func plusOne(_ digits: [Int]) -> [Int] {
        var nums1 = digits
        let count = nums1.count
        var curIndex = count - 1
        let curEle = nums1[curIndex]

        if curEle != 9{
            nums1[curIndex] += 1
        }else{
            nums1[curIndex] = 0
            while curIndex > 0 {
                curIndex -= 1
                if nums1[curIndex] != 9{
                    nums1[curIndex] += 1
                    break
                }else{
                    nums1[curIndex] = 0
                }
            }

            if nums1[0] == 0 {
                nums1.insert(1, at: 0)
            }
        }
        return nums1`
