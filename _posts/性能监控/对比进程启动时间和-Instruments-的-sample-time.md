---
title: 对比进程启动时间和 Instruments 的 sample time
date: 2017-11-27 11:33:20
tags:
category: 性能监控
---

## 获取距离进程的启动时间

  + (NSTimeInterval)processTime {
      static dispatch_once_t onceToken;
      dispatch_once(&onceToken, ^{
    pid_t pid = [[NSProcessInfo processInfo] processIdentifier];
    struct kinfo_proc proc;
    size_t size = sizeof(proc);
    int mib[4] = { CTL_KERN, KERN_PROC, KERN_PROC_PID, pid }; sysctl(mib, 4, &proc, &size, NULL, 0);
    long t = proc.kp_proc.p_starttime.tv_sec;
    long s = proc.kp_proc.p_starttime.tv_usec;
    startTime = t + s * 0.000001;
    });
    return [[NSDate date] timeIntervalSince1970] - startTime; } 

## AppDelegate 中相关的代码

 - (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
        // Override point for customization after application launch.
        for (int i = 0 ; i < 1000000; i++) { [self logtime]; }
    return YES; }

    - (void)logtime {
        NSLog(@"===== dove processTime %f", [DOVProcessTime
        processTime]);
    }

## 测试方案

1. Instruments 调试真机，
2. 在所有的 Samples 中找到最早的 次 logtime 调 
3. 在 Xcode --> Windows --> Devices and Simulator 的界 提取设备的 log

