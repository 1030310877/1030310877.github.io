---
layout: post
title: VsCode编译jar包
categories: vscode
date: 2019-04-12
---

不知道为啥网上查询VSCode编译jar包，几乎查不到任何的内容，所以只能自己研究。

推荐两种方式进行jar包的编译：1. Maven。2.自己编译。

## Maven

vscode+java+maven的方案网上很多，这里不多做描述。需要注意的坑点在于：maven不能直接引用依赖本地jar包，必须为本地jar包建立本地仓库后进行依赖。

## 普通Java项目

### 创建Java项目

VsCode默认支持Eclipse的Java项目，或直接创建Java项目：

![image-20190412150842017](http://1030310877.github.io/images/2019-04-12/image-20190412150842017.png)

创建完成后，根目录下便会有`.classpath`以及`.project`文件。

`src`中存放java源文件。

`bin`是编译输出目录。

### 引入第三方jar包

在根目录下建立libs文件夹，将第三方jar包复制进去，然后在`.classpath`文件中添加一下内容：

```xml
	<classpathentry kind="lib" path="libs/xxx.jar"/>
```

这样就依赖了本地jar包。

### 编译class文件

VSCode会自动在你编辑和保存.java文件时自动进行编译。因此`bin`中的class文件几乎是实时更新的。

如果需要手动编译，执行命令：

![image-20190412151845028](http://1030310877.github.io/images/2019-04-12/image-20190412151845028.png)

### 打jar包

VSCode只会给你编译出class文件，并不会给你打jar包。打jar包的行为可以通过task来进行定义。

配置任务，在`tasks.json`中输入以下内容：

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "clean",
            "type": "shell",
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "shared",
                "showReuseMessage": true,
                "clear": false
            },
            "command": "rm -rf bin && rm MANIFEST.MF",
        },
        {
            "label": "manifest",
            "type": "shell",
            "group": "build",
            "command": "echo \"Manifest-Version: 1.0\" > MANIFEST.MF",
        },
        {
            "label": "package",
            "type": "shell",
            "group": "build",
            "command": "jar",
            "args": [
                "cvfm",
                "${workspaceRoot}/bin/target.jar",
                "${workspaceRoot}/MANIFEST.MF",
                "-C",
                "${workspaceRoot}/bin/",
                ".",
            ],
            "dependsOn": [
                "manifest"
            ]
        },
    ]
}
```

其中包括3个任务，`clean`任务用来清除`bin`文件夹，`manifest`任务用于生成打jar包需要的清单文件，`package`任务用于生成jar包。

在保证`bin`中是最新编译的文件后，直接执行`package`任务，就可以在`bin`下看到`target.jar`包。这个包就可以用来给其他项目引用。