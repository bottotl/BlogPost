---
title: UILabel 省略号逻辑调研结果
date: 2017-03-04 11:06:48
tags: 
categories: UI 开发
---

|转义字符|含义|是否不会显示在屏幕上的字符|导致换行|
|:---|
|\a|响铃(BEL)|NO|NO|
|\b|退格(BS) ，将当前位置移到前一列|NO|NO|
|\f|换页(FF)，将当前位置移到下页开头|NO|YES|
|\n|换行(LF) ，将当前位置移到下一行开头|NO|YES|
|\r|回车(CR) ，将当前位置移到本行开头|NO|NO|
|\t|水平制表(HT) （跳到下一个TAB位置）|NO|NO|
|\v|垂直制表(VT)|NO|YES|
|\\|代表一个反斜线字符''\'|YES|NO|
|\'|代表一个单引号（撇号）字符|YES|NO|
|\"|代表一个双引号字符|YES|NO|
|\?|代表一个问号|YES|NO|
|\0|空字符(NULL)|NO|NO|

<!-- more --> 

##定义

不可见行：都又不可见字符组成的行

##省略号添加逻辑：
1. 最后一行为不可见行，在上方最近的第一个可见行中添加省略号
2. 添加省略号后如果显示不下，需要删除可见字符




#测试数据：\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a

结果

Printing description of lines:
<__NSCFArray 0x7fe7430a7530>(
<CTLine: 0x7fe7430b2920>{run count = 1, string range = (0, 28), width = 0, A/D/L = 13.3301/3.37695/0, glyph count = 28, runs = (

<CTRun: 0x7fe7430ac540>{string range = (0, 28), string = "\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a\a", attributes = {
    NSFont = "<UICTFont: 0x7fe7432719d0> font-family: \".SFUIText-Regular\"; font-weight: normal; font-style: normal; font-size: 14.00pt";
    NSParagraphStyle = "Alignment 4, LineSpacing 4, ParagraphSpacing 0, ParagraphSpacingBefore 0, HeadIndent 0, TailIndent 0, FirstLineHeadIndent 0, LineHeight 0/0, LineHeightMultiple 0, LineBreakMode 1, Tabs (\n    28L,\n    56L,\n    84L,\n    112L,\n    140L,\n    168L,\n    196L,\n    224L,\n    252L,\n    280L,\n    308L,\n    336L\n), DefaultTabInterval 0, Blocks (\n), Lists (\n), BaseWritingDirection -1, HyphenationFactor 0, TighteningForTruncation NO, HeaderLevel 0";
}}

)
}
)

#测试数据：\a\a\a\a\a\a\a\a\a\b\t\f\r\t\v\\\'\"\?\0

