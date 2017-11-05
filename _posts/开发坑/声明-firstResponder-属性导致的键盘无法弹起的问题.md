---
title: 声明 firstResponder 属性导致的键盘无法弹起的问题
date: 2017-03-04 11:59:45
tags: 
- iOS
- 坑
categories: 
- 开发坑
---

手动声明了一个名为 firstResponder 的属性，并且给它赋值。
最终导致了键盘无法弹起。
 
解决方案：firstResponder 换个名字，比如 tFirstResponder。
<!-- more --> 
原因：
可能是这是一个 iOS 用来构建响应者链的私有属性，覆写并且对它进行赋值导致了响应者链构建出了问题。
 
问题代码如下：
#import "ViewController.h"


	@interface ViewController ()
	@property (nonatomic, weak) UIView *firstResponder;
	@end
	 
	@implementation ViewController
	 
	- (void)viewDidLoad {
	    [super viewDidLoad];
	    UITextField *mytextfield = [[UITextField alloc] initWithFrame:CGRectMake(0, 20, 200, 100)];
	    mytextfield.backgroundColor = [UIColor redColor];
	    [self.view addSubview:mytextfield];
	    self.firstResponder = mytextfield;
	    UITapGestureRecognizer *tapGesture = [UITapGestureRecognizer new];
	    [tapGesture addTarget:self action:@selector(viewTap)];
	    [self.view addGestureRecognizer:tapGesture];
	}
	 
	 
	- (void)viewTap {
	    [self.firstResponder resignFirstResponder];
	    [self.view becomeFirstResponder];
	}
	@end