---
title: 统计 Cell 耗时
date: 2017-11-24 10:21:20
tags: 
categorys : 性能监控
---

# 统计 Cell 耗时


为了做性能监控，需要统计每个 view 的调用时间—— layout 是其中较耗时的行为。

## 如何统计时间

我们设想有这样一个 Cell 的层级关系
<!-- more --> 

    -Cell(0)
    |--ContentView(1.1)
    |        |-view(1.1.1)
    |        |-view(1.1.2)
    |
    |-view(1.2)
    |-view(1.3)

最一开始我们只统计了 cell 的 layoutSubview，发现时间非常不准
其实这个 Cell 可能的更新的顺序是

    cellForRow
    0 layout stat 
    0 layout end
    1.1 layout start
    1.1 layout end
    1.1.1 layout start
    1.1.1 layout end
    1.1.2 layout start
    1.1.2 layout end
    1.2 layout start
    1.2 layout end
    1.3 layout start
    1.3 layout end

我们只统计了 1 layout start --> 1 layout end 的时长，后面都漏掉了

## 统计时间方案一：向依赖视图树传递耗时数据

给每个 View 添加一个属性“耗时”，每次布局完成更新这个耗时。

View 负责把这个数据向父 View 传递，直到找到一个 UITableViewCell 停止传递

    @interface UIView (TimeUsage)
    @property (nonatomic, assign) NSTimeInterval layoutSubviewCostTime;
    ……
    @end

    @implementation UIView (TimeUsage)

    - (void)setLayoutSubviewCostTime:(NSTimeInterval)layoutSubviewCostTime {
        if ([self isKindOfClass:[UITableViewCell class]]) {
            UITableViewCell *cell = (UITableViewCell *)self;
            cell.cellAllLayoutCostTime += layoutSubviewCostTime;
        } else {
            [self sendLayoutSubviewCostTimeToSuperView:layoutSubviewCostTime];
        }
    }
    - (void)sendLayoutSubviewCostTimeToSuperView:(NSTimeInterval)costTime {
        if ([self.superview isKindOfClass:[UIWindow class]] ||
            [self.superview isKindOfClass:[UIViewController class]]) {
            return;
        }
        if ([self isKindOfClass:[UITableViewCell class]]) {
            UITableViewCell *cell = (UITableViewCell *)self;
            cell.cellAllLayoutCostTime += costTime;
        } else if (self.superview) {
            [self.superview sendLayoutSubviewCostTimeToSuperView:costTime];;
        }
    }
    @end

这种方案会引发一个非常严重的性能问题：
这个消息一直向父 View 传递，并且这个消息大部分情况会遍历到一个很深的层次才会得到终止（找不到 UITableViewCell）。

## 统计时间方案二：View 添加 Cell 标记

**给所有 View 添加一个关联对象**

    @interface UIView : (TimeUsage)
    @property (nonatomic, weak) UITableViewCell *belongCell;
    @end


**当 Cell 布局结束的时候递归遍历所有的子 View，并把自身赋值给 belongCell**
这样所有需要被统计耗时的 view 就知道自己的耗时数据需要给谁
ps：因为复用，这里需要清空以前的数据

**每个 View 布局完成统计自身耗时，并把该值传给 belongCell**

    if (customView.belongCell) {
        customView.layoutSubviewCostTime = layoutDuration;
    }
    (省略实现)
    @implementation UIView (TimeUsage)
    - (void)setLayoutSubviewCostTime:(NSTimeInterval)layoutSubviewCostTime {
        self.belongCell.cellAllLayoutCostTime += layoutSubviewCostTime;
    }
    @end

实验验证这种方案准确度比较高，符合预期

## 记录 cell 最后一个 view layout 结束的时间

如果每个 View 更新的时候都去记录 Log 肯定是最好的。
但是考虑到 View 的量非常大（Cell 有十几个子 View 非常正常）。每个都记录会导致写日志压力很大。
因此希望能记录单个 Cell 最后一个 view layout 的时间。

**第一直觉是，这东西应该是能够找到的**

思考 layout 的过程
- 先标记需要被布局的 view
- 在合适的时间去布局

理论上我们应该是可以在 Cell layout 的时候递归便利所有子 view，统计有哪些被标记的数量。
每次布局结束就从标记的数量中减去一个，我们就能够知道最后一个更新的 View。

**实际上这个值似乎被隐藏在 View 内部，我们访问不到**（反正我是没有找到）

**换种思路**
这个信息对于写入的时机要求不高。我们不需要每次最后一个 View 更新完成以后立刻把这条数据写入日志。
我们可以在运行时只记录信息，当真正需要写日志之前去拿到最后一条加入数组的数据（也就是最新的），至于日志乱序的问题可以在解析的时候得到解决。

**所以设计了如下方案**

每个 displayLink 循环中都维护了一个数组，这个数组中记录了这一帧中被更新的 cell

    @property (nonatomic, strong) NSMutableArray<UITableViewCell *> *layoutCells;///< 当前 tick 中

每个 View 布局开始和结束的时候查询自己的归属 Cell，并且把信息保存在 Cell 里

    if (customView.belongCell) {
        LayoutInfoLog *layoutLog = [LayoutInfoLog new];
        layoutLog.layoutTime = startTime;
        layoutLog.state = 0;
        layoutLog.view = customView;
        [customView.belongCell.layoutInfoLogs addObject:layoutLog];
    }

