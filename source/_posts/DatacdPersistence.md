---
title: 数据持久化
date: 2018-09-18 15:03:18
tags:
  - 沙盒
  - 数据库
categories: 
- 基础
index_img: /images/CoverImage/wallhaven-499mvw.jpg
---

## UserDefaults

    Let uf = UserDefaults.standard

> 一般用来存储用户信息和基础配置信息 （参考 ： [简书数据持久化思路](https://www.jianshu.com/p/3796886b4953)）

```简单示例
uf.setValue(value,forKey : "Key")   //存
uf.integer(forKey : "Key")    // 取(返回 Int 类型)
```

- OC 中需要调用 synchronize 方法同步
- Swift 则已废弃了此方法，不需要手动调用
- 以上是最基本的用法，但是也是最危险的用法
  1. 在应用内随意覆盖删除数据，直接使用字符串作为 key 值，在存数据和取数据时所使用的 key 时不一致
  2. 作为一个全局的单例, 如果需要存储账户信息, 配置信息, 此时按照最基本的使用方式, 简单的使用 key 来存取数据, 那么 key 值会随着存储的数据越来越多, 到时候不管是新接手的小伙伴还是我们自己都很难明白每个 key 值对应的意义. 也就是说我们不能根据方法调用的上下文明确知道我存取数据的具体含义, 代码的可读性和可维护性就不高

<!-- more -->

## 进阶的两个方面 ----- 一致性和上下文

### 一致性

- 常量保存

```示例
    let defaultStand = UserDefaults.standard
    let defaultKey = "defaultKey"
    defaultStand.set(123, forKey: defaultKey)
    defaultStand.integer(forKey: defaultKey)
//分组存储
    struct AccountInfo
    { let userName = "userName"
    let avatar = "avatar"
    let password = "password"
    let gender = "gender"
    let age = "age" }
// 登录信息
    struct LoginInfo
    { let token = "token"
    let userId = "userId" }
// 配置信息
    struct SettingInfo
    { let font = "font"
    let backgroundImage = "backgroundImage" }

    let defaultStand = UserDefaults.standard
// 账户信息
    defaultStand.set("Chilli Cheng", forKey: AccountInfo().avatar)
    defaultStand.set(18, forKey: AccountInfo().age)
// 登录信息
    defaultStand.set("achj167", forKey: LoginInfo().token)
// 配置信息
    defaultStand.set(24, forKey: SettingInfo().font)
    let userName = defaultStand.string(forKey: AccountInfo().avatar)
    let age = defaultStand.integer(forKey: AccountInfo().age)
    let token = defaultStand.string(forKey: LoginInfo().token)
    let font = defaultStand.integer(forKey: SettingInfo().font)
```

> 以上述为例，将配置信息，用户信息，登录信息三个类归到一个大类中，解决上下文问题

- 上下文

```示例
    struct UserDefaultKeys
        { // 账户信息
            struct AccountInfo
                {
                    let userName = "userName"
                    let avatar = "avatar"
                    let password = "password"
                    let gender = "gender" let age = "age"
                }
        // 登录信息
            struct LoginInfo
                {
                    let token = "token"
                    let userId = "userId"
                }
        // 配置信息
            struct SettingInfo
                {
                    let font = "font"
                    let backgroundImage = "backgroundImage"
                }
        }
    let defaultStand = UserDefaults.standard
// 账户信息
    defaultStand.set("Chilli Cheng", forKey:UserDefaultKeys.AccountInfo().userName)
    defaultStand.string(forKey: UserDefaultKeys.AccountInfo().userName)
```

> 上述方法有一个可改进的地方，就是避免初始化。使用静态变量，直接通过类型名字访问属性的值。

```上下文
    struct AccountInfo {
        static let userName = "userName"
        static let avatar = "avatar"
        static let password = "password"
        static let gender = "gender"
        static let age = "age" }

        defaultStand.set("Chilli Cheng", forKey: UserDefaultKeys.AccountInfo.userName)
        defaultStand.string(forKey: UserDefaultKeys.AccountInfo.userName)
```

#### 枚举分组存储

> 可以让我们不用每次都手动设置初始值，前提是遵守 String 协议
> 不设置 rawValue 的时候，系统会默认给枚举 case 设置跟成员名字相同的原始值 （rawValue）

```枚举
    struct UserDefaultKeys {
    // 账户信息
        enum AccountInfo: String {
            case userName
            case age
        }
    }
    // 存账户信息
      defaultStand.set("Chilli Cheng", forKey: UserDefaultKeys.AccountInfo.userName.rawValue)
      defaultStand.set(18, forKey: UserDefaultKeys.AccountInfo.age.rawValue)
    // 取存账户信息
      defaultStand.string(forKey: UserDefaultKeys.AccountInfo.userName.rawValue)
      defaultStand.integer(forKey: UserDefaultKeys.AccountInfo.age.rawValue)
```

### 优化

> 代码过于冗余
> 优化 —— 先抓鱼，砍熊掌
> Key 值路径过长 --- 原因： 分组存储 key，让 key 有上下文，可读性高
> rawValue 没有必要写 --- 原因： 使用枚举存储 key，无需手动设置初始值

#### 扩展 UserDefault 自定义方法

```扩展
    extension UserDefaults {
        enum AccountKeys: String {
            case userName
            case age
            }
        static func set(value: String, forKey key: AccountKeys) {
            let key = key.rawValue UserDefaults.standard.set(value, forKey: key)
        }
        static func string(forKey key: AccountKeys) -> String? {
            let key = key.rawValue
            return UserDefaults.standard.string(forKey: key)
        }
    }
    // 存取数据
      UserDefaults.set(value: "chilli cheng", forKey: .userName)
      UserDefaults.string(forKey: .userName)
```

##### 前置上下文

> 实现前置，需扩展 UserDefault ，添加 AccountInfo 属性，再调用 AccountInfo 方法，key 值由 AccountInfo 来提供,
> 因为 AccountInfo 提供分组的 key，自定义的一个分组信息, 需要实现既定方法，想到使用协议。
> Swift 中可以对协议 protocol 进行扩展, 提供协议方法的默认实现, 如果遵守协议的类/结构体/枚举实现了该方法, 就会覆盖掉默认的方法.

````上下文
    protocol UserDefaultsSettable {

    /* 只要 AccountInfo 类/结构体/枚举遵守这个协议, 就能调用存取方法,至关重要的问题在于， AccountKeys 从哪儿来?
    我们上面是把 AccountKeys 写在UserDefaults扩展里面的, 在协议里面如何知道这个变量是什么类型呢?
    而且还使用到了 rawValue, 为了通用性, 那就需要在协议里关联类型, 而且传入的值能拿到 rawValue,
    那么这个关联类型需要遵守 RawRepresentable 协议, 这个很关键!!! */

    associatedtype defaultKeys: RawRepresentable }

    /*必须在扩展中使用 where 子语句限制关联类型是字符串类型, 因为 UserDefaults 的 key 就是字符串类型.
    where defaultKeys.RawValue==String，其他类型同理*/

    extension UserDefaultsSettable where defaultKeys.RawValue==String {
        static func set(value: String?, forKey key: defaultKeys) {
            let aKey = key.rawValue
            UserDefaults.standard.set(value, forKey: aKey)
        }
            static func string(forKey key: defaultKeys) -> String? {
                let aKey = key.rawValue
                return UserDefaults.standard.string(forKey: aKey)
            }
    }

```上下文

在 UserDefault 的扩展中定义分组

```上下文
    extension UserDefaults {
        // 账户信息
        struct AccountInfo: UserDefaultsSettable {
                enum defaultKeys: String {
                  case userName
                  case age
                  }
            }
        // 登录信息
        struct LoginInfo: UserDefaultsSettable {
            enum defaultKeys: String {
                case token
                case userId
            }
        }
    }
    //存取数据
    UserDefaults.AccountInfo.set(value: "chilli cheng", forKey: .userName)
    UserDefaults.AccountInfo.string(forKey: .userName)

    UserDefaults.LoginInfo.set(value: "ahdsjhad", forKey: .token)
    UserDefaults.LoginInfo.string(forKey: .token)
````

如果还有需要存储的分类数据, 同样在 UserDefaults extension 中添加一个结构体, 遵守 UserDefaultsSettable 协议, 实现 defaultKeys 枚举属性

在枚举中设置该分类存储数据所需要的 key.UserDefaultsSettable 协议中只实现了存取 string 类型的数据,可以自行在 UserDefaultsSettable 协议中添加 Int, Bool 等类型方法
