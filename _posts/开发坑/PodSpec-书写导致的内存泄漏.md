---
title: PodSpec 书写导致的内存泄漏
date: 2017-08-14 18:31:09
tags:
categories:
- 开发坑
---

代码地址：[TestBuilder](https://github.com/bottotl/TestBuilder)
<!-- more --> 
问题代码:

	- (void)viewDidLoad {
		[super viewDidLoad];
		TestDispatchObjcA *builderA = [TestDispatchObjcA new];
		[builderA buildComposition];
		
		TestDispatchObjcB *builderB = [TestDispatchObjcB new];
		[builderB buildComposition];	
	}
	………………
	………………
	 
	@interface TestDispatchObjcB
	@end
	@implementation TestDispatchObjcB
	#pragma mark - Composition
		- (void)buildComposition {
		dispatch_group_t dispatchGroupInner = dispatch_group_create();
		dispatch_group_enter(dispatchGroupInner);
		NSLog(@"Entering item");
		dispatch_group_leave(dispatchGroupInner);
		dispatch_group_notify(dispatchGroupInner, dispatch_get_main_queue(), ^{
		});
	}
	@end
	
## 问题描述：调用下面代码会导致内存泄漏

	TestDispatchObjcB *builderB = [TestDispatchObjcB new];
	[builderB buildComposition];

## 问题解决方式：

将 PodSpec 文件中  s.platform     = :ios, "5.0" 修改到 8.0 后 重新 pod install 就能够解决问题。