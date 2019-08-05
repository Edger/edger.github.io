---
layout:     wiki
title:      "ADB 常用命令 "
subtitle:   " 一些常用的 ADB 命令 "
date:       2019-01-07 17:27:32
author:     "Edger"
header-img: "img/wiki-adb-command.jpg"
catalog:    true
tags:

    - ADB
---

#### ADB Root 步骤

打开 ROOT 工具，选中盘符后发送 `SC_SWITCH_ROOT`

```shell
adb root

adb shell setprop persist.<vendor>.verity_disable 1

adb disable-verity
```

重启

```shell
adb reboot
```

开机后再次选中盘符后发送 SC_SWITCH_ROOT

```shell
adb root

adb remount
```

#### 获得当前活动窗口的信息，包名以及活动窗体：

```shell
adb shell dumpsys window windows | grep mCurrent
```

#### 杀死进程

```shell
adb shell kill [PID]
```

或

```shell
am force-stop <package_name>
```

#### 查看所有进程列表，Process Status

```shell
adb shell ps
```

#### 查看 package_name 程序进程

```
adb shell ps | grep <package_name>
```

#### 查看 PID 进程状态

```shell
adb shell ps -x [PID]
```

#### 实时监听程序进程的变化

```shell
adb shell top | grep <package_name>
```

运行结果如下：

```
eg:
adb shell ps -x 13699
USER           PID    PPID    VSIZE     RSS     WCHAN      PC               NAME
u0_a94    13699 1734  1653292 28404   ffffffff    00000000 S com.polysaas.mdm (u:6, s:6)
```

#### 包名管理命令，获得对应包名的对应 apk 路径：

```shell
adb shell pm  path com.android.settings
```

#### adb shell 查看当前进程和窗口信息可以使用如下命令：

```shell
adb shell dumpsys window windows | grep "Window #"
```

#### 获取当前 activity：

方法一：

```shell
adb shell logcat | grep ActivityManager
```

方法二：

```shell
adb shell dumpsys activity activities
```

#### 查看当前与用户交互的 activity

方法一：

```shell
adb shell dumpsys activity activities | sed -En -e '/Running activities/,/Run #0/p'
```

方法二：

```shell
adb shell dumpsys activity | grep -i run
```

方法三：

```shell
adb shell dumpsys activity | grep "mFoc"
```
