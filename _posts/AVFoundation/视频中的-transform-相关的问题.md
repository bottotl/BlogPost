---
title: 视频中的 transform 相关的问题
date: 2017-08-14 19:23:57
tags:
categorys : AVFoundation
---

[CGAffineTransform 定义](https://developer.apple.com/documentation/coregraphics/cgaffinetransform)
---

从 AVAssetTrack 中可以获取两个信息：视频尺寸& transform：

	/* indicates the natural dimensions of the media data referenced by the track as a CGSize */
	@property (nonatomic, readonly) CGSize naturalSize;
	/* indicates the transform specified in the track's storage container as the preferred transformation of the visual media data for display purposes;
	its value is often but not always CGAffineTransformIdentity */
	@property (nonatomic, readonly) CGAffineTransform preferredTransform;

## 视频中存储的信息

一般的对于一个播放的时候是竖着视频（如下）来说
![图片2](http://wx1.sinaimg.cn/large/6b5f103fgy1fijh9l4lwmj20bi0icadg.jpg)

他存储的可能是一个横着的图像+旋转信息
![图片2](http://wx1.sinaimg.cn/large/6b5f103fgy1fijh9oq835j20ic0bijvv.jpg)

<!-- more --> 

## 锚点和坐标系
在不同的锚点和坐标系中使用相同的旋转信息会得到不同的结果
1、对于左上角坐标系的图片进行旋转
![旋转前](http://wx2.sinaimg.cn/large/6b5f103fgy1fijh85hxvoj20f90d5q4z.jpg)
![旋转后](http://wx4.sinaimg.cn/large/6b5f103fgy1fijh88971yj20k40bcabq.jpg)
2、对于左下角坐标系的图片进行旋转
![旋转前](http://wx4.sinaimg.cn/large/6b5f103fgy1fijh8an5s6j20g80bimz8.jpg)
![旋转后](http://wx1.sinaimg.cn/large/6b5f103fgy1fijh8doznoj20f00ghmz4.jpg)

## 从 AVAssetTrack 中获取的 transform 
有时候 transform 获取出来是 [0 1 -1 0 0 0] 有时候可能是 [0 1 -1 0 960 0]

对于左上角坐标系而言， [0 1 -1 0 0 0]  代表着把图片以圆点作为锚点顺时针旋转 90°

对于左上角坐标系而言， [0 1 -1 0 960 0]  代表着把图片以圆点作为锚点顺时针旋转 90°，然后沿着 x 轴的正方向平移 960