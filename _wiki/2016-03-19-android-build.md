---
layout:     wiki
title:      "理解 Android 编译命令"
subtitle:   "Android 编译系统常用命令"
date:       2016-03-19 21:19:12
header-img: "img/wiki-adb-command.jpg"
catalog:    true
tags:

    - android
    - tool
---

### 注：本文转自 Gityuan 的博客 [理解 Android 编译命令](http://gityuan.com/2016/03/19/android-build/)。

> 工欲善其事，必先利其器，对于想要深入学习 Android 源码，必须先掌握 Android 编译命令。

## 一、引言

关于 Android Build 系统，这个话题很早就打算整理下，迟迟没有下笔，决定跟大家分享下。先看下面几条指令，相信编译过 Android 源码的人都再熟悉不过的。
```shell
source setenv.sh
lunch
make -j12
```
记得最初刚接触 Android 时，同事告诉我用上面的指令就可以编译 Android 源码，指令虽短但过几天就记不全或者忘记顺序，每次编译时还需要看看自己的云笔记，冰冷的指令总是难以让我记忆。后来我决定认真研究下这个指令的含义。知其然还需知其所以然，这样能更深层次的理解并记忆，才能与自身的知识体系建立强连接，或许还有意外收获，果然如此，接下来跟大家分享一下在研究上述几条指令含义的过程中，深入了解到的 Android Build(编译) 系统。

## 二、编译命令

准备好编译环境后，编译 Android 源码的第一步是 `source build/envsetup.sh`，其中 source 命令就是用于运行 shell 脚本命令，功能等价于 "."，因此该命令也等价于 `.build/envsetup.sh`。在文件 `envsetup.sh` 声明了当前会话终端可用的命令，这里需要注意的是当前会话终端，也就意味着每次新打开一个终端都必须再一次执行这些指令。起初并不理解为什么新开的终端不能直接执行 make 指令，到这里总算明白了。

接下来，解释一下本文开头的引用的命令：
```shell
source setenv.sh // 初始化编译环境，包括后面的 lunch 和 make 指令
lunch // 指定此次编译的目标设备以及编译类型
make  -j12 // 开始编译，默认为编译整个系统，其中 - j12 代表的是编译的 job 数量为 12。
```
所有的编译命令都在 `envsetup.sh` 文件能找到相对应的 function，比如上述的命令 `lunch`，`make`，在文件一定能找到
```shell
function lunch(){
    ...
}

function make(){
    ...
}
```
`source envsetup.sh`，需要 cd 到 setenv.sh 文件所在路径执行，路径可能在 build/envsetup.sh，或者 integrate/envsetup.sh，再或者不排除有些厂商会封装自己的 .sh 脚本，但核心思路是一致的。

具体实现这里就不展开说明，下面精炼地总结了一下各个指令用法和功效。


#### 2.1 代码编译

| 编译指令 | 解释 |
|---|---|
|m | 在源码树的根目录执行编译 |
|mm | 编译当前路径下所有模块，但不包含依赖 |
|mmm [module_path]| 编译指定路径下所有模块，但不包含依赖 |
|mma | 编译当前路径下所有模块，且包含依赖 |
|mmma [module_path]| 编译指定路径下所有模块，且包含依赖 |
|make [module_name] | 无参数，则表示编译整个 Android 代码 |

下面列举部分模块的编译指令：

| 模块 | make 命令 | mmm 命令 |
|---|---|---|
|init|make init| mmm system/core/init
|zygote|make app_process|mmm frameworks/base/cmds/app_process|
|system_server|make services|mmm frameworks/base/services|
|java framework|make framework|mmm frameworks/base|
|framework 资源 | make framework-res|mmm frameworks/base/core/res|
|jni framework|make libandroid_runtime|mmm frameworks/base/core/jni|
|binder|make libbinder|mmm frameworks/native/libs/binder

上述 mmm 命令同样适用于 mm/mma/mmma，编译系统采用的是增量编译，只会编译发生变化的目标文件。当需要重新编译所有的相关模块，则需要编译命令后增加参数 `-B`，比如 make -B [module_name]，或者 mm -B [module_path]。


**Tips:**

