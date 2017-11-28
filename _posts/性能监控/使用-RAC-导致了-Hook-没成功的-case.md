---
title: 使用 RAC 导致了 Hook 没成功的 case
date: 2017-11-28 10:23:25
tags:
category: 性能监控
---

记录一个问题。

UIView 的 layoutSubview 被调用的时候 hook 当前 Class。 发现后一次 hook 在一定情况下永远不生效。

检查发现，有人动态给这个实例创建了一个子类（和 KVO 类似），并且把用关联对象把原始 layoutSubview 的 IMP 保存了起来。
<!-- more --> 
然后用 forwardInvocation 替换原始实现。
系统调用这个实例的 layoutSubview 会进入 forwardInvocation，他可以进行一些行为（记录开始结束时间）。
但是由于我们 Hook 的时机在他之后，当 forwardInvocation 调用原始实现的时候会直接越过我们的方法。