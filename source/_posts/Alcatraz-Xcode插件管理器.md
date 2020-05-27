---
title: Xcode
date: 2018-09-03 14:53:52
tags:
  - xcode
categories:
  - 工具
index_img: /images/CoverImage/wallhaven-73kq69.jpg
---

# 先前

> 使用带有插件的 Xcode，容易卡死和 crash，所以使用自签名 xcode

1. 先复制备份一个 Xcode，不带任何插件（复制版 xcode 使用时模拟器提示连不上网络，无法启动，需要改动名字）
2. 重新创建证书
   打开电脑里的钥匙串 -> 菜单栏钥匙串访问 -> 证书助理 -> 创建证书... -> 名称 XcodeSinger，自签名根证书身份类型，代码签名证书类型
3. 命令行
   先跟名字，后跟的 app 路径

   ```示例
   sudo codesign -f -s XcodeSigner /Applications/Xcode.app
   ```

4. 重启，选择 loadBuddle
5. 失效插件解决

   ```示例
   find ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins -name Info.plist -maxdepth 3 | xargs -I{} defaults write {} DVTPlugInCompatibilityUUIDs -array-add `defaults read /Applications/Xcode.app/Contents/Info DVTPlugInCompatibilityUUID`
   ```

6. 不支持的插件或不兼容插件需要删除

<!-- more -->

# 简单安装 Alcatraz --- 插件管理器

1. 简介
   Alcatraz 是一个能帮你管理 Xcode 插件丶模版及颜色配置的工具.它可以直接集成在 Xcode 的图形界面中,让你感觉就像在使用 Xcode 自带的功能一样.
2. 安装和删除
   使用如下的终端来安装 Alcatraz:

   ```示例
   curl -fsSL https://raw.github.com/supermarin/Alcatraz/master/Scripts/install.sh | sh
   ```

   如果你不想使用 Alcatraz 了,可以使用如下命令来删除:

   ```示例
   rm -rf ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins/Alcatraz.xcplugin
   ```

   删除所有通过 Alcatraz 安装的安装包

   ```示例
   rm -rf ~/Library/Application\ Support/Alcatraz/
   ```

3. 重启 Xcode，弹出‘Load Buddle’选项提示框

# 真机手机系统 beat 版测试

1. 在 xcode 中增加测试版系统调试包，这是调试包位置

   ```示例
   /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport
   ```

2. 最新调试包下载，（iOS12）

[百度网盘下载地址](https://pan.baidu.com/s/1nSn9dkzuYHBig6mzLymbZA)
