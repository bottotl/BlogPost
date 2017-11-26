---
title: Hook 的策略
date: 2017-11-26 19:09:04
tags:
category: 性能监控
---

想要替换整个项目里面所有 class 自己感兴趣的方法（layoutSubviews;cellForRow;viewDidLoad）以便统计方法调用时长

## 迭代 1.0

- 启动以后拿到所有 class
- 生成一棵以 NSObject 为根节点的继承树
- 在树中找到 UIView 这个节点，向下递归遍历 Swizzle 所有节点我们想要 hook 的方法（layoutSubviews 等）
- 在树中找到 UIViewController 这个节点，向下递归遍历 Swizzle 所有节点我们想要 hook 的方法（viewDidLoad 等）

**生成树的过程太慢。** App 启动以后直接 CPU 跑满卡死。（这东西直接跑在项目里面估计要被同事们打死的）

## 迭代 1.1
不放在主线程跑，异步去跑，效果还是不太能接受。

## 迭代 2.0

把 Hook 阶段进行拆分。
1. 先 Hook 基类
2. 当基类的方法被调用时去判断当前对象是否已经被 Hook。如果没有就去 hook

**一、Hook 基类**
我们只 Hook 一些基类的方法

- UIView : layoutSubviews
- UIViewContoller : viewDidLoad

**二、动态 Hook**

*这一步执行时必须保证第二步已经完成*

当基类的方法被调用的时候需要执行的逻辑

1. 根据调用对象的 Class 和 SEL 拼接字符串
2. 通过字符串判断是否已经 Hook
3. Hook if needed

举个例子：
我们启动的时候只 Hook `UIViewContoller` 的 `viewDidLoad`。
在 `viewDidLoad` 方法检查这个实例是否被 hook。如果没有就 hook 一次。

**缺点** hook以后只能等下一次调用才会起作用。需要先进这个 VC 去触发一次 hook 的行为，然后退退出，下次再进来才能有数据。

## 迭代 2.1

写 Category ，在 `+ (void)initialize` 把这个 class 加入一个缓存数组里。
在合适的时候异步 hook 缓存数组中所有 class。
