---
title: nlclslist - WBBlades
date: 2022-01-13 14:49:06
tags:
	- nlclslist
	- WBBlades
categories:
	- 底层实现
index_img: /images/AppSize/wallhaven-72zwm3.png
---

以 WBBlades 为源码解读二进制可执行文件 Mach-O
# 源码
下面是 WBBlades 对 `nlclslist` 部分解析的源代码。
```objectivec
+ (void)readNLClsList:(section_64)nlclsList set:(NSMutableSet *)classrefSet fileData:(NSData *)fileData {
    //nlclslist
    NSRange range = NSMakeRange(nlclsList.offset, 0);
    unsigned long long max = [fileData length];
     for (int i = 0; i < nlclsList.size / 8; i++) { // 每个类的地址为 8 个字节，因为是类地址列表，所以连续读取
         @autoreleasepool {
           
             unsigned long long classAddress;
             NSData *data = [WBBladesTool readBytes:range length:8 fromFile:fileData];
             [data getBytes:&classAddress range:NSMakeRange(0, 8)];
             classAddress = [WBBladesTool getOffsetFromVmAddress:classAddress fileData:fileData];
             //method name 150 bytes maximum
             if (classAddress > 0 && classAddress < max) {

                 class64 targetClass;
                 [fileData getBytes:&targetClass range:NSMakeRange(classAddress,sizeof(class64))];
                 
                 class64Info targetClassInfo = {0};
                 unsigned long long targetClassInfoOffset = [WBBladesTool getOffsetFromVmAddress:targetClass.data fileData:fileData];
                 targetClassInfoOffset = (targetClassInfoOffset / 8) * 8; // 对齐？
                 NSRange targetClassInfoRange = NSMakeRange(targetClassInfoOffset, 0);
                 data = [WBBladesTool readBytes:targetClassInfoRange length:sizeof(class64Info) fromFile:fileData];
                 [data getBytes:&targetClassInfo length:sizeof(class64Info)];
                 unsigned long long classNameOffset = [WBBladesTool getOffsetFromVmAddress:targetClassInfo.name fileData:fileData];
                 
                 //class name 50 bytes maximum
                 uint8_t *buffer = (uint8_t *)malloc(CLASSNAME_MAX_LEN + 1);
                 buffer[CLASSNAME_MAX_LEN] = '\0';
                 [fileData getBytes:buffer range:NSMakeRange(classNameOffset, CLASSNAME_MAX_LEN)];
                 NSString *className = NSSTRING(buffer);
                 free(buffer);
                 if (className){
                     [classrefSet addObject:className];
                 }
             }
         }
     }
}
```

方法中的 `nlclslist` 是 section 数据，这个 section 属于 `__Data` 段。
classrefSet 就是一个初始化空的集合，用于之后的比较。fileData 就是整个二进制文件的数据。

## 拆解

接下来依次解析：

```objectivec
NSRange range = NSMakeRange(nlclsList.offset, 0);
// uint32_t	offset;		/* file offset of this section */
unsigned long long max = [fileData length];
```

range 由 section 的 offset 得来，代表的是 section 在二进制文件中的偏移量。

```objectivec
for (int i = 0; i < nlclsList.size / 8; i++) {
}
```

这里的遍历比较条件里， `nlclsList.size / 8` 。首先 `nlclsList.size` 代表的是所有类的地址的集合的大小，而一个地址的大小是 8 位。

其实下面的初始化也解答了这个问题

```objectivec
unsigned long long classAddress; // 八个字节
NSData *data = [WBBladesTool readBytes:range length:8 fromFile:fileData]; // 根据偏移量获取 8 字节的数据
[data getBytes:&classAddress range:NSMakeRange(0, 8)]; // 8 字节二进制数据映射到数据结构
```

```objectivec
classAddress = [WBBladesTool getOffsetFromVmAddress:classAddress fileData:fileData];
```

（VMAddress 地址偏移量计算单独记录）

通过类地址，找到具体类名的简单流程如下：

`classAddress` → `class` → `class.data` → `classInfoOffset` → `classInfo` → `classInfo.name`→ `classNameOffset`

### Class

```objectivec
class64 targetClass;
[fileData getBytes:&targetClass range:NSMakeRange(classAddress,sizeof(class64))];
```

第一步，根据类地址，获取到类数据。（这里的类地址，更准确的描述是在二进制文件中的偏移量）

### ClassInfo

```objectivec
class64Info targetClassInfo = {0};
unsigned long long targetClassInfoOffset = [WBBladesTool getOffsetFromVmAddress:targetClass.data fileData:fileData];
targetClassInfoOffset = (targetClassInfoOffset / 8) * 8; // 对齐？
NSRange targetClassInfoRange = NSMakeRange(targetClassInfoOffset, 0);
data = [WBBladesTool readBytes:targetClassInfoRange length:sizeof(class64Info) fromFile:fileData];
[data getBytes:&targetClassInfo length:sizeof(class64Info)];
```

### ClassName

```objectivec
unsigned long long classNameOffset = [WBBladesTool getOffsetFromVmAddress:targetClassInfo.name fileData:fileData];
//class name 50 bytes maximum
uint8_t *buffer = (uint8_t *)malloc(CLASSNAME_MAX_LEN + 1);
buffer[CLASSNAME_MAX_LEN] = '\0';
[fileData getBytes:buffer range:NSMakeRange(classNameOffset, CLASSNAME_MAX_LEN)];
NSString *className = NSSTRING(buffer);
free(buffer);
if (className){
	[classrefSet addObject:className];
}
```

![list](/images/AppSize/nclslist-progress.png)