Printing description of lines:
<__NSCFArray 0x7fe890c2aed0>(
<CTLine: 0x7fe890c2cb10>{run count = 2, string range = (0, 12), width = 32.3611, A/D/L = 13.3301/3.37695/1.33583, glyph count = 12, runs = (

<CTRun: 0x7fe890c2e290>{string range = (0, 11), string = "\a\a\a\a\a\a\a\a\a\b\t", attributes = {
    NSFont = "<UICTFont: 0x7fe890e2e2c0> font-family: \".SFUIText-Regular\"; font-weight: normal; font-style: normal; font-size: 14.00pt";
    NSParagraphStyle = "Alignment 4, LineSpacing 4, ParagraphSpacing 0, ParagraphSpacingBefore 0, HeadIndent 0, TailIndent 0, FirstLineHeadIndent 0, LineHeight 0/0, LineHeightMultiple 0, LineBreakMode 1, Tabs (\n    28L,\n    56L,\n    84L,\n    112L,\n    140L,\n    168L,\n    196L,\n    224L,\n    252L,\n    280L,\n    308L,\n    336L\n), DefaultTabInterval 0, Blocks (\n), Lists (\n), BaseWritingDirection -1, HyphenationFactor 0, TighteningForTruncation NO, HeaderLevel 0";
}}


<CTRun: 0x7fe890c351d0>{string range = (11, 1), string = "\f", attributes = <CFBasicHash 0x7fe890fe4ac0 [0x10389ba40]>{type = mutable dict, count = 2,
entries =>
	1 : <CFString 0x109c41030 [0x10389ba40]>{contents = "NSParagraphStyle"} = Alignment 4, LineSpacing 4, ParagraphSpacing 0, ParagraphSpacingBefore 0, HeadIndent 0, TailIndent 0, FirstLineHeadIndent 0, LineHeight 0/0, LineHeightMultiple 0, LineBreakMode 1, Tabs (
    28L,
    56L,
    84L,
    112L,
    140L,
    168L,
    196L,
    224L,
    252L,
    280L,
    308L,
    336L
), DefaultTabInterval 0, Blocks (
), Lists (
), BaseWritingDirection -1, HyphenationFactor 0, TighteningForTruncation NO, HeaderLevel 0
	2 : <CFString 0x103b4f830 [0x10389ba40]>{contents = "NSFont"} = <CTFont: 0x7fe890c33760>{name = .HiraKakuInterface-W3, size = 14.000000, matrix = 0x0, descriptor = <CTFontDescriptor: 0x7fe890c087f0>{attributes = <CFBasicHash 0x7fe890c2de80 [0x10389ba40]>{type = mutable dict, count = 2,
entries =>
	1 : <CFString 0x103b55b70 [0x10389ba40]>{contents = "NSCTFontFeatureSettingsAttribute"} = (
        {
        CTFeatureSelectorIdentifier = 8;
        CTFeatureTypeIdentifier = 22;
    }
)
	2 : <CFString 0x103b557d0 [0x10389ba40]>{contents = "NSFontNameAttribute"} = <CFString 0x103b4e150 [0x10389ba40]>{contents = ".HiraKakuInterface-W3"}
}
>}}
}
}

)
},
<CTLine: 0x7fe890c161f0>{run count = 1, string range = (12, 1), width = 3.80078, A/D/L = 13.3301/3.37695/0, glyph count = 1, runs = (

<CTRun: 0x7fe890c30a90>{string range = (12, 1), string = "\r", attributes = {
    NSFont = "<UICTFont: 0x7fe890e2e2c0> font-family: \".SFUIText-Regular\"; font-weight: normal; font-style: normal; font-size: 14.00pt";
    NSParagraphStyle = "Alignment 4, LineSpacing 4, ParagraphSpacing 0, ParagraphSpacingBefore 0, HeadIndent 0, TailIndent 0, FirstLineHeadIndent 0, LineHeight 0/0, LineHeightMultiple 0, LineBreakMode 1, Tabs (\n    28L,\n    56L,\n    84L,\n    112L,\n    140L,\n    168L,\n    196L,\n    224L,\n    252L,\n    280L,\n    308L,\n    336L\n), DefaultTabInterval 0, Blocks (\n), Lists (\n), BaseWritingDirection -1, HyphenationFactor 0, TighteningForTruncation NO, HeaderLevel 0";
}}

)
},
<CTLine: 0x7fe890c2ef60>{run count = 2, string range = (13, 2), width = 32.3611, A/D/L = 13.3301/3.37695/1.33583, glyph count = 2, runs = (

<CTRun: 0x7fe890c2dc00>{string range = (13, 1), string = "\t", attributes = {
    NSFont = "<UICTFont: 0x7fe890e2e2c0> font-family: \".SFUIText-Regular\"; font-weight: normal; font-style: normal; font-size: 14.00pt";
    NSParagraphStyle = "Alignment 4, LineSpacing 4, ParagraphSpacing 0, ParagraphSpacingBefore 0, HeadIndent 0, TailIndent 0, FirstLineHeadIndent 0, LineHeight 0/0, LineHeightMultiple 0, LineBreakMode 1, Tabs (\n    28L,\n    56L,\n    84L,\n    112L,\n    140L,\n    168L,\n    196L,\n    224L,\n    252L,\n    280L,\n    308L,\n    336L\n), DefaultTabInterval 0, Blocks (\n), Lists (\n), BaseWritingDirection -1, HyphenationFactor 0, TighteningForTruncation NO, HeaderLevel 0";
}}


<CTRun: 0x7fe890c34d90>{string range = (14, 1), string = "\v", attributes = <CFBasicHash 0x7fe890fe4ac0 [0x10389ba40]>{type = mutable dict, count = 2,
entries =>
	1 : <CFString 0x109c41030 [0x10389ba40]>{contents = "NSParagraphStyle"} = Alignment 4, LineSpacing 4, ParagraphSpacing 0, ParagraphSpacingBefore 0, HeadIndent 0, TailIndent 0, FirstLineHeadIndent 0, LineHeight 0/0, LineHeightMultiple 0, LineBreakMode 1, Tabs (
    28L,
    56L,
    84L,
    112L,
    140L,
    168L,
    196L,
    224L,
    252L,
    280L,
    308L,
    336L
), DefaultTabInterval 0, Blocks (
), Lists (
), BaseWritingDirection -1, HyphenationFactor 0, TighteningForTruncation NO, HeaderLevel 0
	2 : <CFString 0x103b4f830 [0x10389ba40]>{contents = "NSFont"} = <CTFont: 0x7fe890c33760>{name = .HiraKakuInterface-W3, size = 14.000000, matrix = 0x0, descriptor = <CTFontDescriptor: 0x7fe890c087f0>{attributes = <CFBasicHash 0x7fe890c2de80 [0x10389ba40]>{type = mutable dict, count = 2,
entries =>
	1 : <CFString 0x103b55b70 [0x10389ba40]>{contents = "NSCTFontFeatureSettingsAttribute"} = (
        {
        CTFeatureSelectorIdentifier = 8;
        CTFeatureTypeIdentifier = 22;
    }
)
	2 : <CFString 0x103b557d0 [0x10389ba40]>{contents = "NSFontNameAttribute"} = <CFString 0x103b4e150 [0x10389ba40]>{contents = ".HiraKakuInterface-W3"}
}
>}}
}
}

)
},
<CTLine: 0x7fe890c2fa30>{run count = 1, string range = (15, 5), width = 21.1025, A/D/L = 13.3301/3.37695/0, glyph count = 5, runs = (

<CTRun: 0x7fe890c2dd30>{string range = (15, 5), string = "\'"?\x00", attributes = {
    NSFont = "<UICTFont: 0x7fe890e2e2c0> font-family: \".SFUIText-Regular\"; font-weight: normal; font-style: normal; font-size: 14.00pt";
    NSParagraphStyle = "Alignment 4, LineSpacing 4, ParagraphSpacing 0, ParagraphSpacingBefore 0, HeadIndent 0, TailIndent 0, FirstLineHeadIndent 0, LineHeight 0/0, LineHeightMultiple 0, LineBreakMode 1, Tabs (\n    28L,\n    56L,\n    84L,\n    112L,\n    140L,\n    168L,\n    196L,\n    224L,\n    252L,\n    280L,\n    308L,\n    336L\n), DefaultTabInterval 0, Blocks (\n), Lists (\n), BaseWritingDirection -1, HyphenationFactor 0, TighteningForTruncation NO, HeaderLevel 0";
}}

)
}
)

