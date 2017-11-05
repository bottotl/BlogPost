---
title: 实现末尾特殊折行的 Label
date: 2017-08-14 18:25:32
tags: 
categories: UI 开发
---

# 实现末尾特殊折行的 Label
单行效果图
![单行](http://wx3.sinaimg.cn/large/6b5f103fgy1fijcitlzr2j20d801zwex.jpg)

多行效果图
![多行](http://wx1.sinaimg.cn/large/6b5f103fgy1fijcir3530j20mb03rq4l.jpg)

<!-- more --> 
## 需求分析

文本分成两部分：正文、价格

1. 价格永远要露出
2. 价格是一个整体，不能显示成如下
 ![折行](http://wx3.sinaimg.cn/large/6b5f103fgy1fijciwderlj20g203umxh.jpg)

--- 

## 具体实现原理

### 使价格成为一个整体

先介绍一个神奇的字符：[\u2060](http://www.fileformat.info/info/unicode/char/2060/index.htm)。

这个字符可以确保它两边的字符能被识别为一个整体，不被折行

**原始字符:**

"标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题"
+"￥999"

**对原始字符串进行处理**

"标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题标题 "
+"￥[\u2060]9 [\u2060] 9 [\u2060] 9"

ps:[]是为了方便理解，不要把这两个符号也拼到字符串里面

### 价格永远要露出

问题转化为，把“省略号”替换成 “省略号+价格”
相关代码：

    for (int i = 0; i < MIN(CFArrayGetCount(lines), numberOfLines); i++) {
            CTLineRef line = CFArrayGetValueAtIndex(lines, i);
            CGPoint point = origins[i];
            if (i == numberOfLines - 1 && CFArrayGetCount(lines) > numberOfLines) { /// 最后一行 并且展示不下
                [self drawLastLine:line width:rect.size.width inContext:context origin:point];
            } else {
                CGContextSetTextPosition(context, point.x , point.y);
                CTLineDraw(line, context);
            }
     }
     
其他代码

    - (void)drawLastLine:(CTLineRef)line width:(CGFloat)width inContext:(CGContextRef)context origin:(CGPoint)point {
        CFRange textRange = CTLineGetStringRange(line);
        NSMutableAttributedString *lineAttributedString = [[NSMutableAttributedString alloc] initWithAttributedString:[self.attributedText attributedSubstringFromRange:NSMakeRange(textRange.location, (self.attributedText.length - textRange.location))]];
        NSDictionary *tokenAttributes = [self.attributedText attributesAtIndex:self.attributedText.length - 1
                                                                effectiveRange:NULL];
        NSAttributedString *tokenAttributedString = [self makePriceTruncationAttributedText:self.priceText tokenAttributes:tokenAttributes];
        [lineAttributedString appendAttributedString:tokenAttributedString];
        CTLineRef truncationToken = CTLineCreateWithAttributedString((__bridge CFAttributedStringRef)tokenAttributedString);
        CTLineRef truncationLine = CTLineCreateWithAttributedString((__bridge CFAttributedStringRef)lineAttributedString);
        CTLineTruncationType truncationType = kCTLineTruncationEnd;
        CTLineRef truncatedLine = CTLineCreateTruncatedLine(truncationLine, width, truncationType, truncationToken);
        CGContextSetTextPosition(context, point.x , point.y);
        CTLineDraw(truncatedLine, context);
    }
     
    - (NSAttributedString * _Nonnull)makePriceTruncationAttributedText:(NSString *)price tokenAttributes:(NSDictionary *)attributes {
        NSMutableAttributedString *attr = [[NSMutableAttributedString alloc] initWithString:@"\u2026\u2060 " attributes:attributes];
        [attr appendAttributedString:[self makePriceAttributedText:price]];
        return attr;
    }
     
    - (NSAttributedString * _Nonnull)makePriceAttributedText:(NSString *)price {
        NSDictionary *priceDic = @{NSFontAttributeName : [UIFont systemFontOfSize:14.f],
                                   NSForegroundColorAttributeName : [UIColor colorWithWhite:1 alpha:0.8]};
        return [[NSAttributedString alloc] initWithString:[self makePriceString:price] attributes:priceDic];
    }
     
    - (NSString * _Nonnull)makePriceString:(NSString *)price {
        if (!price.length) return @"";
        NSString *priceString = [NSString stringWithFormat:@" %@",price];
        NSMutableString *newPriceString = [[NSMutableString alloc] init];
        for (int i = 0; i < priceString.length; i++) {
            [newPriceString appendString:[priceString substringWithRange:NSMakeRange(i, 1)]];
            if (i < priceString.length) {
                [newPriceString appendString:@"\u2060"];
            }
        }
        return newPriceString.copy;
    }