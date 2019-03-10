---
layout: post
title: 已有Android项目接入cocos2d-x
categories: cocos2d-x
date: 2019-03-10
---

近日来一直在学习Cocos2d-x引擎，几乎就是从入门到放弃。不得不说Cocos2d因为是开源免费的东西，导致文档其实并不完善，多数问题还是要靠论坛来解决，但论坛这种不知何时会有回复的东西，也是不太靠谱的。

以下记录的是我从0开始，学习如何将cocos2d-x引擎接入现有Android项目的过程，一些官方文档中已有说明的我就一笔带过，主要记录踩到的坑。

### 目标

在不破坏原有Android项目结构的基础上，以so的形式接入cocos2d-x引擎。如果是新的Android项目，请使用cocos命令创建Android项目即可。

### 准备工作

1. cocos2d-x源码
2. cocos creator编辑器（这个自带引擎，也可以不用去下cocos2d-x源码，需要知道的是`这两个引擎源码可能不一样`）
3. 参照 [官方文档](https://docs.cocos.com/cocos2d-x/manual/zh/installation/) 搭建环境
4. 需要接入引擎的常规Android项目

### 大体思想

首先说明下为什么接入引擎我会需要cocos creator。本来是打算通过手动精简cocos2d-x源码后，编译源码生成so的方式来接入。但Android项目本身还需要额外的一些东西JNI，加上在精简源码的过程中，js-binding一直报错，导致我进度卡住无法继续，因此想到通过cocos creator的方式进行引擎的精简。

言归正传，这次的方法的流程就是：

1. 在cocos creator的项目设置中精简不需要的模块。
2. 在构建发布中选择Android平台进行构建(link和default的方式都可以)，然后编译（开了调试模式，引擎会打log，关了包会适当小一点）。
3. 编译完成后，打开build目录，分析生成的Android项目的结构，提取需要的部分移植到需要接入引擎的Android项目中。

### 创建cocos2d-x项目

没啥好说的，直接用cocos creator进行创建，建一个空项目就行。

### 精简模块

在cocos creator的菜单栏的项目中，找到项目设置，将不需要的模块取消勾选即可。

### 生成Android项目

精简完模块后，就可以使用构建和编译来生成包含引擎的Android项目。

### 分析Android项目结构

生成好Android项目后，打开项目的build文件夹。可以看到这是个cocos项目中包了个Android项目，我们实际需要的是Android项目中包个cocos项目。因此接下来我们开始分析这个项目结构。

未完待续

