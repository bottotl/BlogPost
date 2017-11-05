---
title: isExcludedFromBackupKey 的使用
date: 2017-03-04 12:09:28
tags: iOS
categories: 学习
---

使用设备&工具

- iPhone 5s iOS 9.1 （已越狱）
- PP助手 for Mac
- iTunes
- Xcode 8.0 beta 2

[这里是 Demo 的地址](https://github.com/bottotl/isExcludedFromBackupKeyDemo)
 <!-- more --> 
 
实验步骤：

1. 在 <Application_Data>/Documents 文件夹下写入文件夹&文件，修改 isExcludedFromBackupKey 值
1. 使用 iTunes 进行备份
1. 使用 iTunes 从备份中恢复
1. 使用 PP助手 进入 <Application_Data>/Documents 查看相关文件状况
1. 实验方案：
1. 为了验证 isExcludedFromBackupKey 如何正确使用，验证了以下3种情况：
1. 文件夹的 isExcludedFromBackupKey 为 false，文件的 isExcludedFromBackupKey 值为 false
1. 文件夹的 isExcludedFromBackupKey 为 false，文件的 isExcludedFromBackupKey 值为 true
1. 文件夹的 isExcludedFromBackupKey 为 true，文件A 的 isExcludedFromBackupKey 值为 true，文件B 的isExcludedFromBackupKey 值为 false
实验结果

**情况一**

同步前

文件名 | 存在 | sExcludedFromBackupKey
---|---|---
cage1 | true | false
A1 |	true|	false

同步后
 
文件名| 存在 | isExcludedFromBackupKey
---|---|---
cage1|true|false
A1|true| false
 
**情况二**

同步前

文件名 | 存在 |isExcludedFromBackupKey
---|---|---
cage2|true|false
A1|true|false
A2|true|true

同步后

文件名 | 存在 | isExcludedFromBackupKey
---|---|---
cage2|true|false
A1|	true|	false
A2|	false|	\
 
**情况三**

 同步前
 
文件名| 存在|isExcludedFromBackupKey
---|---|---
cage3|true|true
A1|true|	false
A2|	true	|true
 
同步后

文件名|存在 | isExcludedFromBackupKey
---|---|---
cage3|false|	\
A1	|false|	\
A2	|false|	\

