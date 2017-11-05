---
title: AVAssetWriterInput 引起的 crash
date: 2017-08-14 18:41:08
tags:
categorys : AVFoundation
---


如果 asset 中没有音频轨道
却调用了如下方法给 writer 添加了一个 input 就会导致报错:

<!-- more --> 

> -[AVAssetWriterInput appendSampleBuffer:] Must start a session (using -[AVAssetWriter startSessionAtSourceTime:) before appending sample buffers'

 
	AVAssetWriterInput *assetWriterAudioInput = [AVAssetWriterInput assetWriterInputWithMediaType:AVMediaTypeAudio outputSettings:[self writerOutputAudioSettings]];
        // Add the input to the writer if possible.
        
        if ([assetWriter canAddInput:assetWriterAudioInput]) {
            [assetWriter addInput:assetWriterAudioInput];
        } 
