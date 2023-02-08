---
title: flutter笔记
url: https://www.yuque.com/endday/blog/ipz5yw
---

<a name="hYz8m"></a>

# 环境

- [android studio](https://developer.android.google.cn/studio/)
- 模拟器，cd 到夜神模拟器的bin目录下

```bash
nox_adb.exe connect 127.0.0.1:62001
```

Flutter upgrade升级一直停留在 Running pub upgrade...，我用的是VPN，也不行。使用[Flutter Setup: Running pub upgrade](http://www.5imoban.net/jiaocheng/hbuilder/2020/0914/3962.html)这个方法也不行。

最后用国内镜像解决了。方法：
1、计算机->属性->高级系统设置->环境变量，在用户变量下添加下面两个：

PUB\_HOSTED\_URL [https://pub.flutter-io.cn](https://pub.flutter-io.cn/)
FLUTTER\_STORAGE\_BASE\_URL [https://storage.flutter-io.cn](https://storage.flutter-io.cn/)

2、重启电脑

3、将flutter SDK->bin目录->cache

4、打开命令行，重新输入 flutter doctor，会自动升级：
