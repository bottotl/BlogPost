---
title: AVCompositon 插入视频轨道却无法被 AVPlayer 播放
date: 2017-08-14 18:43:55
tags:
categorys : AVFoundation
---

	- (BOOL)insertTimeRange:(CMTimeRange)timeRange ofTrack:(AVAssetTrack *)track atTime:(CMTime)startTime error:(NSError * _Nullable *)outError;
	
其中的 startTime 给了一个 CMTimeMakeWithSeconds(1, NSEC_PER_SEC) 创建的 CMTime ，导致了生成的 AVCompositon 无法被 AVPlayer 播放。

修改为 CMTimeMakeWithSeconds(1, 600) 就可以正常工作。