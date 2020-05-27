---
title: WKWebView默认缓存策略与HTTP缓存协议
date: 2020-05-26 09:03:27
tags: 
- WKWebView
- HTTP
categories: 
- 网络
index_img: /images/CoverImage/wallhaven-ym6lrk.jpg
---

[原文地址：WKWebView默认缓存策略与HTTP缓存协议](https://juejin.im/post/5df75e3a6fb9a016266459da)
[可能是最被误用的 HTTP 响应头之一 Cache-Control: must-revalidate](https://zhuanlan.zhihu.com/p/60357719)

[https://juejin.im/post/5df75e3a6fb9a016266459da](https://juejin.im/post/5df75e3a6fb9a016266459da)

今天同事反应H5更新了资源，但iOS App里面仍然使用的是旧的缓存资源。为什么会这样呢？要弄清楚这个问题，首先得弄清楚WKWebView的缓存原理。

## 一、WKWebView默认缓存策略

下图是[苹果官方文档](https://developer.apple.com/documentation/foundation/nsurlrequestcachepolicy/nsurlrequestuseprotocolcachepolicy?language=occ)提供的默认缓存策略(NSURLRequestUseProtocolCachePolicy)的流程图。

[https://s3-us-west-2.amazonaws.com/secure.notion-static.com/edbc39f4-b706-45cb-949b-b473385c83ee/172020dc96d8ec8f](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/edbc39f4-b706-45cb-949b-b473385c83ee/172020dc96d8ec8f)

官方文档上是这样描述的：

> For the HTTP and HTTPS protocols, NSURLRequestUseProtocolCachePolicy performs the following behavior: 1. If a cached response does not exist for the request, the URL loading system fetches the data from the originating source. 2. Otherwise, if the cached response does not indicate that it must be revalidated every time, and if the cached response is not stale (past its expiration date), the URL loading system returns the cached response. 3. If the cached response is stale or requires revalidation, the URL loading system makes a HEAD request to the originating source to see if the resource has changed. If so, the URL loading system fetches the data from the originating source. Otherwise, it returns the cached response.

官方文档说，

1. 缓存不存在，则直接请求。
2. 缓存存在，且缓存response头没有指明每次必须校验资源更新(revalidated这个词可能会产生误导，后文说)，且缓存没有过期，则系统会直接返回缓存，**不会发起请求**。
3. 如果缓存过期了或者要求每次必须校验资源更新，则会发起一个校验资源更新的请求，如果(服务器告诉客户端)资源有更新则使用服务器返回来的新数据，如果资源没有更新则使用本地缓存。

上面官方文档只是说了个大概的原理，具体指标和细节并没有说清楚。

1. 什么情况下会缓存数据？
2. 什么情况下每次都需要校验资源更新？
3. 缓存过期时间是多久？
4. 校验资源更新的过程是怎么样的？revalidated的指标是什么？

**实际上，WKWebView默认缓存策略完全遵循HTTP缓存协议，苹果并没有做额外的事情**，上面的流程图和文档描述只是简略描述了HTTP缓存协议的一个流程。**也就是说，你想弄清楚WKWebView默认缓存策略，你得弄清楚HTTP缓存协议**。

## 二、HTTP缓存协议

http缓存协议这个词是我自己造的哈，本节要讲的实际上就是HTTP协议中和缓存有关的请求头、响应头的作用和用法。

**客户端默认缓存行为实际上是由服务器控制的，客户端和服务器通过HTTP请求头和响应头中的缓存字段来交流，进而影响客户端的行为。** 下面就来介绍一下相关字段。

### 1. Pragma、Expires

在 http1.0 时代，给客户端设定缓存方式可通过这两个字段。

Pragma是一个通用头，它只有no-cache这一个值。

> 通用头：该字段可以用于请求头，也可用于响应头。（注意：同一个属性在请求头和响应头中意义可能不一样，例如下文中的Cache-Control）

作为请求头，表示不使用缓存，直接从源服务器获取资源，这是HTTP1.0的用法，HTTP1.1的用法是Cache-Control:no-cache。不过为了兼容HTTP1.0，一般Pragma:no-cache和Cache-Control:no-cache联用，如下。

```p
Cache-Control:no-cache
Pragme:no-cache
```

作为响应头，[RFC2616文档](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.32)说，Pragma : no-cache的行为并没有被定义，不能保证它的意义和Cache-Control:no-cache一致。

Expires，响应头，表示缓存过期的时刻，这个是服务器时间。例如`Expires: Fri, 11 Jun 2021 11:33:01 GMT`

Pragma、Expires的局限：响应报文中Expires所定义的缓存时间是相对服务器上的时间而言的，如果客户端上的时间跟服务器上的时间不一致（特别是用户修改了自己电脑的系统时间），那缓存时间可能就没啥意义了。

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9cddc3bd-d774-4b18-9fba-9a95815d0f1b/172020dc8b27cf0c.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9cddc3bd-d774-4b18-9fba-9a95815d0f1b/172020dc8b27cf0c.png)

### 2. Cache-Control

http1.1新增了 Cache-Control 来配置缓存信息，主要包括：能否缓存、缓存过期时间、是否每次校验等。

Cache-Control是通用头。

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/195e46bd-c293-4086-8e58-5e89ee21e6dd/172020dc8b321bc5.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/195e46bd-c293-4086-8e58-5e89ee21e6dd/172020dc8b321bc5.png)

下图是Cache-Control可选值表。你也可以查阅[HTTP官方文档](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9.1)14.9Cache-Control部分。

Cache-Control 允许自由组合可选值，用逗号分隔。`Cache-Control: max-age=3600, no-cache` 上面这句意思是，缓存过期时间是1小时，每次都必须向服务器进行资源更新校验。

下面介绍几个常用的可选值。

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/516d29de-fb51-4770-8fba-469ffc931894/172020dc8b490fb9.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/516d29de-fb51-4770-8fba-469ffc931894/172020dc8b490fb9.png)

### must-revalidate

文章开头我们提到了苹果的流程图可能会让人产生歧义，这里来解释一下坑在哪里。 苹果文档和流程图中有个判断，缓存存在，则需要判断是否需要每次都校验，用的是“revalidated”这个词。然后你看到Cache-Control可选值里面有个must-revalidate值，你是不是毫不犹豫地就向下面这样写了。`Cache-Control: max-age=3600, must-revalidate` 我就尝试设置一个过期时间，但是又希望每次都去校验更新，于是我像上面这样写，结果客户端仍然是用的缓存，根本没有网络请求发出去。 我很幸运地看到了这篇文章，[可能是最被误用的 HTTP 响应头之一 Cache-Control: must-revalidate](https://zhuanlan.zhihu.com/p/60357719)，强烈推荐阅读！

HTTP 规范是不允许客户端使用过期缓存的，除了一些特殊情况，比如校验请求发送失败的时候。而must-revalidate指令是用来排除这些特殊情况的。带有 must-revalidate 的缓存过期后，在任何情况下，都必须成功 revalidate 后才能使用，没有例外，即使校验请求发送失败也不可以使用过期的缓存。**也就是说，有个大前提是缓存过期了，如果缓存没过期客户端会直接使用缓存，并不会发起校验，显然不是字面上每次都校验更新的意思。must-revalidate 命名为 never-return-stale更合理。而真正每次都校验更新，应该用no-cache这个字段。** 把上面错误的写法改成下面这样就OK了：缓存有效期1小时，每次请求都校验更新。`Cache-Control: max-age=3600, no-cache`

### no-cache

作为请求头，告知中间服务器不使用缓存，向源服务器发起请求。 作为响应头，no-cache并不是字面上的不缓存，而是每次使用前都得先校验一下资源更新。

### no-store

作为响应头，带有no-store的响应不会被缓存到任意的磁盘或者内存里，no-store它才是真正的“no-cache”。

### max-age

作为请求头，max-age=0表示不管response怎么设置，在重新获取资源之前，先进行资源更新校验。 作为响应头，max-age=x表示，缓存有效期是x秒。

### Cache-Control的局限

很多时候，缓存过期了但是资源并没有修改，会发送多余的请求和数据；或者资源修改了缓存还没过期，客户端仍然在用缓存。Cache-Control无法及时和客户端同步。

### 3. Last-Modified、If-Modified-Since

为了弥补Cache-Control无法及时判断资源是否有更新的不足，有了Last-Modified、if-Modified-Since字段。

### Last-Modified

响应头，这次命名没有问题了，这个字段的值就是资源在服务器上最后修改时刻。例如`If-Modified-Since: Thu, 31 Mar 2016 07:07:52 GMT`

### If-Modified-Since

请求头，客户端通过该字段把Last-Modified的值回传给服务端；客户端带上这个字段表示这次请求是向服务端做校验资源更新校验。如果资源没有修改，则服务端返回304不返回数据，客户端用缓存；资源有修改则返回200和数据。 例如`If-Modified-Since: Thu, 31 Mar 2016 07:07:52 GMT`

### Last-Modified的启发式(heuristic)缓存

```p
HTTP/2 200
Date: Wed, 27 Mar 2019 22:00:00 GMT
Last-Modified: Wed, 27 Mar 2019 12:00:00 GMT

```

上面这个响应，没有显示地指明需要缓存，没有Cache-Control，也没有 Expires，只有Last-Modified修改时间，这种情况会产生启发式缓存。缓存时长=(date_value - last_modified_value) * 0.10 ，这是由 HTTP 规范推荐的算法，但规范中仅仅是推荐而已，并没有做强制要求。比如 Firefox 中就在这个算法的基础上还和 7 天时长取了一次最小值。 何如禁用由 Last-Modified响应头造成的启发式缓存：正确的做法是在响应头中加上 Cache-Control: no-cache。

### Last-Modified、If-Modified-Since的缺陷

无法识别内容是否发生实质性的变化，可能只是修改了文件但是内容没有变化；无法识别一秒内进行多次修改的情况。

### 4. ETag、If-None-Match

为了弥补Last-Modified的无法判断内容实质性变化的缺陷，于是有了ETag和If-None-Match字段，这对字段的用法和Last-Modified、If-Modified-Since相似，服务器在响应头中返回ETag字段，客户端在下次请求时在If-None-Match中回传ETag对应的值。

响应头，给资源计算得出一个唯一标志符（比如md5标志），加在响应头里一起返给客户端，例如`Etag: "5d8c72a5edda8d6a"`

### If-None-Match

请求头，客户端在下次请求时回传ETag值给服务器。`If-None-Match: "5d8c72a5edda8d6a""`

### 5. 优先级

上面这些缓存控制字段如果同时出现，他们的优先级如何呢？**优先级：Pragma > Cache-Control > Expires > Last-Modified > ETag** 这是我在iOS下测试的出来的结论，仅供参考。下面是测试的过程。

> 响应头没有任何缓存字段，每次启动都会发起请求，返回200。 第一次启动，响应头添加Pragma:no-cache和Cache-Control:max-age；第二次启动，会发起请求，返回304，说明Pragma生效了，Pragma > Cache-Control。 第一次启动，响应头没有过期时间，只有Last-Modified；第二次启动，使用缓存，没有发起请求，说明启发式缓存(上文中有提到)生效。 第一次启动，响应头没有过期时间，只有ETag；第二次启动，会发起请求，返回304，说明做了资源更新校验。 第一次启动，响应头没有过期时间，同时有ETag和Last-Modified；第二次启动，使用缓存，没有发起请求，启发式缓存生效，说明Last-Modified>ETag。

更多关于HTTP头部字段，可以查看[HTTP协议官方文档](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9.1)。 全英文的，看着头大？我还无意中发现了中文版的。**火狐浏览器F12调出控制台，请求头和响应头左边的问号（下图）是可以点的**！点击直接跳转到对用头字段的网页，真可谓“哪里不会点哪里，妈妈再也不用担心我的学习了！”哈哈哈哈——

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/336fffb2-2983-43bf-8d8a-3c48ccd1911e/17203de4f4890319.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/336fffb2-2983-43bf-8d8a-3c48ccd1911e/17203de4f4890319.png)

## 三、实战：浏览器的行为

介绍完上面的HTTP缓存协议，下面我们来实战一下，梳理下浏览器的整个交互过程，加深对上面各个字段的理解。 这里再次抛出苹果给的流程图看一眼，实际上浏览器（无论是PC还是移动端）的执行过程就是这个流程图。

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/49f55ad9-6919-4289-8e45-5ed5f060757e/172020dc96d8ec8f.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/49f55ad9-6919-4289-8e45-5ed5f060757e/172020dc96d8ec8f.png)

下面我们结合上面的流程图，以火狐浏览器、百度首页的css文件例，一步步进行说明。不同浏览器的行为可能不一致(刷新、强刷等操作浏览器会强行添加一些请求头，不同浏览器可能添加的不一样)，但是他们遵循的HTTP协议规则是一致的。

### 1.第一次请求（相当于iOS第一次启动）

第一次请求没有缓存，浏览器发出请求。 我们可以看到，返回的响应头中包含了Cache-Control、ETag、Expires、Last-Modified等多个缓存控制字段。浏览器进行缓存。

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e8260e19-c1b5-4a61-b310-acbfb25a4243/17203df28e1c8f80.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e8260e19-c1b5-4a61-b310-acbfb25a4243/17203df28e1c8f80.png)

