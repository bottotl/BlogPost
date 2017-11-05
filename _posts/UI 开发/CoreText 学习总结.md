---
title: CoreText 学习总结
date: 2017-03-04 11:06:48
tags: [UILabel, iOS, CoreText]
categories: UI 开发
---

===

## 需求

#### 给出一段多行文本，要求给前面部分文字添加圆角边框（下面简称这个为标签），要求能够控制标签和文字之间的距离。

<!-- more --> 

---

## 最初方案

由于 AttributeString 自带的加边框的属性不支持圆角，加之开发时间比较紧，于是采用了：

UILabel->UIImage->NSTextAttachment->AttributeString

	+ (UIImage *)imageWithLabel:(UILabel *)label {
    UIGraphicsBeginImageContextWithOptions(label.frame.size, NO, [UIScreen mainScreen].scale);
    CGContextRef context = UIGraphicsGetCurrentContext();
    [label.layer renderInContext:context];
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return image;
	}


（NSTextAttachment + AttributeString) = FinalAttributeString 传给 UILabel ，交给 UILabel 去进行展示。


测试了一个极限情况：让标签文本很多
产生的问题：
tableView  去计算高度的时候消耗了大量的时间。
计算高度伪代码：

	new UILabel 
	UILabel.text =  FinalAttributeString
	UILabel sizeToFit
	return UILabel.height

总结：imageWithLabel 会导致大量的时间的浪费

-----
## 方案优化

使用 CoreText 去绘制文本，计算文本排版，获取需要绘制矩形的区域。

使用 Core Graphics 对矩形区域进行圆角边框的绘制
#### 大致过程

	get AttributeString 
	CTFrameSetter = CreateCTFrameSetter(AttributeString)
	CTFrame = CTFrameSetter(CGPath)
	
	// 处理 CTFrame -> CTLine -> CTRun
	origins[]（基准线坐标）= GetOrigns(CTFrame)
	
	Draw(CTFrame)/Draw(CTLine)/Draw(CTRun)
	

#### 类设计

- XXLabel:UIView 
- XXLayout
- AttributeString(XXBorder)

###### AttributeString(XXBorder)

扩展 AttributeString，可以添加边框的宽度、颜色等属性，到时候需要提交给 XXLabel。
###### XXLabel
模拟 UILabel 的属性，接收 AttributeString ，根据 attribute 绘制文字和边框
###### XXLayout
传入 AttributeString ，可以获取需要展示的高度。（主要在 TableView 算高 和 [XXLabel sizeToFit] 的时候使用）

主要绘制伪代码：

	 - (void)drawRect:(CGRect)rect {
	 	CTFrameSetter = CreateCTFrameSetter(AttributeString)
		CTFrame = CTFrameSetter(CGPath)
		
		origins[]（基准线坐标）= GetOrigns(CTFrame)
		Rects[]// 保存了边框的矩形
		for (origin in origins) {
			Temp Rect //
			CTRuns = GetRuns(CTLine)
			for (CTRun in CTRuns) {
				attribute = GetAttributes(CTRun)
				if (attribute 有边框属性){
					记录 CTRun 的 Rect。
					根据边框属性确定是否需要和 Temp Rect 合并
					把 Rect 保存到 Rects 中
				}
			}
		}
		Draw(CTFrame)
	 }


#### 导致的问题

很难控制标签和正文之间的距离
很难实现标签文本和正文实现水平居中

#### 解决方案

1. 在生成 AttributeString 的时候就通过插入空格（@"  标签文本  "）去实现控制标签和正文的距离
2. 在生成 AttributeString 中的标签文本前后分别插入 attachment 实现占位（attachment 被换行了的话会导致 bug ）
3. 根据 attribute 和标签的 CTRun 的坐标计算新坐标，一个个 CTRun 进行重新绘制。



#### 总结

1. 根据边框属性去计算需要绘制 CTRun 的位置，然后对其进行特殊处理，获取后面的 CTRun 的基准线，使得两者水平居中
2. 需要大量特殊判断，不利于扩展

## 目前方案

用 CoreText 绘制文字，标签使用一个 UIView ,贴在 View 中

#### 类设计

- AtrtributedStringProcesser : NSObject
- AttributeStringAttachment : NSObject
- RichTextDrawer : NSObject
- RichTextLabel : UIView
- RichTextLayout : NSObject

#### 流程：

1. 生成 Attachment ，表示需要占位的 Width，Ascend，Descend.
2. 生成 AtrtributedString， **文本作为 Identifier（到时候根据 Identifier 会去问 delegate 索取需要插入的 View）** ，把 Attachment 当做 attribute 插入。
3. 把 AtrtributedString 传给 RichTextLabel 
4. RichTextLabel 通过 AtrtributedStringProcesser 把 AtrtributedString 中的作为 Identifier 的文本过滤成一个字符（不能是空格回车之类的，这里会有坑）
5. RichTextLabel 通知 RichTextDrawer 去绘制 AtrtributedString
6. RichTextDrawer 处理 AtrtributedString ，根据 Identifier 去向 delegate 索取需要传入的 View，完成绘制。

#### 总结

- 只绘制文本，预留出空间给 View 进行展示
- 绘制效率高