下一个 displayLink 到来的时候遍历 Cell 数组获取自己想要的数据，并且写入日志

    - (void)beforeTick:(NSNotification *)noti {
        [self.layoutCells enumerateObjectsUsingBlock:^(UITableViewCell * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
            LayoutInfoLog *lastLayoutLog = obj.layoutInfoLogs.lastObject;
            LogInfo *layoutLog = [LogInfo new];
            layoutLog.objIdentifier = [NSString stringWithFormat:@"%p", obj];
            layoutLog.event = @"cell end layout";
            layoutLog.timestamp = lastLayoutLog.layoutTime;
            [self.timeLogs addObject:layoutLog];
        }];
    }


实现细节
---- 

## hook layoutSublayersOfLayer 还是 layoutSubviews？

其中 layoutSublayersOfLayer 是 UIView 实现的 CALayer 的协议
layoutSubviews 是 UIView 的实例方法

最终选择在 layoutSublayersOfLayer 上下钩子。

**理由一：layoutSublayersOfLayer 的调用包含了 layoutSubview 的调用**

调用栈很清晰

    -[UIView(CALayerDelegate) layoutSublayersOfLayer:] ()
    -[CALayer layoutSublayers] ()
    CA::Layer::layout_if_needed(CA::Transaction*) ()
    -[UIView(Hierarchy) layoutBelowIfNeeded] ()
    +[UIView(Animation) performWithoutAnimation:] ()
    -[UITableView _createPreparedCellForGlobalRow:withIndexPath:willDisplay:] ()
    -[UITableView _createPreparedCellForGlobalRow:willDisplay:] ()
    -[UITableView _updateVisibleCellsNow:isRecursive:] ()
    -[UITableView layoutSubviews] ()
    -[UIView(CALayerDelegate) layoutSublayersOfLayer:] ()
    -[CALayer layoutSublayers] ()
    CA::Layer::layout_if_needed(CA::Transaction*) ()
    CA::Context::commit_transaction(CA::Transaction*) ()
    CA::Transaction::commit() ()
    _afterCACommitHandler ()
    __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__ ()
    __CFRunLoopDoObservers ()
    __CFRunLoopRun ()
    CFRunLoopRunSpecific ()
    GSEventRunModal ()
    UIApplicationMain ()
    main at /xxx/main.m:14
    start ()

**理由二：layoutSublayerOfLayer 和 layoutSubviews 不是一一对应关系**

从下面的 log 可以看出， layoutSubviews 触法不够及时

    ====== layoutSublayersOfLayer time=17079.555645 <xxTableView: 0x7fe6b696d000; baseClass = UITableView; frame = (0 0; 375 724); clipsToBounds = YES; autoresize = W+H; gestureRecognizers = <NSArray: 0x6080008502c0>; layer = <CALayer: 0x608000620e40>; contentOffset: {0, 1310.6666666666667}; contentSize: {375, 2034.503999999899}; adjustedContentInset: {0, 0, 0, 0}> start
    ======*** layoutSublayersOfLayer time=17079.555808 <xxTableView: 0x7fe6b696d000; baseClass = UITableView; frame = (0 0; 375 724); clipsToBounds = YES; autoresize = W+H; gestureRecognizers = <NSArray: 0x6080008502c0>; layer = <CALayer: 0x608000620e40>; contentOffset: {0, 1310.6666666666667}; contentSize: {375, 2034.503999999899}; adjustedContentInset: {0, 0, 0, 0}> end 
    ====== layoutSublayersOfLayer time=17079.621023 <xxEmptyHeaderFooterView: 0x7fe6c206de00; baseClass = UITableViewHeaderFooterView; frame = (0 2034.5; 375 0.0001); text = ''; autoresize = W; userInteractionEnabled = NO; layer = <CALayer: 0x60c000a28a00>> start
    ----- layoutSubviews start time=17079.621051 <xxEmptyHeaderFooterView: 0x7fe6c206de00; baseClass = UITableViewHeaderFooterView; frame = (0 2034.5; 375 0.0001); text = ''; autoresize = W; userInteractionEnabled = NO; layer = <CALayer: 0x60c000a28a00>>
    ----- layoutSubviews end time=17079.621082 <xxEmptyHeaderFooterView: 0x7fe6c206de00; baseClass = UITableViewHeaderFooterView; frame = (0 2034.5; 375 0.0001); text = ''; autoresize = W; userInteractionEnabled = NO; layer = <CALayer: 0x60c000a28a00>>
    ======*** layoutSublayersOfLayer time=17079.621149 <xxEmptyHeaderFooterView: 0x7fe6c206de00; baseClass = UITableViewHeaderFooterView; frame = (0 2034.5; 375 0.0001); text = ''; autoresize = W; userInteractionEnabled = NO; layer = <CALayer: 0x60c000a28a00>> end 

## 还有遗漏的 -drawRect

-drawRect 的方法还没有统计。
绘制是一个耗时操作是毋庸置疑的，并且这个方法会强制生成后备缓冲区。对内存&GPU 的压力都很大，是很容易引起性能降低的点。
UILabel 内部也是通过实现这个方法使用 CoreText 进行绘制的（iOS 11 中观察结果是这样的）。

前期尝试去 hook UIView 的这个方法，发现似乎有一些没有设置背景色的 View 背景色成了黑色。
具体原因还没有去深究，暂时先放弃记录。（反正通过覆写 drawRect 去做业务的情况比较少）
