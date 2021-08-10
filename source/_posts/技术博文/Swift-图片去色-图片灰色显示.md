---
title: Swift 图片去色 图片灰色显示
date: 2019-09-09 16:53:25
tags:
  - 图片去色
categories:
  - 开发
index_img: /images/CoverImage/wallhaven-wydk1r.jpg
---

参考地址：
实现方法一：[Swift 实现更改图片的颜色 ](https://onevcat.com/2013/04/using-blending-in-ios/)
实现方法二：[iOS 中使用 blend 改变图片颜色](https://www.jianshu.com/p/f0340cfc0833)

当前网上搜到的方法几乎都是这两种。但是不知道在我的代码中不能完美完美实现。

- 第一种方法实现，发现是在原来的颜色上盖上一种半透明的灰色遮罩。原来的颜色还是能展示。

后者参数参照喵神的使用 overlay 保留图片的灰度

```
let grayImage = image?.imageWithTintColor(tintColor: .black, blendMode: .overlay)
```

方法实现

```
    func imageWithTintColor(tintColor:UIColor, blendMode:CGBlendMode) -> UIImage? {
        UIGraphicsBeginImageContextWithOptions(self.size, false, 0.0)
        tintColor.setFill()
        let bounds = CGRect(x: 0, y: 0, width: self.size.width, height: self.size.height)
        UIRectFill(bounds)
        self.draw(in: bounds, blendMode: blendMode, alpha: 1.0)

        if blendMode != .destinationIn {
            self.draw(in: bounds, blendMode: .destinationIn, alpha: 1.0)
        }

        let tintedImage = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        return tintedImage
    }
```

当使用的参数为 `.gray`
![灰色不明显](https://img-blog.csdnimg.cn/20190826161623849.jpg)
当使用的参数是`.black`
![全黑](https://img-blog.csdnimg.cn/20190826161711140.jpg)
和需求不一致，变成了全黑没有保留原来图片的灰度。（毕竟时间过去太久，如果有朋友知道问题在哪，麻烦告知一下，谢谢）

- 第二种方法实现，发现原来透明的背景变成了黑色，需要再做一步扣除黑色背景的操作。

```
let grayImage = image?.grayImage()?.transparentColor(colorMasking: [0, 32, 0, 32, 0, 32])
```

图片去色

```
 func grayImage() -> UIImage?
    {
        UIGraphicsBeginImageContext(self.size)
        let colorSpace = CGColorSpaceCreateDeviceGray()
        let context = CGContext(data: nil , width: Int(self.size.width), height: Int(self.size.height),bitsPerComponent: 8, bytesPerRow: 0, space: colorSpace, bitmapInfo: CGImageAlphaInfo.none.rawValue)
        context?.draw(self.cgImage!, in: CGRect.init(x: 0, y: 0, width: self.size.width, height: self.size.height))
        let cgImage = context!.makeImage()
        let grayImage = UIImage(cgImage: cgImage!, scale: self.scale, orientation: self.imageOrientation)
        return grayImage

    }
```

当前图片展示是这种形式，多出来了一个黑色背景
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190826162725493.jpg)

扣除黑色背景（[Swift - 去处图片的白色、黑色背景（使 UIImage 背景透明）](https://www.hangge.com/blog/cache/detail_1496.html)）

```
    func transparentColor(colorMasking:[CGFloat]) -> UIImage? {
        if let rawImageRef = self.cgImage {
            UIGraphicsBeginImageContext(self.size)
            if let maskedImageRef = rawImageRef.copy(maskingColorComponents: colorMasking) {
                let context: CGContext = UIGraphicsGetCurrentContext()!
                context.translateBy(x: 0.0, y: self.size.height)
                context.scaleBy(x: 1.0, y: -1.0)
                context.draw(maskedImageRef, in: CGRect(x:0, y:0, width:self.size.width, height:self.size.height))
                let result = UIGraphicsGetImageFromCurrentImageContext()
                UIGraphicsEndImageContext()
                return result
            }
        }
        return nil
    }
```

这样拿到的就是我想要的，图片去色且保留了图片的灰度展示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190826162659266.jpg)
