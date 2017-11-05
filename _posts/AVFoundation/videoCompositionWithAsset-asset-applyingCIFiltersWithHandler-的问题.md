---
title: 'videoCompositionWithAsset:asset:applyingCIFiltersWithHandler: 的问题'
date: 2017-08-14 18:49:49
tags:
categorys : AVFoundation
---

这个 api 只会作用在一个轨道上
>AVFoundation calls your applier block once for each frame to be displayed (or processed for export) from the asset’s first enabled video track. In that block, you access the video frame and return a filtered result using the provided

如果你希望实现多轨道的视频合成效果，可能需要自己写一个视频合成器（AVVideoCompositing）了
