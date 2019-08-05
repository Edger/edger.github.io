---
layout:     wiki
title:      "Logcat 常用命令 "
subtitle:   " 通过 Logcat 抓取特定的 log "
date:       2019-01-07 17:27:32
author:     "Edger"
header-img: "img/wiki-adb-command.jpg"
catalog:    true
tags:

    - ADB
    - Logcat
---



做 Android 开发，尤其是 Android 应用开发，我们在调试的时候，抓 log 是必不可少的。一般我们可以通过 `adb` 或者 Android Studio 的 `Logcat` 来抓取 log 。调试应用的时候，我们往往只需要抓取该应用的相关 log ，其他系统或者应用的 log 则可以过滤掉。

这就引出了一个问题，如何只抓取特定应用的 log？如果使用 Android Studio 的话，这是比较好做到的，只需要配置一下自带的 Logcat 就好，本文最后会介绍。但是如果需要用 adb 命令来实现这一操作呢？我发现网上很少文章介绍到这一操作，基本上都是一些介绍 `adb logcat` 基本命令的。不知道大家是不是都是直接使用 
`adb logcat | grep <str>` 这样的命令来抓 log 的，反正我觉得这样抓的话，要不就不是很全，因为有时候我需要看这个应用从头到尾的 log 去了解整个流程，要不就带有其他不相关的 log ，影响查看。

所以，为了能够抓取指定应用的相关 log ，我特地研究了一下 logcat 的相关用法并 Google 了一下相关的资料，最后整理如下。



##### 抓取特定应用的 log

```bash
adb logcat '*:V' -v color -v time | grep `adb shell ps | grep <package_name> | cut -c11-14`
```
或

```bash
adb logcat '*:V' --pid=`adb shell ps | grep <package_name> | cut -c11-14` -v color -v time
```
PS: `cut -c11-14` 需要按情况修改，因为通过 `PS` 命令获取到的 `PID` 在字符串中的列数不一定是这个

##### Android 7.0 以上可使用下面这条命令：
```bash
adb logcat '*:V' --pid=`adb shell pidof -s <package_name>` -v color -v time
```
PS: `'*:V'` 改为 `'<TAG>:V'` 即可抓取包含 `<TAG>` 的 log
##### 抓取特定的 log

```bash
adb logcat '*:V' --pid=`adb shell pidof -s <package_name>` -v color -v time | grep -E "<str>" --color=auto
```

##### 过滤掉不需要的 log

```bash
adb logcat '*:V' --pid=`adb shell pidof -s <package_name>` -v color -v time | grep -vE "<str>" --color=auto
```

##### 在 Android Studio 中使用过滤器过滤不需要的 log，如下：

![Android Studio Logcat 过滤器](/img/in-post/WIKI-Logcat-Command-Android-Studio-Filter.png)

在 Android Studio 过滤器中过滤字符串的正则表达式如下：

- 查找不以 `baidu` 开头的字符串：`^(?!baidu).*$`
- 查找不以 `com` 结尾的字符串：`^.*?(?<!com)$`
- 查找不含有 `baidu` 的字符串：`^(?!.*baidu).*$`

具体可以参考  [**利用正则表达式排除特定字符串**](https://www.cnblogs.com/wangqiguo/archive/2012/05/08/2486548.html) 这篇博客。

