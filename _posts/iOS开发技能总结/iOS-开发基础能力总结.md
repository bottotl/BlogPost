---
title: iOS 开发基础能力总结
date: 2018-01-16 19:27:38
tags: iOS开发技能总结
---

##知识查询

开发文档查阅
业界动态

暂时空缺，这里可以添加很多资源链接

##工具使用

**Xcode**
1. facebook/chisel
2. 断点调试
3. LLDB

**Instruments**

**CocoaPods**

**使用 FLEX UI 调试**
<!-- more --> 
**命令行工具**
1. XcodeBuild
2. xcrun 工具集
3. xctool

## Objective-C

**基本句法(C 语言)**
1. const/static 等关键字
2. 正确定义一个字符串常量

**[命名规范](https://developer.apple.com/library/content/documentation/General/Conceptual/DevPedia-CocoaCore/CodingConventions.html#//apple_ref/doc/uid/TP40008195-CH53-SW1)**

**Cocoa Core Competencies**
https://developer.apple.com/library/content/documentation/General/Conceptual/DevPedia-CocoaCore/
1. Class cluster
2. ……

**Category**

**内存管理**
1. 引用计数与 ARC
1. [内存管理关键字](https://www.jianshu.com/p/da05788327ed)
1. 循环引用分析
1. AutoReleasePool

**@property**
1. Objective-C 和 C 的区别
1. 覆写 getter/setter
1. @synthesize/@dynamic

**block**
1. 抽象意义
1. [Strong-Weak Dance](https://www.jianshu.com/p/4e6153ea2734)
1. _block
1. 简单实现原理

**KVC/KVO**

**对象模型/消息机制**
1. self 与 super
1. Unrecognized Selector 错误

**runtime**
1. Method Swizzing / Class Swizzing
1. 动态创建实例
1. runtime.h / NSObjeCRuntime.h 中的函数

## Cocoa 基础

### UIKit

**UIApplicaiton**
AppDelegate 管理的应用生命周期

**UIViewContoller**
1. UIViewContoller 的生命周期
1. UITableViewController/UICollectionViewController
1. UINavigationController
1. 页面跳转 （push/present）
1. 如何防止臃肿的 UIViewController

**UIView**
1. UIControl 以及其相关子嘞
1. UIScrollView 以及其相关的子类
1. UIWebView / WKWebView

### Foundation
**基础的数据结构和字面量语法**

**多线程**
NSOperation
与 GCD 对比

线程同步设施

- 各种锁
- @synchronized

**网络相关**

- NSURLConnection
- NSURLSession
- NSURLProtocol
- 序列化与反序列化，缓存

### GCD

串行/并行队列，队列与线程
- dispatch_queue
- dispatch_async
- dispatch_once
- sync,group,barrier 等
- semaphore

### 其他
**UIEvent,触摸事件**

	- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event;
	- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event;

手势事件

**CG与CF类库**
**简单动效实现**
**runloop**

## 数据

**序列化与反序列化 NSCoder/NSCoding**
**UserDefault**
**SQLite**
**Core Date**

## 常用的第三方库
**AFNetwork**
**Masonry**
**Mantle**
**libextobjc**
**ReactiveCocoa**
……

## Cocoa 常见的设计模式
**委托模式**
**target-action**
**广播模式**
**观察者模式**
**Responder Chain**
**单例**

## 架构
**MVC**
**路由跳转**

## 其他
Crash 分析
安全常识
性能调优
FRP 思维以及 RAC 库的使用

## JavaScript

**基本语法**
**JavascriptCore/JSBridge 与 Native 的交互**
**JSPatch**