- 对于 `m、mm、mmm、mma、mmma` 这些命令的实现都是通过 `make` 方式来完成的。
- mmm/mm 编译的效率很高，而 make/mma/mmma 编译较缓慢；
- make/mma/mmma 编译时会把所有的依赖模块一同编译，但 mmm/mm 不会;
- 建议：首次编译时采用 make/mma/mmma 编译；当依赖模块已经编译过的情况，则使用 mmm/mm 编译。


### 2.2 代码搜索

| 搜索指令 | 解释 |
|---|---|
|cgrep | 所有 **C/C++** 文件执行搜索操作 |
|jgrep | 所有 **Java** 文件执行搜索操作 |
|ggrep | 所有 **Gradle** 文件执行搜索操作 |
|mangrep [keyword]| 所有 **AndroidManifest.xml** 文件执行搜索操作 |
|mgrep [keyword]| 所有 **Android.mk** 文件执行搜索操作 |
|sepgrep [keyword]| 所有 **sepolicy** 文件执行搜索操作 |
|resgrep [keyword]| 所有本地 **res/*.xml** 文件执行搜索操作 |
|sgrep [keyword]| 所有资源文件执行搜索操作 |

上述指令用法最终实现方式都是基于 `grep` 指令，各个指令用法格式：
```shell
xgrep [keyword]  //x 代表的是上表的搜索指令
```
例如，搜索所有 AndroidManifest.xml 文件中的 `launcher` 关键字所在文件的具体位置，指令
```shell
mangrep launcher
```
再如，搜索所有 Java 代码中包含 zygote 所在文件
```shell
jgrep zygote
```
又如，搜索所有 system_app 的 selinux 权限信息
```shell
sepgrep system_app
```
**Tips:** Android 源码非常庞大，直接采用 grep 来搜索代码，不仅方法笨拙、浪费时间，而且搜索出很多无意义的混淆结果。根据具体需求，来选择合适的代码搜索指令，能节省代码搜索时间，提高搜索结果的精准度，方便定位目标代码。

### 2.3 导航指令

| 导航指令 | 解释 |
|---|---|
|croot | 切换至 Android 根目录 |
|cproj | 切换至工程的根目录 |
|godir [filename]| 跳转到包含某个文件的目录 |

**Tips:**  当每次修改完某个文件后需要编译时，执行 `cproj` 后会跳转到当前模块的根目录，也就是 Android.mk 文件所在目录，然后再执行 mm 指令，即可编译目标模块；当进入源码层级很深后，需要返回到根目录，使用 `croot` 一条指令完成；另外 `cd -` 指令可用于快速切换至上次目录。

### 2.4 信息查询

| 查询指令 | 解释 |
|---|---|
|hmm | 查询所有的指令 help 信息 |
|**findmakefile**| 查询当前目录所在工程的 Android.mk 文件路径 |
|print_lunch_menu | 查询 lunch 可选的 product|
|printconfig | 查询各项编译变量值 |
|gettop | 查询 Android 源码的根目录 |
|gettargetarch | 获取 TARGET_ARCH 值 |

### 2.5 其他指令

上述只是列举比较常用的指令，还有其他指令，而且不同的 build 编译系统，支持的指令可能会存在一些差异，当忘记这些编译指令，可以通过执行 `hmm`，查询指令的帮助信息。

最后再列举两个比较常用的指令：

- make clean：执行清理操作，等价于 `rm -rf out/`
- make update-api：更新 API，在 framework API 改动后需执行该指令，Api 记录在目录 `frameworks/base/api`；


## 三、编译系统

Android 编译系统是 Android 源码的一部分，用于编译 Android 系统，Android SDK 以及相关文档。该编译系统是由 Make 文件、Shell 以及 Python 脚本共同组成，其中最为重要的便是 Make 文件。关于编译系统可参考 [理解 Android Build 系统](http://www.ibm.com/developerworks/cn/opensource/os-cn-android-build/)。

### 3.1 Makefile 分类

整个 Build 系统的 Make 文件分为三大类：

- **系统核心的 Make 文件：** 定义了 Build 系统的框架，文件全部位于路径 `/build/core`，其他 Make 文件都是基于该框架编写的；
- **针对产品的 Make 文件：** 定义了具体某个型号手机的 Make 文件，文件路径位于 `/device`，该目录下往往又以公司名和产品名划分两个子级目录，比如 `/device/qcom/msm8916`；
- **针对模块的 Make 文件：** 整个系统分为各个独立的模块，每个模块都一个专门的 Make 文件，名称统一为 "Android.mk"，该文件定义了当前模块的编译方式。Build 系统会扫描整个源码树中名为 "Android.mk" 的问题，并执行相应模块的编译工作。


### 3.2 编译产物

经过 `make` 编译后的产物，都位于 `/out 目录 `，该目录下主要关注下面几个目录：

- /out/host：Android 开发工具的产物，包含 SDK 各种工具，比如 adb，dex2oat，aapt 等。
- /out/target/common：通用的一些编译产物，包含 Java 应用代码和 Java 库；
- /out/target/product/[product_name]：针对特定设备的编译产物以及平台相关 C/C++ 代码和二进制文件；

在 / out/target/product/[product_name] 目录下，有几个重量级的镜像文件：

- system.img: 挂载为根分区，主要包含 Android OS 的系统文件；
- ramdisk.img: 主要包含 init.rc 文件和配置文件等；
- userdata.img: 被挂载在 /data，主要包含用户以及应用程序相关的数据；

当然还有 boot.img，reocovery.img 等镜像文件，这里就不介绍了。

### 3.3 Android.mk 解析

在源码树中每一个模块的所有文件通常都相应有一个自己的文件夹，在该模块的根目录下有一个名称为 “Android.mk”
的文件。编译系统正是以模块为单位进行编译，每个模块都有唯一的模块名，一个模块可以有依赖多个其他模块，模块间的依赖关系就是通过模块名来引用的。也就是说当模块需要依赖一个 jar 包或者 apk 时，必须先将 jar 包或 apk 定义为一个模块，然后再依赖相应的模块。

对于 Android.mk 文件，通常都是以下面两行
```shell
LOCAL_PATH := $(call my-dir)  // 设置当编译路径为当前文件夹所在路径
include $(CLEAR_VARS)  // 清空编译环境的变量（由其他模块设置过的变量）
```
为方便模块编译，编译系统设置了很多的编译环境变量，如下：

- LOCAL_SRC_FILES：当前模块包含的所有源码文件；
- LOCAL_MODULE：当前模块的名称（具有唯一性）；
- LOCAL_PACKAGE_NAME：当前 APK 应用的名称（具有唯一性）；
- LOCAL_C_INCLUDES：C/C++ 所需的头文件路径;
- LOCAL_STATIC_LIBRARIES：当前模块在静态链接时需要的库名;
- LOCAL_SHARED_LIBRARIES：当前模块在运行时依赖的动态库名;
- LOCAL_STATIC_JAVA_LIBRARIES：当前模块依赖的 Java 静态库;
- LOCAL_JAVA_LIBRARIES：当前模块依赖的 Java 共享库;
- LOCAL_CERTIFICATE：签署当前应用的证书名称，比如 platform。
- LOCAL_MODULE_TAGS：当前模块所包含的标签，可以包含多标签，可能值为 debgu,eng,user,development 或 optional（默认值）

针对这些环境变量，编译系统还定义了一些便捷函数，如下：

- $(call my-dir)：获取当前文件夹路径；
- $(call all-java-files-under, <src>)：获取指定目录下的所有 Java 文件；
- $(call all-c-files-under, <src>)：获取指定目录下的所有 C 文件；
- $(call all-Iaidl-files-under, <src>) ：获取指定目录下的所有 AIDL 文件；
- $(call all-makefiles-under, <folder>)：获取指定目录下的所有 Make 文件；

示例：
```shell
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)

# 获取所有子目录中的 Java 文件
LOCAL_SRC_FILES := $(call all-subdir-java-files)

# 当前模块依赖的动态 Java 库名称
LOCAL_JAVA_LIBRARIES := com.gityuan.lib

# 当前模块的名称
LOCAL_MODULE := demo

# 将当前模块编译成一个静态的 Java 库
include $(BUILD_STATIC_JAVA_LIBRARY)
```