### 2.在浏览器地址栏直接回车（相当于iOS第二次启动）

如下图可以看到，浏览器没有发送请求，而是直接使用了缓存数据。 浏览器的判断过程：首先判断是否有缓存，有缓存，是否需要校验资源更新，不需要（响应头没有Cache-Control:no-cache字段），然后判断缓存过期了吗，没过期（响应头Cache-Control:max-age=315360000），于是浏览器直接使用缓存，不进行请求。

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/00ca80fe-ae8b-4084-9a10-5df445f5c6f0/17203dfbc636c52c.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/00ca80fe-ae8b-4084-9a10-5df445f5c6f0/17203dfbc636c52c.png)

### 3.刷新页面（F5/点击工具栏中的刷新按钮/右键菜单重新加载）

从结果来看，浏览器仍然使用的是缓存。但是这次有发送资源更新校验的请求，服务端返回304，表示资源没有变动，浏览器使用缓存。 我们可以注意到，刷新页面，火狐浏览器(其它浏览器行为可能不一样)向请求头里强行添加了几个字段。

```p
Cache-Control:max-age=0
If-Modified-Since:Mon, 07 Nov 2016 07:51:11 GMT
If-None-Match: "352b-540b1498e39c0"
```

Cache-Control:max-age=0，表示不管上次的响应头设置的是什么，这次请求都会进行资源更新校验。 If-Modified-Since，回传资源最后修改时间给服务器校验 If-None-Match，回传ETag给服务器校验 浏览器的判断过程：缓存是否存在，存在，是否需要校验资源更新，需要（Cache-Control:max-age=0），发起资源校验请求，由于资源没有修改，服务器返回304，浏览器使用缓存数据。

