---
title: 利用 Sourcetree 来管理 SVN 代码仓库
date: 2019-09-09 16:45:18
tags:
  - 代码管理
categories:
  - 基础
index_img: /images/CoverImage/wallhaven-839voo.jpg
---

git 控制本地代码版本

<!-- more_text -->

## 踩坑

```错误
Can't locate Git/SVN.pm in @INC (you may need to install the Git::SVN module) (@INC contains: /usr/local/git/share/perl5
/Applications/Sourcetree.app/Contents/Resources/git_local/lib/perl5/site_perl /Library/Perl/5.18/darwin-thread-multi-2level /Library/Perl/5.18
/Network/Library/Perl/5.18/darwin-thread-multi-2level /Network/Library/Perl/5.18
/Library/Perl/Updates/5.18.4 /System/Library/Perl/5.18/darwin-thread-multi-2level
/System/Library/Perl/5.18 /System/Library/Perl/Extras/5.18/darwin-thread-multi-2level /System/Library/Perl/Extras/5.18 .) at
/Applications/Sourcetree.app/Contents/Resources/git_local/libexec/git-core/git-svn line 22. BEGIN failed--compilation aborted at
/Applications/Sourcetree.app/Contents/Resources/git_local/libexec/git-core/git-svn line 22.
```

大多数解决方案是其中的两步或者三步，但是都未解决我的问题
找到一个完整的解决方案 [git-svn 在 mac 下遇到的问题](https://github.wangkaimin.com/2018/09/05/git-svn-mac-error.html)

```命令
sudo cpan Git::SVN
sudo ln -s /Applications/Xcode.app/Contents/Developer/Library/Perl/5.18/darwin-thread-multi-2level/SVN/ /Library/Perl/5.18/SVN
sudo mkdir /Library/Perl/5.18/auto
sudo ln -s /Applications/Xcode.app/Contents/Developer/Library/Perl/5.18/darwin-thread-multi-2level/auto/SVN/ /Library/Perl/5.18/auto/SVN
```

初始化 git-svn

```
git svn init svn://username@host/filepath
```

init 后面跟的是 svn 的地址。我用的就是 https 开头

拉取代码

```
git svn fetch
```

后面可以跟参数拉取特定版本的代码，不添加则是最新。例如

```
git svn fetch -r 8333:HEAD
```

回车之后输入电脑密码。第二次回车输入 svn 密码

此时对应的代码已经下载到本地。有源代码，也有版本控制信息

在 Sourcetree 中选择添加已经存在的本地仓库

这样基本就完成了利用 Sourcetree 管理 svn 代码仓库

```
git status // 查看当前代码修改状态，是否有修改代码为提交暂存区

git svn dcommit // 提交到远程 svn 仓库
```
