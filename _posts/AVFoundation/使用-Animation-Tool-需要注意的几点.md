---
title: 使用 Animation Tool 需要注意的几点
date: 2017-08-14 18:52:38
tags:
categorys : AVFoundation
---


## 在模拟器上不起作用
AVVideoCompositionCoreAnimationTool 在模拟器上似乎无法被正常渲染。只能渲染出一个背景色（摊手）

## 自定义 AVVideoCompositionInstruction 下如果需要使用
那么需要把 enablePostProcessing 参数设为 YES。

## 坐标系
转化的问题。

## 动画时间坐标与视频时间坐标不一样
>Use this constant to set the CoreAnimation's animation beginTime property to be time 0.
The constant is a small, non-zero, positive value which prevents CoreAnimation from replacing 0.0 with CACurrentMediaTime.

## removedOnCompletion 属性
默认情况下，动画完成后，为提高性能，图层会把动画移除，这样动画的时间一旦过去就无法返回了，但这种逻辑不符合视频动画：因为可能会重新播放等，所以 removedOnCompletion 必须置为NO

## 添加的 layer 的对渲染树有要求

>The videoLayer should be in the animationLayer sublayer tree. The animationLayer should not come from, or be added to, another layer tree.

	+(instancetype)videoCompositionCoreAnimationToolWithPostProcessingAsVideoLayer:(CALayer *)videoLayer inLayer:(CALayer *)animationLayer;
	
## 尝试获取动画对应的图像遇到的问题

	+ (instancetype)videoCompositionCoreAnimationToolWithAdditionalLayer:(CALayer *)layer asTrackID:(CMPersistentTrackID)trackID;

可以通过这个 API 把 Core Animation 加到一个空的轨道里面。
但是开发阶段，我自己写了一个 AVVideoCompositing，尝试通过 AVAsynchronousVideoCompositionRequest 的

	- (nullable CVPixelBufferRef)sourceFrameByTrackID:(CMPersistentTrackID)trackID CF_RETURNS_NOT_RETAINED;
去获取 animation 动画的画面的时候，发现背景色是黑色的，不是不透明的，因此导致我把从其他轨道中获取的图片与动画图片进行合成的时候，这就出问题了。

>当时的解决方案:
>
>因为现在水印什么都是白色的，先用一个 CIFilter 把除了白色的部分都过滤掉，然后把处理后的 CIImage 和其他图片进行合成，效果喜人……

其实只需要把 AVVideoCompositionInstruction 的 enablePostProcessing 参数设为 YES。那就不需要通过 sourceFrameByTrackID 去把动画取出来，系统默认会帮你合成


