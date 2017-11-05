---
title: AVVideoCompositionInstruction 的 timeRange 需要注意的点
date: 2017-08-14 18:46:38
tags:
categorys : AVFoundation
---

	@interface AVVideoComposition : NSObject <NSCopying, NSMutableCopying> 
	/* Indicates instructions for video composition via an NSArray of instances of classes implementing the AVVideoCompositionInstruction protocol.
	   For the first instruction in the array, timeRange.start must be less than or equal to the earliest time for which playback or other processing will be attempted
	   (note that this will typically be kCMTimeZero). For subsequent instructions, timeRange.start must be equal to the prior instruction's end time. The end time of
	   the last instruction must be greater than or equal to the latest time for which playback or other processing will be attempted (note that this will often be
	   the duration of the asset with which the instance of AVVideoComposition is associated).
	*/
	@property (nonatomic, readonly, copy) NSArray<id <AVVideoCompositionInstruction>> *instructions;
	
第一个 instruction 的 timeRange.start 必须比 composition 的开始时间要早（或者相同）。
接下来的相邻的 instruction 的 timeRange.start 和 timeRange.end 必须相同。
最后一个 instruction 的 timeRange.end 必须比 composition 的结束时间要迟（或者相同）。

<!-- more --> 

写了几个帮助方法，可以帮助检查插入的时间轴是否有问题

	- (BOOL)instructionsGuard:(NSArray <UGCAVCustomVideoCompositionInstruction *> *)instructions
	            withStartTime:(CMTime)startTime
	               andEndTime:(CMTime)endTime {
	    
	    /// first timeRange.start must not greater than startTime
	    if (CMTimeCompare(instructions.firstObject.timeRange.start, startTime) == 1) {
	        NSLog(@"first timeRange.start must not greater than startTime");
	        return NO;
	    }
	    
	    if (CMTimeCompare(CMTimeAdd(instructions.lastObject.timeRange.start,
	                                instructions.lastObject.timeRange.duration), endTime) < 0) {
	        NSLog(@"last instruction must be greater than or equal to the latest time for which playback or other processing will be attempted");
	        return NO;
	    }
	    
	    __block CMTimeRange lastTimeRange = kCMTimeRangeInvalid;
	    __block BOOL flag = YES;
	    [instructions enumerateObjectsUsingBlock:^(UGCAVCustomVideoCompositionInstruction * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
	        NSLog(@"timeRang[%lu]", (unsigned long)idx);
	        [self logTimeRange:obj.timeRange];
	        if (!CMTIMERANGE_IS_VALID(obj.timeRange)) {
	            *stop = YES;
	            flag = NO;
	            NSLog(@"timeRange not valid at index: %lu", (unsigned long)idx);
	        }
	        if (CMTIMERANGE_IS_VALID(lastTimeRange)) {
	            CMTime endtime = CMTimeAdd(lastTimeRange.start, lastTimeRange.duration);
	            if (CMTimeCompare(endtime, obj.timeRange.start) != 0) {
	                *stop = YES;
	                flag = NO;
	                NSLog(@"timeRange not valid at index: %lu \n error: For subsequent instructions, timeRange.start must be equal to the prior instruction's end time", (unsigned long)idx);
	            }
	        }
	        lastTimeRange = obj.timeRange;
	    }];
	    return flag;
	}
	
	- (void)logTimeRange:(CMTimeRange)timeRange {
   	 NSLog(@"start:%f \n end:%f",CMTimeGetSeconds(timeRange.start), CMTimeGetSeconds(CMTimeAdd(timeRange.duration, timeRange.start)));
	}
