---
layout: post
title: Untiy从入门到自闭(一)——VS环境搭建
categories: unity
tags: [unity, vscode]
date: 2020-04-25
---

# Unity+VsCode开发环境搭建

### 安装Unity和Vscode

前往官网下载Untiy和VsCode

### 安装VsCode插件

<img src="https://1030310877.github.io/1030310877.github.io/images/1.png" style="zoom: 67%;" />

### 修改Untiy配置

新建一个Untiy项目，打开菜单栏Edit——Preferences——External Tools，修改External Script Editor为Visual Studio Code，勾选Generate all .csproj files选项。

<img src="https://1030310877.github.io/1030310877.github.io/images/2.png" style="zoom:50%;" />

### 使用VsCode进行开发

做了以上配置后，也就能使用Vscode进行C#和Unity的开发了。但需要注意的是，一定要从Untiy项目根目录，以C#项目形式打开Script文件，如果直接从Unity中双击Script打开，那只会以单个C#文件形式打开，这样就不会有C#和Untiy相关的补全提示。如果已经安装要求进行了打开，仍然没有自动补全提示，则检查.Net版本是否正确。

### 安装和检查.Net版本是否正确

使用Vscode从根目录打开Unity项目，打开`Assembly-CSharp.csproj`文件，搜索`TargetFrameworkVersion`，看到对应的.Net版本号。只有你本机安装的.Net版本和这个对应上才行，如果没有，则去官网下载对应版本的.Net，安装完成后，重新打开VsCode即可。
