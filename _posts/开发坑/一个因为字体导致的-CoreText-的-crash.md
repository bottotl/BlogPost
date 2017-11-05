---
title: 一个因为字体导致的 CoreText 的 crash
date: 2017-08-14 15:13:56
tags:
categories:
- 开发坑
---

crash log

	Thread 0 Crashed:
	0 libsystem_kernel.dylib __pthread_kill (in libsystem_kernel.dylib) + 8
	1 libsystem_c.dylib abort (in libsystem_c.dylib) + 140
	2 libsystem_malloc.dylib _nano_vet_and_size_of_live (in libsystem_malloc.dylib) + 0
	3 libsystem_malloc.dylib nano_realloc (in libsystem_malloc.dylib) + 376
	4 libsystem_malloc.dylib malloc_zone_realloc (in libsystem_malloc.dylib) + 180
	5 CoreFoundation CFCharacterSetIntersect (in CoreFoundation) + 1696
	6 CoreText TComponentFont::CopyCharacterSet() const (in CoreText) + 724
	7 CoreText nw_path_set_reason_description (in libsystem_network.dylib) + 76
	8 CoreText TBaseFont::CharactersCovered(unsigned short const*, long) const (in CoreText) + 56
	9 CoreText TComponentFont::CharactersCovered(unsigned short const*, long) const (in CoreText) + 160
	10 CoreText TFontCascade::CreateFallback(__CTFont const*, __CFString const*, CTEmojiPolicy) const (in CoreText) + 976
	11 CoreText TGlyphEncoder::AppendUnmappedCharRun(unsigned int, TCFRef&, __CTFont const*, CFRange&, CFRange, TGlyphList&, TGlyphList&, TFontCascade const&, TGlyphEncoder::ClusterMatching, bool) (in CoreText) + 704
	12 CoreText TGlyphEncoder::RunUnicodeEncoderRecursively(unsigned int, TCFRef&&, __CTFont const*, CFRange, TGlyphList&, TGlyphList&, TFontCascade const*, TGlyphEncoder::ClusterMatching, bool) (in CoreText) + 1820
	13 CoreText TGlyphEncoder::AppendUnmappedCharRun(unsigned int, TCFRef&, __CTFont const*, CFRange&, CFRange, TGlyphList&, TGlyphList&, TFontCascade const&, TGlyphEncoder::ClusterMatching, bool) (in CoreText) + 1604
	14 CoreText TGlyphEncoder::RunUnicodeEncoderRecursively(unsigned int, TCFRef&&, __CTFont const*, CFRange, TGlyphList&, TGlyphList&, TFontCascade const*, TGlyphEncoder::ClusterMatching, bool) (in CoreText) + 1820
	15 CoreText CABackingStoreUpdate_ (in QuartzCore) + 996
	16 CoreText _nano_malloc_check_clear (in libsystem_malloc.dylib) + 412
	17 CoreText TTypesetterAttrString::Initialize(__CFAttributedString const*) (in CoreText) + 264
	18 CoreText TTypesetterAttrString::TTypesetterAttrString(__CFAttributedString const*) (in CoreText) + 108
	19 CoreText CTFramesetterCreateWithAttributedString (in CoreText) + 80
<!-- more --> 	
似乎是因为 CoreText 处理富文本的时候，如果给富文本设置 NSFontAttributeName 时使用了 UIFont 就会导致这个问题，修改成 CTFontRef 就可以了。
因为无法复现，所以讲代码进行了下面的替换

原始的代码：

	 + (NSAttributedString *)createTag:(NSString *)text font:(UIFont *)font {
	     if (!(text.length > 0)) {
	         return [[NSAttributedString alloc]initWithString:@""];
	     }
	    return [[NSAttributedString alloc]initWithString:text
	                                          attributes:@{NSFontAttributeName : font}];
	 }

修改后的代码：

	 + (NSAttributedString *)createTag:(NSString *)text font:(UIFont *)font {
	     if (!(text.length > 0)) {
	         return [[NSAttributedString alloc]initWithString:@""];
	     }
	         CTFontRef ctFont = CTFontCreateWithName((__bridge CFStringRef)font.fontName,
                                            font.pointSize,
                                            NULL);
    	NSAttributedString *attri = [[NSAttributedString alloc]initWithString:text
                                          attributes:@{(NSString *)kCTFontAttributeName : (__bridge id)ctFont}];
   	 	CFRelease(ctFont);
    	return attri;
	 }

观察了几个版本，没有继续发现 Crash