### 4.谷歌浏览器强制刷新cmd+shift+R（因为火狐没这功能，所以这里换成谷歌浏览器测试）

结果：浏览器进行了请求，服务器返回200和数据。 我们注意到，谷歌浏览器在请求头中强行添加了两个字段。

```p
pragma: no-cache
cache-control: no-cache
```

cache-control: no-cache，在请求头中表示，（包括中间服务器）不要使用缓存，去源服务器请求资源。**注意：cache-control: no-cache作为请求头和响应头意义是不一样的。作为请求头表示不缓存，作为响应头表示每次都得去校验资源更新。** pragma: no-cache，和cache-control: no-cache是一个意思，只是为了兼容HTTP1.0。 浏览器的判断过程：有不使用缓存的标记（cache-control: no-cache），直接发起请求。

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9155dab2-5e1c-44eb-8ea2-afe9f1f88e78/172020dcf4a2ccc0.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9155dab2-5e1c-44eb-8ea2-afe9f1f88e78/172020dcf4a2ccc0.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2392299e-a899-4061-87d5-545eea25141c/17203e114a20327e.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2392299e-a899-4061-87d5-545eea25141c/17203e114a20327e.png)

## 四、WKWebView默认缓存策略总结

### （一）回答文章开头的几个问题

### 1. 什么情况下会缓存数据

