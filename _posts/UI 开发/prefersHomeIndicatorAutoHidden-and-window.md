---
title: prefersHomeIndicatorAutoHidden and window
date: 2017-11-03 10:47:13
tags:
---

## 问题
适配 iPhone X 的过程中发现 prefersHomeIndicatorAutoHidden 不起作用。

## 原因
keyWindow 之上还有一个 window。
并且大小为 [UIScreen mainScreen].bounds

## 解决方案
大小修改为 CGRectInset([UIScreen mainScreen].bounds, 0, 1) 就不会影响 prefersHomeIndicatorAutoHidden 生效。

因为 prefersHomeIndicatorAutoHidden 的实现原理不明，解决方案有效性待观察。