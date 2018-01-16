---
title: ReplayKit 调用时机的坑
date: 2017-12-10 13:32:48
tags:
categories:
- 开发坑
---

因为在 demo 中测试 ReplayKit，直接把开始录制的行为写在了 didFinishLaunching 中
导致出现如下输出

    Domain=com.apple.ReplayKit.RPRecordingErrorDomain Code=-5803 "开始录制失败" UserInfo={NSLocalizedDescription=开始录制失败}

曾经看到过 ReplayKit 是记录 UIWindow 的每一次渲染，忽然意识到这个时候 window 初始化都没完成。
delay 录屏行为以后发现可以正常使用了