第一次启动的时候，如果响应头中不包含任何缓存控制字段（Expires、Cache-Control:max-age、Last-Modified等），那么不会缓存（仍然可能会有物理缓存，只是不使用），下次直接发起请求。如果响应头包含了缓存控制字段，大多数情况下这次数据会被缓存，下次启动的时候执行缓存逻辑判断。

### 2. 什么情况下**每次**都需要校验资源更新

a. 如果响应头中包含Cache-Control:no-cache 或 Pragma:no-cache。 b. 如果请求头中包含了Cache-Control:max-age=0，这个结论是对的，但是WKWebView的默认策略不会出现这种情况。 c. 响应头中缓存控制字段只有ETag字段，没有过期时间和修改时间。

### 3. 缓存过期时间是多久

a. 响应头中Cache-Control:max-age=3600，表示缓存1小时(3600/60/60)，单位秒。 b. 响应头中Expires的值表示过期时刻（服务器时间）。 c. 响应头中，如果没有上述两个字段，但有Last-Modified字段，则触发启发式缓存，缓存时间=(date_value - last_modified_value) * 0.1。 优先级 Cache-Control:max-age > Expires > Last-Modified。

### 4. 校验资源更新的过程是怎么样的？revalidated的指标是什么

revalidated的指标有两个：Last-Modified最后修改时刻、ETag资源唯一标识。 服务器返回数据时会在响应头中返回上面两个指标（有可能只有1个，也可以2个都有），客户端再次发起请求时会把这两个指标回传给服务器。 If-Modified-Since: Last-Modified的值 If-None-Match: ETag的值 服务器进行比对，如果客户端的资源是最新的，则返回304，客户端使用缓存数据；如果服务器资源更新了，则返回200和新数据。

