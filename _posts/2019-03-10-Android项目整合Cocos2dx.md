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

![1](http://1030310877.github.io/images/2019-03-10/1.jpg)

### 生成Android项目

精简完模块后，就可以使用构建和编译来生成包含引擎的Android项目。

### 分析Android项目结构

生成好Android项目后，打开项目的build文件夹。可以看到这是个cocos项目中包了个Android项目，我们实际需要的是Android项目中包个cocos项目。因此接下来我们开始分析这个项目结构。

![2](http://1030310877.github.io/images/2019-03-10/2.jpg)

- cocos-project-template.json — Cocos Creator生成的部分项目配置，不用管
- frameworks — 生成的Android项目、IOS项目等都在里面
- jsb-adapter — js-binding相关
- main.js — Cocos游戏入口文件
- Project.json — Cocos游戏配置文件
- res、src 游戏资源目录
- Simulator 模拟器相关，不用管

除去frameworks、jsb-adapter就是个标准的cocos项目。我们接着看Android项目中的gralde依赖关系。

根部的build.gradle没什么好说的，直接进入app文件夹，查看module的build.gradle

```groovy
import org.apache.tools.ant.taskdefs.condition.Os

apply plugin: 'com.android.application'

android {
    compileSdkVersion PROP_COMPILE_SDK_VERSION.toInteger()
    buildToolsVersion PROP_BUILD_TOOLS_VERSION

    defaultConfig {
        minSdkVersion PROP_MIN_SDK_VERSION
        targetSdkVersion PROP_TARGET_SDK_VERSION
        versionCode 1
        versionName "1.0"
        externalNativeBuild {
            ndkBuild {
                if (!project.hasProperty("PROP_NDK_MODE") || PROP_NDK_MODE.compareTo('none') != 0) {
                    // skip the NDK Build step if PROP_NDK_MODE is none
                    targets 'cocos2djs'
                    arguments 'NDK_TOOLCHAIN_VERSION=clang'
                    
                    def module_paths = [project.file("/Applications/CocosCreator.app/Contents/Resources/cocos2d-x"),
                                        project.file("/Applications/CocosCreator.app/Contents/Resources/cocos2d-x/cocos"),
                                        project.file("/Applications/CocosCreator.app/Contents/Resources/cocos2d-x/external")]
                    if (Os.isFamily(Os.FAMILY_WINDOWS)) {
                        arguments 'NDK_MODULE_PATH=' + module_paths.join(";")
                    }
                    else {
                        arguments 'NDK_MODULE_PATH=' + module_paths.join(':')
                    }

                    arguments '-j' + Runtime.runtime.availableProcessors()
                    abiFilters.addAll(PROP_APP_ABI.split(':').collect{it as String})
                }
            }
        }
    }

    sourceSets.main {
        java.srcDir "src"
        res.srcDir "res"
        jniLibs.srcDir "libs"
        manifest.srcFile "AndroidManifest.xml"
    }

    externalNativeBuild {
        ndkBuild {
            if (!project.hasProperty("PROP_NDK_MODE") || PROP_NDK_MODE.compareTo('none') != 0) {
                path "jni/Android.mk"
            }
        }
    }

    buildTypes {
        release {
            debuggable false
            jniDebuggable false
            renderscriptDebuggable false
            externalNativeBuild {
                ndkBuild {
                    arguments 'NDK_DEBUG=0'
                }
            }
        }

        debug {
            debuggable true
            jniDebuggable true
            renderscriptDebuggable true
            externalNativeBuild {
                ndkBuild {
                    arguments 'NDK_DEBUG=1'
                }
            }
        }
    }
}

android.applicationVariants.all { variant ->
    // delete previous files first
    delete "${buildDir}/intermediates/assets/${variant.dirName}"

    variant.mergeAssets.doLast {
        copy {
           from "${buildDir}/../../../../../res"
           into "${buildDir}/intermediates/assets/${variant.dirName}/res"
        }

        copy {
            from "${buildDir}/../../../../../src"
            into "${buildDir}/intermediates/assets/${variant.dirName}/src"
        }        

        copy {
            from "${buildDir}/../../../../../jsb-adapter"
            into "${buildDir}/intermediates/assets/${variant.dirName}/jsb-adapter"
        }

        copy {
            from "${buildDir}/../../../../../main.js"
            from "${buildDir}/../../../../../project.json"
            into "${buildDir}/intermediates/assets/${variant.dirName}"
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar','*.aar'])
    implementation fileTree(dir: "/Applications/CocosCreator.app/Contents/Resources/cocos2d-x/cocos/platform/android/java/libs", include: ['*.jar'])
    implementation project(':libcocos2dx')
}
```

从`dependencies`中可以看到，这个Android项目依赖了:

1. `"/Applications/CocosCreator.app/Contents/Resources/cocos2d-x/cocos/platform/android/java/libs"`下的jar包
2. 一个名叫`libcocos2dx`的module

libs下的jar包，除了显示的目录中可以找到，在所下载的cocos源码中，源码/cocos/platform/android中也能找到。

而`libcocos2dx`的module可以看外面一层的`setting.gradle`中找到。在源码/cocos/platform/android也能看到。

推荐全部使用项目文件中指定的目录，因为cocos creator和下载的源码会存在不一样的情况。

然后我们分析NDK项目。在`defaultConfig`中的ndk可以看出，它指向编译了三个文件夹`cocos2d-x`、`cocos2d-x/cocos`、`cocos2d-x/external`，这个就是cocos源码项目；以及指向本地的一个Android.mk，可以看到对应jni下只有一个`hellojavascript`文件夹，里面有一个`main.cpp`文件。

最后我们看`variant.mergeAssets.doLast`任务，可以看到，这个task将最外层的游戏资源全部拷贝到了assets中，这就说明了从Android启动游戏，我们需要哪些游戏文件。

到这里，如何在已有Android项目中接入cocos引擎已经有了眉目。

1. 将libs的jar包拷贝到已有项目中进行jar包依赖。
2. 依赖`libcocos2dx`module，推荐拷贝这个module到Android项目中进行依赖。
3. 将NDK编译生成的so找到后加入到Android项目中进行依赖。
4. 将最外层的游戏资源，全部移动或拷贝到Android项目的assets文件夹中。

做完以上4点，就能将cocos整合到已有Android项目中了。



### PS

ndk编译生成的so，在cocos生成的android项目目录/app/build/intermediates/ndkBuild/debug/obj/local/对应的架构目录中。不要被这个so的大小吓到，当它打到apk包中（或直接zip压缩）后，它的大小会天差地别。

