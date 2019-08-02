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

##### 抓取特定应用的 log
```bash
adb logcat '*:V' -v color -v time | grep `adb shell ps | grep <package_name> | cut -c11-14`

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