### （二）WKWebView默认缓存策略流程总结

对照文章开头的流程图，WKWebView默认缓存策略流程总结如下：

1. 是否有缓存，没有则直接发起请求。有则进行下一步。
2. 是否每次都得进行资源更新校验（响应头是否有Cache-Control:no-cache或Pragma:no-cache字段），不需要则进入3，需要则进入4。
3. 缓存是否过期（响应头，Cache-Control:max-age、Expires、Last-Modified启发式缓存），没过期则使用缓存，不发起请求。过期了则进入4。
4. 客户端发起资源更新校验请求（请求头，If-Modified-Since: Last-Modified值、If-None-Match: ETag值），如果资源没有更新，服务器返回304，客户端使用缓存；如果资源有更新，服务器返回200和资源。

## 五、解决方案：数据更新后仍然有缓存的问题

弄清楚了原理，回到文章开头的问题，H5资源更新了，但是iOS有缓存没有同步还是显示的原来的数据。那么怎么解决呢？ App端是做不了什么的，这个问题需要后台处理。

### 方案一：响应头，添加Cache-Control:no-cache

经过我的调试发现，服务器返回资源的响应头是`Cache-Contol: max-age=36000000` 问题的原因在于服务器响应头的缓存字段配置不合理，没有配置资源更新校验字段，而缓存过期时间又过长，因此，即使服务器资源更新了客户端也不会请求新的资源，而是直接使用“没有过期”的资源。

