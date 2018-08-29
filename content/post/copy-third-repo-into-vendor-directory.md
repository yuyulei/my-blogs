+++
title = "引入第三方库到 vendor 目录时遇到的问题"
date = 2018-08-24T16:35:03+08:00
tags = ["git"]
draft = false
+++

项目中依赖了一些第三方的代码库，本地编译的时候没有问题，因为可以通过本地的 GOPATH 定位到第三方代码库。但是希望这个项目能够独立存在（不依赖本地的第三方代码），因此在项目中建立了一个 vendor 的目录，用来存放那些第三方代码。

<!--more-->

# 引入第三方库到 vendor 目录时遇到的问题

项目中依赖了一些第三方的代码库，本地编译的时候没有问题，因为可以通过本地的 GOPATH 定位到第三方代码库。但是希望这个项目能够独立存在（不依赖本地的第三方代码），因此在项目中建立了一个 vendor 的目录，用来存放那些第三方代码。


### 问题

直接 copy 第三方代码到 vendor 下的时候发现，这个代码库都是以 submodule 的形式存在。


### 后果

代码提交到线上进行 CI，报错：依赖某个第三方库的代码没有在各种 GOPATH 目录下找到。但实际上，在 vendor 目录下是有的，只是那个代码库是以 submodule 的形式存在。

### 解决

将那些以 submodule 形式存在的代码库，转成普通用来引用的代码。

**参考:** 
https://stackoverflow.com/questions/22264498/git-thinks-some-copied-code-is-a-submodule
https://stackoverflow.com/questions/1759587/un-submodule-a-git-submodule

当你 copy 代码库到项目中时候，项目中的 git 有时候会认为 这个是 submodule，即使这个代码库中没有 .gitmodule。

解决的办法比较简单：

* git rm --cached Src/Foo/FooBar
* rm -rf  Src/Foo/FooBar/.git 
* git add Src/Foo/FooBar

然后重新上传代码即可看到效果。

