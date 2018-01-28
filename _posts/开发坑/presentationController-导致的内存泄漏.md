---
title: presentationController 导致的内存泄漏
date: 2017-12-18 19:00:54
tags:
---

先放一个 [Demo](https://github.com/bottotl/TestPresentationDemo.git)

某同事错误得把 `presentedViewController` 写成了 `presentationController`，结果导致了内存泄漏。

`presentationController` 的声明中有如下话：

>If you have not yet presented the current view controller, accessing this property creates a presentation controller based on the current value in the modalPresentationStyle property. Always set the value of that property before accessing any presentation controllers.

调用这个属性会导致系统创建一个 `_UIFullScreenPresentationController`.
这是一个 `UIPresentationController` 的子类，用来管理模态的过场动画。

这时候就很蛋疼了，哪怕 `UINavigationContoller` 把 VC pop 出去了，VC 也得不到释放（被 `_UIFullScreenPresentationController`）持有了

`_UIFullScreenPresentationController` 和 VC 互相持有。
正常情况下，动画做完，系统会自动释放管理器（
`_UIFullScreenPresentationController`），但是因为不正常的调用导致他们都得不到释放。

更可悲的是 `_UIFullScreenPresentationController` 的属性都是只读的，你甚至没法手动断开两人的引用环。本来想尝试让他们能正常释放，还是放弃了，毕竟这种奇葩的调用也是少数。