我们做出如下修改，在资源的响应头中添加no-cache字段，这样每次浏览器都会先去校验资源更新，就解决了这个问题。`Cache-Control:no-cache`

### 方案二：资源链接加后缀（md5、版本号等）

```js
<script src="test.js?ver=113"></script>
https://hm.baidu.com/hm.js?e23800c454aa573c0ccb16b52665ac26
http://tb1.bdstatic.com/tb/_/tbean_safe_ajax_94e7ca2.js
http://img1.gtimg.com/ninja/2/2016/04/ninja145972803357449.jpg

```

可以在资源文件后面加上版本号，每次更新资源的时候变更版本号；还可以在URL后面加上了md5参数，甚至还可以将md5值作为文件名的一部分。 采用上述方法，你可以把缓存时间设置的特别长，那么在文件没有变动的时候，浏览器直接使用缓存文件；而在文件有变化的时候，由于文件版本号的变更，或md5变化导致文件名变化，请求的url变了，浏览器会当做新的资源去处理，一定会发起请求，所以不存在更新后仍然有缓存的情况。通过这样的处理，增长了静态资源，特别是图片资源的缓存时间，避免该资源很快过期，客户端频繁向服务端发起资源请求，服务器再返回304响应的情况（有Last-Modified/Etag）。

## 六、补充：iOS原生请求默认策略的一些问题

### 1. iOS原生请求默认策略也遵循上面的规则吗

——是的。 NSURLRequest的默认缓存策略是NSURLRequestUseProtocolCachePolicy，完全遵循上文讲得HTTP缓存协议。看下面的例子。

```oc
- (void)requestData
{
    NSLog(@"开始请求");
    NSString *url = @"http://www.4399.com/jss/lx6.js";
    NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:url]];
    NSURLSessionDataTask *task = [[NSURLSession sharedSession] dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        if (!error) {
            NSLog(@"%@",response);
        }
    }];
    [task resume];
}

```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/91dedb76-77a6-4ff4-9402-7516d9b01afb/172020dd266adb5d.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/91dedb76-77a6-4ff4-9402-7516d9b01afb/172020dd266adb5d.png)

第一次请求时通过抓包工具看到，响应头设置了比较长的缓存时间。按照上文的讲述的，在缓存没有过期的情况下，下次请求会直接返回缓存数据，不在请求。 经过测试，再次请求时抓包工具显示确实没有请求发出。同时completionHandler回调，code返回200，data返回数据。甚至，你可以把网断了，仍然会有上述回调，code200，data返回数据。印着了上述结论。

### 2. iOS客户端需要自己处理 "304 Not Modified" 响应吗

不需要。 还是上面的例子，我们先把模拟器上的App删了（清除缓存），重新run。这次我通过抓包工具对这个请求打断点，在第一次请求返回时在响应头添加no-cache字段，来测试下收到304响应时客户端completionHandler回调的情况。加入no-cache字段后，第二次请求效果如下：

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bbb09792-adff-4e1c-a227-b07467e05077/172020dd33e69a86.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bbb09792-adff-4e1c-a227-b07467e05077/172020dd33e69a86.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9ff62674-27bb-4971-8aa7-e25f023c02c5/172020dd3e8615fd.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9ff62674-27bb-4971-8aa7-e25f023c02c5/172020dd3e8615fd.png)

