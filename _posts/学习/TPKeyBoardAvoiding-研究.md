---
title: TPKeyBoardAvoiding 研究
date: 2017-03-04 11:42:57
tags:
categories: 学习
---

[TPKeyboardAvoiding](https://github.com/michaeltyson/TPKeyboardAvoiding)

<!-- more --> 

>There are a hundred and one proposed solutions out there for how to move `UITextField` and `UITextView` out of the way of the keyboard during editing -- usually, it comes down to observing `UIKeyboardWillShowNotification` and `UIKeyboardWillHideNotification`, or implementing `UITextFieldDelegate` delegate methods, and adjusting the frame of the superview, or using `UITableView`'s `scrollToRowAtIndexPath:atScrollPosition:animated:`, but most proposed solutions tend to be quite DIY, and have to be implemented for each view controller that needs it.

解决 `UITextField` and `UITextView` 不被弹起的键盘遮盖的作法有很多，通用的方法有两种种

- 观察 `UIKeyboardWillShowNotification` 和 `UIKeyboardWillHideNotification` 通知
- 去实现 `UITextFieldDelegate` delegate，然后通过`调整 superview 的 frame`，或者 `UITableView` 的 `scrollToRowAtIndexPath:atScrollPosition:animated:` 滑到指定位置

作者认为这样定制性太强

>This is a relatively universal, drop-in solution: `UIScrollView` and `UITableView` subclasses that handle everything.
<br><br>
When the keyboard is about to appear, the subclass will find the subview that's about to be edited, and adjust its frame and content offset to make sure that view is visible, with an animation to match the keyboard pop-up. When the keyboard disappears, it restores its prior size.
<br><br>
It should work with basically any setup, either a UITableView-based interface, or one consisting of views placed manually.
<br><br>
It also automatically hooks up "Next" buttons on the keyboard to switch through the text fields.

采取的方案是：继承 `UIScrollView` 和 `UITableView` ，把事件 hook 住，实现了下面的功能。

- 当键盘弹起，子类去找到将要编辑的子视图（subview），调整 `frame` 和 `content offset` 保证子视图不被遮盖。
- 当键盘消失，恢复弹起前的状态。
- 在使用 `IB` 或者 `手动布局` 的情况都能够起作用。
- 自动 hook 了 `next` 按钮
___

#### 对 TPKeyboardAvoidingTableView 进行分析

**在每个初始化函数调用的时候注册事件监听**

    - (void)setup {
    if ( [self hasAutomaticKeyboardAvoidingBehaviour] ) return;
    
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(TPKeyboardAvoiding_keyboardWillShow:) name:UIKeyboardWillChangeFrameNotification object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(TPKeyboardAvoiding_keyboardWillHide:) name:UIKeyboardWillHideNotification object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(scrollToActiveTextField) name:UITextViewTextDidBeginEditingNotification object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(scrollToActiveTextField) name:UITextFieldTextDidBeginEditingNotification object:nil];
    }

注册了通知：

- UIKeyboardWillChangeFrameNotification 键盘的 frame 改变时立刻调用
- UIKeyboardWillHideNotification 键盘隐藏时调用
- UITextViewTextDidBeginEditingNotification 有一个 text view 将要进入编辑状态
- UITextFieldTextDidBeginEditingNotification 有一个 text field 将要进入编辑状态

**hasAutomaticKeyboardAvoidingBehaviour**

    - (BOOL)hasAutomaticKeyboardAvoidingBehaviour {
        if ( [self.delegate isKindOfClass:[UITableViewController class]] ) {
        // Theory: Apps built using the iOS 8.3 SDK (probably: older SDKs not tested) seem to handle keyboard
        // avoiding automatically with UITableViewController. This doesn't seem to be documented anywhere
        // by Apple, so results obtained only empirically.
        return YES;
        }
        return NO;
    }
作者在 iOS 8.3 SDK 下猜测 UITableViewController 可能自动对键盘收起做了处理。如果判断使用了 UITableViewController 则代码都不会起作用。

**setContentSize**

    - (void)setContentSize:(CGSize)contentSize {
        if ( [self hasAutomaticKeyboardAvoidingBehaviour] ) {
         [super setContentSize:contentSize];
         return;
        }
        if (CGSizeEqualToSize(contentSize, self.contentSize)) {
            // Prevent triggering contentSize when it's already the same
            // this cause table view to scroll to top on contentInset changes
            return;
        }
        [super setContentSize:contentSize];
        [self TPKeyboardAvoiding_updateContentInset];
    }

除了***修改 contentSize*** 会调用，当 ***contentInset 改变***的时候也会调用。

**willMoveToSuperview**

    - (void)willMoveToSuperview:(UIView *)newSuperview {
        [super willMoveToSuperview:newSuperview];
        if ( !newSuperview ) {
            [NSObject cancelPreviousPerformRequestsWithTarget:self selector:@selector(TPKeyboardAvoiding_assignTextDelegateForViewsBeneathView:) object:self];
        }
    }

当 tableView 被***加入或移除其他视图的子视图列表中时调用***。当 ***tableView 被释放之类的情况也会调用***

