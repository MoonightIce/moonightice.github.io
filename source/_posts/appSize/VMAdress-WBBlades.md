---
title: VMAdress - WBBlades
date: 2022-01-13 15:20:33
tags:
categories:
	- 底层实现
index_img: /images/AppSize/wallhaven-6o826l.jpeg
---
```objectivec
+ (unsigned long long)getOffsetFromVmAddress:(unsigned long long )address fileData:(NSData *)fileData{
    
    mach_header_64 mhHeader;
    [fileData getBytes:&mhHeader range:NSMakeRange(0, sizeof(mach_header_64))];
    
    unsigned long long currentLcLocation = sizeof(mach_header_64);
    for (int i = 0; i < mhHeader.ncmds; i++) {
        load_command* cmd = (load_command *)malloc(sizeof(load_command));
        [fileData getBytes:cmd range:NSMakeRange(currentLcLocation, sizeof(load_command))];
        
        if (cmd->cmd == LC_SEGMENT_64) {//LC_SEGMENT_64:(section header....)
            segment_command_64 segmentCommand;
            [fileData getBytes:&segmentCommand range:NSMakeRange(currentLcLocation, sizeof(segment_command_64))];
            if (address >= segmentCommand.vmaddr && address <= segmentCommand.vmaddr + segmentCommand.vmsize) {
                free(cmd);
                // address = 4335337760 == 0000 0001 0268 0120
                // 上一个segment地址（TEXT）vmAddress == 0000 0001 0000 0000
                return address - (segmentCommand.vmaddr - segmentCommand.fileoff);
            }
        }
        currentLcLocation += cmd->cmdsize;
        free(cmd);
    }
    
    return address;
}
```

通过 section 里集合计算的地址其实是 VmAddress 内存中的地址，那么需要通过偏移量计算，映射到二进制文件里的位置。

在一开始的 Mach-O 文件里，除了 Mach Header 结构，剩下的其实就是 segment_command_64 结构了（segment_command_64 包含了 load command）。

```objectivec
segmentCommand = {
  cmd = 25
  cmdsize = 72
  segname = "__PAGEZERO"
  vmaddr = 0 //表示__PAGEZERO段加载到虚拟内存中的地址，从0x000000000开始
  vmsize = 4294967296 // 表示__PAGEZERO段在虚拟内存中所占据的空间大小 0000 0001 0000 0000
  fileoff = 0 //表示当前__PAGEZERO段在Mach-O文件中的位置。
  filesize = 0// 表示__PAGEZERO段在Mach-O文件中的大小，此处File Size为0表示在Mach-O文件中
							//并没有__PAGEZERO段，在Mach-O文件被加载进虚拟内存中，才会附加上__PAGEZERO段。
  maxprot = 0//表示当前段在虚拟内存中所需要的最高内存保护
  initprot = 0//表示当前段的初始内存保护
  nsects = 0//表示当前段中所包含的Section的数量
  flags = 0
}

segmentCommand = {
  cmd = 25
  cmdsize = 1912
  segname = "__TEXT"
  vmaddr = 4294967296
  vmsize = 34439168
  fileoff = 0
  filesize = 34439168
  maxprot = 5
  initprot = 5
  nsects = 23
  flags = 0
}

segmentCommand = {
  cmd = 25
  cmdsize = 1912
  segname = "__DATA"
  vmaddr =  4329406464
  vmsize = 6717440
  fileoff = 34439168
  filesize = 6635520
  maxprot = 3
  initprot = 3
  nsects = 23
  flags = 0
}
```

![segment](/images/AppSize/segment-vmadress.png)

这里特意复制了连续三个 segmentCommand 的数据，其中的重点字段是

- `vmaddr`
- `vmsize`
- `fileoff`

判断条件里是指地址所在的  segmentCommand 段

```objectivec
if (address >= segmentCommand.vmaddr && address <= segmentCommand.vmaddr + segmentCommand.vmsize) 
```

计算的后半部分，这里拿到的是在内存中，加载的二进制文件 segmentCommand 段的位置

```objectivec
address - (segmentCommand.vmaddr - segmentCommand.fileoff)
// =
address - segmentCommand.vmaddr + segmentCommand.fileoff
```

前半部分就是 section 里的遍历计算得到的地址，相对于当前  segmentCommand 的偏移量。然后加上二进制文件里的当前 segmentCommand 的偏移量，做为起始位置，最后得到的就是在二进制文件里的具体位置，然后就可以再拿到对应的数据。

所以倒退回去再看，括号里计算的是 segmentCommand 段从虚拟内存的起始位置到二进制文件的起始位置的偏移量。那么最后得到的也就是二进制里的位置