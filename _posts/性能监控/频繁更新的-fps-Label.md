---
title: 频繁更新的 fps Label
date: 2018-01-08 13:22:18
tags: 性能监控
---

在性能监控的项目中非常基本的一个控件就是用于展示 FPS 信息的 Label。
通过在 displayLink 中获取 CoreAnimation 的 frameID 变化计算出瞬时的 FPS，通过 UILabel 展示在屏幕中。

实践过程中发现 UILabel 会有一些问题。

## 性能消耗比较大
因为更新的频次太高，每个 displayLink 都会重新计算新的 FPS ，强制 UILabel 刷新会有大约8%的 CPU 消耗。

## 主线程卡顿时无法更新