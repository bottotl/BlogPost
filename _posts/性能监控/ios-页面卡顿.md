---
title: ios 页面卡顿
date: 2017-11-28 11:06:25
tags:
categoty: 性能监控
---
平衡性能一个很重要的举措就是充分利用 GPU 和 CPU 的性能。
Domo https://github.com/bottotl/TooManyLayersExample.git
## CPU 压力过大
的时候提前加载、懒加载、重用
## GPU 压力过大
当 GPU 压力太大的时候尝试把一部分工作交给 CPU 进行处理。
在 session_238__ios_app_performance__graphics_and_animations 中有讲一个例子， layer 特别多的时候 GPU 负载很高，没办法以60帧进行展示，设置光栅化属性以后依旧没有改善。采取的措施是用 drawRect 方法直接利用 Core Graphics 把 CPU 一部分工作转移到 CPU 上。
 