大家可以看到， 请求头，**自动（注意，这是系统自己实现的，并不需要客户端手动添加，这也进一步证明iOS原生请求也是遵循Http缓存协议的**）带上了if-None-Match和if-Modified-Since这两个字段。那是因为第一次响应头中我们添加了no-cache字段，表示下次请求需要校验资源更新。 响应头，服务器返回了304 Not Modified。 下面来看看completionHandler回调情况：

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6601655e-8988-467c-bf53-c014fb2ac29b/172020dd413100ac.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6601655e-8988-467c-bf53-c014fb2ac29b/172020dd413100ac.png)

从日志中我们可以看出completionHandler回调返回的code仍然是200。

### 苹果系统内部对304 Not Modified响应做了特殊处理

- code字段，固定返回200
- data字段，因为服务端返回的304报文是不带data数据字段的，但是苹果又得把data通过completionHandler回调给客户端，苹果会去缓存中取data数据，返回的data字段和第一次响应的data是同一个。
- response字段，返回的是第二次请求304的响应头，而不是第一次请求缓存的响应头。可以通过下图佐证，第一次和第二次回调的响应头不一致。

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/762b47ac-b5ac-4a91-9f47-3e1f5b7066aa/17203e1bde9af09f.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/762b47ac-b5ac-4a91-9f47-3e1f5b7066aa/17203e1bde9af09f.png)

**综上，苹果内部帮我们处理了"304 Not Modified"响应。对客户端来说，你只需要知道返回200就是没有异常，拿着data用就行了。至于，数据来自缓存还是来自服务器，缓存有没有过期，需不需要校验资源更新等，都交给苹果吧。**

### 3. code都返回200，那我怎么知道返回的是缓存数据还是服务器数据呢**

苹果并没有提供相关的API，不过我们可以间接的去判断。 请求前先去取缓存NSCachedURLResponse，NSCachedURLResponse对象有个response属性，在completionHandler回调时去比对缓存的response和返回的response是否相同。系统也没有提供比对NSURLResponse的方法，这里我们比对NSHTTPURLResponse的allHeaderFields属性。

```oc
- (void)requestData
{
    NSLog(@"开始请求");
    NSString *url = @"http://www.4399.com/jss/lx6.js";
    NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:url]];
    NSCachedURLResponse *cachedURLResponse = [[NSURLCache sharedURLCache] cachedResponseForRequest:request];
    NSURLResponse *cacheResponse = cachedURLResponse.response;
    NSURLSessionDataTask *task = [[NSURLSession sharedSession] dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        if ([[cacheResponse valueForKey:@"allHeaderFields"] isEqual:[response valueForKey:@"allHeaderFields"]]) {
            //响应头相同,是缓存数据
            NSLog(@"allHeaderFields 相同");
        }
    }];
    [task resume];
}

```

实际上，后台把缓存字段配置好后，客户端不需要关心返回的数据是否来自缓存，好像没有这样的应用场景。

**如果觉得这篇文章对你有帮助，请点个赞吧。如果有疑问可以关注我的公众號给我留言。转载请注明出处，谢谢！**

参考链接：[WKWebView的缓存问题](https://xiaozhuanlan.com/topic/8524137069) [](http://www.cnblogs.com/lolDragon/p/6774509.html)

[iOS webview加载时序和缓存问题总结](http://www.cnblogs.com/lolDragon/p/6774509.html)

[WKWebView缓存问题 - 图片资源](https://www.jianshu.com/p/a3911d4808ba)

[对NSURLRequestUseProtocolCachePolicy的理解](https://www.jianshu.com/p/de5254cb1e40)

[苹果官网文档:NSURLRequestUseProtocolCachePolicy](https://developer.apple.com/documentation/foundation/nsurlrequestcachepolicy/nsurlrequestuseprotocolcachepolicy?language=occ)

[HTTP缓存控制小结](https://imweb.io/topic/5795dcb6fb312541492eda8c)

[HTTP/1.1官方协议RFC2616](https://www.w3.org/Protocols/rfc2616/rfc2616.html)

[可能是最被误用的 HTTP 响应头之一 Cache-Control: must-revalidate](https://zhuanlan.zhihu.com/p/60357719)
