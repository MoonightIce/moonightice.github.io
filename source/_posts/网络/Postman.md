---
title: Postman
date: 2018-10-29 16:36:54
tags:
  - API
categories: 工具
index_img: /images/CoverImage/wallhaven-r2pqk1.jpg
---

# PostMan 使用笔记

![Postman](/images/PostMan/Postman.png)

> 一次使用 API 调试工具，之间都是之间采用的网页端输入各项信息，耗费时间过多且内容重复，学习使用 Postman

<!-- more -->

1. 所使用的是 Chrome 插件版本。在之后应该会换成 Mac 版
2. 简单的写下现在所知道的一些功能
   - 可根据网页返回的内容设置全局变量和环境变量
   - 可分组保存，而且可以统一 Run 测试接口
   - 可导出生成 Markdown 格式接口文档

![Postman](/images/PostMan/Postman-Environment.png)

- 环境变量，主要用于切换服务器地址，测试或者正式服务器
- 全局变量，主要是每个环境都需要用到且不容易改动

![Postman](/images/PostMan/ManagerEnvironment.png)

![Postman](/images/PostMan/ManagerEnvironmentList.png)

- Globlas 里设置的是全局变量
- 使用变量的时，需要使用双花括号
- 在 URl 地址，Header，Pre-request Script，Tests 中都可以使用变量
- 在 request 之后 _{}_ 中的变量会被替换成实际的值

![Postman](/images/PostMan/Postman-URL.png)
![Postman](/images/PostMan/ManagerEnvironmentKeyValues.png)

- body 部分是用来传参数，以登录是需要的用户名和密码为例

![Postman](/images/PostMan/Postman-Bodys.png)

- header 头部文件，用到的是为 ck
- 自带的选项还没有进行研究了解

![Postman](/images/PostMan/Postman-Header.png)
![Postman](/images/PostMan/Postman-Headers.png)

# Pre-request Script && Tests

- Pre-request Script 脚本是在执行 request 请求之前运行，可以在里面预先些所需的变量
- Tests 脚本是在 request 请求之后运行，可以对返回信息进行提取过滤，比如例子中，在脚本中动态设置了全局变量 ck 值
- 脚本运行成功后，或在下方的提示框中可查看，会出现绿色的 PASS 提示框

![Postman](/images/PostMan/Postman-Tests.png)

[参考地址-Postman-变量/环境/过滤](https://blog.csdn.net/zxz_tsgx/article/details/51681080)
