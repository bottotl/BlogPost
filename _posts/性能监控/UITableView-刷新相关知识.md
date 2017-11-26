---
title: UITableView 刷新相关知识
date: 2017-11-25 13:03:19
tags:
category: 性能监控
---
UITableView 刷新相关知识
----- 

---

## UITableView 布局逻辑

**阶段一：滑动或者其他的行为使得 tableView 被标记为脏**

**阶段二：脏 view 布局**

**阶段三：脏 view 递归询问子 view 更新**

---

### TableView 布局的内部实现
因为`UIKit`内部实现不可见，但是我们能够通过苹果相关文档的描述和调用栈的顺序猜测出内部大概的实现
<!-- more --> 
**布局的入口**

苹果在 `runloop` 加了三个 `observer`

- _beforeCACommitHandler
- _afterCACommitHandler
- CA::Transaction::observer_callback(__CFRunLoopObserver*, unsigned long, void*)

`_afterCACommitHandler ()` 和 `CA::Transaction::observer_callback` 内部实现差不多，区别主要是触发的时机，但是不影响我们分析 tableView 是怎么被更新的
回调的 block 伪代码如下

    CA::Transaction::commit() {
        CA::Context::commit_transaction(CA::Transaction*) () {
            CA::Layer::layout_if_needed(CA::Transaction*) () {
                CollectLayersData *layerTree;
                CA::Layer::collect_layers_(CA::Layer::CollectLayersData*);
                for in layerTree {
                    [layer layoutSublayers];
                }
            }
        }
    }

`CA::Layer::layout_if_needed` 的阶段收集了需要被 `layout` 的 `layer`
遍历这个数组通知布局消息。这时这个数组里面就包含了 tableView

---

综上所述故事大概是这样的：

- 滑动的时候 `TableView` 的 `offset` 被修改了，导致自身被标记为脏
- `CA::Transaction::commit()` 时候发现 `TableView` 需要被更新
- `TableView` 的 `layoutSubLayers` 方法中被调用

**UIView:layoutSubLayers**

    - layoutSubLayers {
       //设置 CALayer 的属性
       self.attr = attr;
       // layout subLayers
       NSArray *layers = self.subLayers;
       for in layers {
           [self layoutSublayersOfLayer:layer];
       }  
    }

另一篇文章[UIView layoutSublayersOfLayer:](http://www.jft0m.com/2017/11/25/UIView-CALayerDelegate-layoutSublayersOfLayer/)猜测了一下内部可能的实现

**UITableView:layoutSubviews **

- 计算可视区域
- 通过可视区域得出需要被复用的 `Cell`，并且调用 `dataSource` 的 `CellForRow`
- 遍历通知 `Cell` 进行 `layout`
- `Cell` 自身以递归的形式通知 `子View` 进行 layout（如果为脏）

调用堆栈

    -[CALayer layoutSublayers] ()

    CA::Layer::layout_if_needed(CA::Transaction*) ()

    -[UIView(Hierarchy) layoutBelowIfNeeded] ()

    +[UIView(Animation) performWithoutAnimation:] ()

    -[UITableView _createPreparedCellForGlobalRow:withIndexPath:willDisplay:] ()

    -[UITableView _createPreparedCellForGlobalRow:willDisplay:] ()

    -[UITableView _updateVisibleCellsNow:isRecursive:] ()

    -[UITableView layoutSubviews] ()


### CA::Transaction::commit()

Transaction commit 有四个阶段
- **layout（布局）：** 在这个阶段，程序设置 View / Layer 的层级信息，设置 layer 的属性，如 frame，background color 等等。[UIView layoutSubViews]和[CALayer layoutSublayers] 就是在这个阶段调用的。
- **Dispaly（画）：** 在这个阶段程序会创建 layer 的 backing image，无论是通过 setContents 将一个 image 传給 layer，还是通过 drawRect: 或 drawLayer: inContext: 来画出来的。所以 drawRect: 等函数是在这个阶段被调用的
1. **Prepare（准备）：** 准备Prepare：在这个阶段，Core Animation 框架准备要渲染的 layer 的各种属性数据，以及要做的动画的参数，准备传递給 render server。同时在这个阶段也会解压要渲染的 image。
1. **Commit（提交）：** 提交Commit：在这个阶段，Core Animation 打包 layer 的信息以及需要做的动画的参数，通过 IPC（inter-Process Communication）传递給 render server。这是一个递归操作，根据打包的Layer层级复杂度来决定递归的次数。大量连续递归的CA::Layer::commit_if_needed调用是这个阶段的显著特征。

### Core Animation 相关知识
[深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/) 

> 当在操作 UI 时，比如改变了 Frame、更新了 UIView/CALayer 的层次时，或者手动调用了 UIView/CALayer 的 setNeedsLayout/setNeedsDisplay方法后，这个 UIView/CALayer 就被标记为待处理，并被提交到一个全局的容器去。
苹果注册了一个 Observer 监听 BeforeWaiting(即将进入休眠) 和 Exit (即将退出Loop) 事件，回调去执行一个很长的函数：
_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv()。这个函数里会遍历所有待处理的 UIView/CAlayer 以执行实际的绘制和调整，并更新 UI 界面。

>CFRunLoopObserver {order = 1999000, activities = 0xa0,
    callout = _afterCACommitHandler}
                
>CFRunLoopObserver {order = 2000000, activities = 0xa0,
        callout = _ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv}

    typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
        kCFRunLoopEntry = (1UL << 0),
        kCFRunLoopBeforeTimers = (1UL << 1),
        kCFRunLoopBeforeSources = (1UL << 2),
        kCFRunLoopBeforeWaiting = (1UL << 5),
        kCFRunLoopAfterWaiting = (1UL << 6),
        kCFRunLoopExit = (1UL << 7),
        kCFRunLoopAllActivities = 0x0FFFFFFFU
    };

翻译一下：    
0xa0 = 10100000 表示 Core Animation 监听了 kCFRunLoopBeforeWaiting 和 kCFRunLoopExit  两个事件。

- order = 1999000 调用 _afterCACommitHandler
- order = 2000000 调用 _ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv

### Core Animation 做了什么

_afterCACommitHandler 大致做了两件事情
1. CA::Transaction::commit()
2. _cleanUpAfterCAFlushAndRunDeferredBlocks


第二个就不是特别清楚了，其中一种调用堆栈如下

    _cleanUpAfterCAFlushAndRunDeferredBlocks
    _runAfterCACommitDeferredBlocks
    -[UITableView _userSelectRowAtPendingSelectionIndexPath:]
    -[UITableView _selectRowAtIndexPath:animated:scrollPosition:notifyDelegate:]
    -[xxViewController tableView:didSelectRowAtIndexPath:]
    -[UINavigationController pushViewController:animated:]
    -[UINavigationController pushViewController:transition:forceImmediate:]
    _UINavigationSoundsEnabled
    IsSystemSoundEnabled
    _CFPreferencesGetAppBooleanValueWithContainer


**CA::Transaction::observer_callback(__CFRunLoopObserver*, unsigned long, void*)**

调用堆栈和_afterCACommitHandler很像
当滑动的 tableView 都时候会通过这个回调触发界面更新


## 参考

http://www.jianshu.com/p/5a4ff799239f