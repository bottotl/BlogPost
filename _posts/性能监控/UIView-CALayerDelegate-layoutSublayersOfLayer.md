---
title: '[UIView(CALayerDelegate) layoutSublayersOfLayer:]'
date: 2017-11-25 16:05:04
tags:
category: 性能监控
---

简单猜测一下，如果有错误希望大家帮忙纠正。调用栈在最下面

## 1.准备

1.1 遍历了 `layer` 数组。并且调用了 `_layoutEngine`，发了一个 `performPendingChangeNotifications` 

1.2 和 window 通讯

    _applyInvocationsTo:window:

<!-- more --> 
## 2.Debug 相关的逻辑

中间有一些 `UIViewLayoutFeedbackLoopDebugger` 的东西

    _UIViewLayoutFeedbackLoopDebugger layoutFeedbackLoopDebugger

## 3.正式开始前的准备

    didEnterLayoutSublayersOfLayerForView:

3.1 跑一个循环
给每个 view 的 layoutEngine 、delegate、_hostsLayoutEngine 赋值

    for UIView._imminentLayoutSubviewsCount {
        layoutEngine = ？
        delegate = ？
        hostsLayoutEngine = ？
    }

3.2 通知 `vc` 和 `navigationer` 一些不知道干什么（我们不关心）的事情

    // ==== vc
    _viewControllerToNotifyOnLayoutSubviews
    _lastNotifiedTraitCollection
    _canSkipTraitsAndOverlayUpdatesForViewControllerToNotifyOnLayoutResetState
    _updateTraitsIfNecessary
    ……

## 4.真正Layout

4.1 LayoutEngine 

猜测和 AutoLayout 有关系

    _usesLayoutEngineHostingConstraints
    _resetLayoutEngineHostConstraints

4.2 view layoutSubviews

    willSendLayoutSubviewsToView:
    layoutSubviews
    didSendLayoutSubviewsToView

4.3 循环 4.1和4.2 一直到直系的子 View 完成布局

    _wantsReapplicationOfAutoLayoutWithLayoutDirtyOnEntry
    _updateConstraintsAsNecessaryAndApplyLayoutFromEngine
4.4 更新 SafeArea
    _updateSafeAreaInsets

## 5.结束布局前

5.1 通知 vc 

    willSendViewDidLayoutSubviewsToViewControllerOfView

vc 调用 viewDidLayoutSubviews

    didSendViewDidLayoutSubviewsToViewControllerOfView:

vc 的 embeddedDelegate 调用 viewControllerViewDidLayoutSubviews

    @protocol _UIViewControllerContentViewEmbedding
    - (void)viewController:(id)arg1 viewDidAppear:(_Bool)arg2;
    - (void)viewController:(id)arg1 viewDidDisappear:(_Bool)arg2;
    - (void)viewController:(id)arg1 viewWillAppear:(_Bool)arg2;
    - (void)viewController:(id)arg1 viewWillDisappear:(_Bool)arg2;
    - (void)viewControllerViewDidLayoutSubviews:(id)arg1;
    - (void)viewControllerViewWillLayoutSubviews:(id)arg1;
    @end

5.2 AutoLayout 相关逻辑

    _wantsReapplicationOfAutoLayoutWithLayoutDirtyOnEntry:
    _updateConstraintsAsNecessaryAndApplyLayoutFromEngine

5.3 转场动画相关（UIPresentationController）

    containerViewDidLayoutSubviews
    
5.4 Debug 相关

    _toolsDebugAlignmentRects
    _alignmentDebuggingOverlayCreateIfNecessary:
    _toolsDebugColorViewBounds
    _colorViewBoundsOverlayCreateIfNecessary:
    _toolsDebugShouldDetectClippedViews

5.5 圆角相关的逻辑

    detectAndHandleClippedView
## 6.结束布局

6.1 根据布局时间计算这次的 layout 的 hash 值。

    _validateLayoutHashHasChangedWithLayoutTime:
这里就有点奇怪了，拿到布局的时间能用来做什么？

6.2 正式退出
    willExitLayoutSublayersOfLayerForView

## tableView 的调用栈为例

    UIKit`-[UIView(CALayerDelegate) layoutSublayersOfLayer:]:
        0x1029ee03a <+0>:    pushq  %rbp
        0x1029ee03b <+1>:    movq   %rsp, %rbp
        0x1029ee03e <+4>:    pushq  %r15
        0x1029ee040 <+6>:    pushq  %r14
        0x1029ee042 <+8>:    pushq  %r13
        0x1029ee044 <+10>:   pushq  %r12
        0x1029ee046 <+12>:   pushq  %rbx
        0x1029ee047 <+13>:   subq   $0x118, %rsp              ; imm = 0x118 
        0x1029ee04e <+20>:   movq   %rdx, %r14
        0x1029ee051 <+23>:   movq   %rdi, %r13
        0x1029ee054 <+26>:   movq   0x109bc6d(%rip), %rax     ; (void *)0x000000010a39e070: __stack_chk_guard
        0x1029ee05b <+33>:   movq   (%rax), %rax
        0x1029ee05e <+36>:   movq   %rax, -0x30(%rbp)
        0x1029ee062 <+40>:   movq   0x1463da7(%rip), %rax     ; UIView._layer
        0x1029ee069 <+47>:   movq   (%r13,%rax), %rdi
        0x1029ee06e <+52>:   movq   0x1419b73(%rip), %rsi     ; "needsLayout"
        0x1029ee075 <+59>:   movq   0x109c58c(%rip), %r15     ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee07c <+66>:   callq  *%r15
        0x1029ee07f <+69>:   movl   %eax, %r12d
        0x1029ee082 <+72>:   movq   0x1419eff(%rip), %rsi     ; "_shouldSkipNormalLayoutForSakeOfTemplateLayout"
        0x1029ee089 <+79>:   movq   %r13, %rdi
        0x1029ee08c <+82>:   callq  *%r15
        0x1029ee08f <+85>:   testb  %al, %al
        0x1029ee091 <+87>:   jne    0x1029ee968               ; <+2350>
        0x1029ee097 <+93>:   movq   0x1463b1a(%rip), %rbx     ; UIView._viewFlags
        0x1029ee09e <+100>:  movq   (%r13,%rbx), %rax
        0x1029ee0a3 <+105>:  movq   %rax, %rcx
        0x1029ee0a6 <+108>:  notq   %rcx
        0x1029ee0a9 <+111>:  movabsq $0x60000000000000, %rdx   ; imm = 0x60000000000000 
        0x1029ee0b3 <+121>:  testq  %rdx, %rcx
        0x1029ee0b6 <+124>:  jne    0x1029ee19a               ; <+352>
        0x1029ee0bc <+130>:  btq    $0x31, %rax
        0x1029ee0c1 <+135>:  jb     0x1029ee0fc               ; <+194>
        0x1029ee0c3 <+137>:  movq   0x1418646(%rip), %rsi     ; "_layoutEngine"
        0x1029ee0ca <+144>:  movq   %r13, %rdi
        0x1029ee0cd <+147>:  callq  *0x109c535(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee0d3 <+153>:  movq   %rax, %rdi
        0x1029ee0d6 <+156>:  callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee0db <+161>:  testq  %rax, %rax
        0x1029ee0de <+164>:  jne    0x1029ee191               ; <+343>
        0x1029ee0e4 <+170>:  movq   0x1418dcd(%rip), %rsi     ; "_hostsLayoutEngine"
        0x1029ee0eb <+177>:  movq   %r13, %rdi
        0x1029ee0ee <+180>:  callq  *0x109c514(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee0f4 <+186>:  testb  %al, %al
        0x1029ee0f6 <+188>:  jne    0x1029ee19a               ; <+352>
        0x1029ee0fc <+194>:  movq   0x8(%r13,%rbx), %rax
        0x1029ee101 <+199>:  testb  $0x1, %al
        0x1029ee103 <+201>:  je     0x1029ee968               ; <+2350>
        0x1029ee109 <+207>:  movq   0x1418600(%rip), %rsi     ; "_layoutEngine"
        0x1029ee110 <+214>:  movq   %r13, %rdi
        0x1029ee113 <+217>:  callq  *%r15
        0x1029ee116 <+220>:  movq   %rax, %rdi
        0x1029ee119 <+223>:  callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee11e <+228>:  movq   %r15, %rbx
        0x1029ee121 <+231>:  movq   %rax, %r15
        0x1029ee124 <+234>:  movq   0x1419e55(%rip), %rsi     ; "performPendingChangeNotifications"
        0x1029ee12b <+241>:  movq   %r15, %rdi
        0x1029ee12e <+244>:  callq  *%rbx
        0x1029ee130 <+246>:  movq   %r15, %rdi
        0x1029ee133 <+249>:  movq   %rbx, %r15
        0x1029ee136 <+252>:  movq   0x1463a7b(%rip), %rbx     ; UIView._viewFlags
        0x1029ee13d <+259>:  callq  *0x109c4cd(%rip)          ; (void *)0x00000001050f6cc0: objc_release
        0x1029ee143 <+265>:  movq   0x10(%r13,%rbx), %rax
        0x1029ee148 <+270>:  movq   (%r13,%rbx), %rcx
        0x1029ee14d <+275>:  movq   0x8(%r13,%rbx), %rdx
        0x1029ee152 <+280>:  andq   $-0x2, %rdx
        0x1029ee156 <+284>:  movq   %rcx, (%r13,%rbx)
        0x1029ee15b <+289>:  movq   %rax, 0x10(%r13,%rbx)
        0x1029ee160 <+294>:  movq   %rdx, 0x8(%r13,%rbx)
        0x1029ee165 <+299>:  btq    $0x31, %rcx
        0x1029ee16a <+304>:  jb     0x1029ee968               ; <+2350>
        0x1029ee170 <+310>:  movq   %r13, %rdi
        0x1029ee173 <+313>:  movq   0x1418596(%rip), %rsi     ; "_layoutEngine"
        0x1029ee17a <+320>:  callq  *0x109c488(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee180 <+326>:  movq   %rax, %rdi
        0x1029ee183 <+329>:  callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee188 <+334>:  testq  %rax, %rax
        0x1029ee18b <+337>:  je     0x1029ee31c               ; <+738>
        0x1029ee191 <+343>:  movq   %rax, %rdi
        0x1029ee194 <+346>:  callq  *0x109c476(%rip)          ; (void *)0x00000001050f6cc0: objc_release
        0x1029ee19a <+352>:  movq   0x10(%r13,%rbx), %rax
        0x1029ee19f <+357>:  movq   (%r13,%rbx), %rcx
        0x1029ee1a4 <+362>:  movq   0x8(%r13,%rbx), %rdx
        0x1029ee1a9 <+367>:  testb  $0x40, %dl
        0x1029ee1ac <+370>:  jne    0x1029ee2a3               ; <+617>
        0x1029ee1b2 <+376>:  btq    $0x25, %rcx
        0x1029ee1b7 <+381>:  jae    0x1029ee22e               ; <+500>
        0x1029ee1b9 <+383>:  movabsq $-0x2000000001, %rsi      ; imm = 0xFFFFFFDFFFFFFFFF 
        0x1029ee1c3 <+393>:  andq   %rsi, %rcx
        0x1029ee1c6 <+396>:  movq   %rdx, 0x8(%r13,%rbx)
        0x1029ee1cb <+401>:  movq   %rcx, (%r13,%rbx)
        0x1029ee1d0 <+406>:  movq   %rax, 0x10(%r13,%rbx)
        0x1029ee1d5 <+411>:  movq   0x1459f64(%rip), %rax     ; (void *)0x0000000103e8bb30: _UIAppearance
        0x1029ee1dc <+418>:  movq   %rax, -0xb8(%rbp)
        0x1029ee1e3 <+425>:  movq   0x14125ee(%rip), %rsi     ; "window"
        0x1029ee1ea <+432>:  movq   %r13, %rdi
        0x1029ee1ed <+435>:  callq  *%r15
        0x1029ee1f0 <+438>:  movq   %rax, %rdi
        0x1029ee1f3 <+441>:  callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee1f8 <+446>:  movl   %r12d, %ebx
        0x1029ee1fb <+449>:  movq   %r15, %r12
        0x1029ee1fe <+452>:  movq   %rax, %r15
        0x1029ee201 <+455>:  movq   0x141a170(%rip), %rsi     ; "_applyInvocationsTo:window:"
        0x1029ee208 <+462>:  movq   -0xb8(%rbp), %rdi
        0x1029ee20f <+469>:  movq   %r13, %rdx
        0x1029ee212 <+472>:  movq   %r15, %rcx
        0x1029ee215 <+475>:  callq  *%r12
        0x1029ee218 <+478>:  movq   %r15, %rdi
        0x1029ee21b <+481>:  movq   %r12, %r15
        0x1029ee21e <+484>:  movl   %ebx, %r12d
        0x1029ee221 <+487>:  movq   0x1463990(%rip), %rbx     ; UIView._viewFlags
        0x1029ee228 <+494>:  callq  *0x109c3e2(%rip)          ; (void *)0x00000001050f6cc0: objc_release
        0x1029ee22e <+500>:  movq   0x1463bdb(%rip), %rax     ; UIView._layer
        0x1029ee235 <+507>:  cmpq   %r14, (%r13,%rax)
        0x1029ee23a <+512>:  jne    0x1029ee968               ; <+2350>
        0x1029ee240 <+518>:  movq   0x145a541(%rip), %rdi     ; (void *)0x0000000103e8ad98: _UIViewLayoutFeedbackLoopDebugger
        0x1029ee247 <+525>:  movq   0x1419692(%rip), %rsi     ; "layoutFeedbackLoopDebugger"
        0x1029ee24e <+532>:  callq  *%r15
        0x1029ee251 <+535>:  movq   %rax, %rdi
        0x1029ee254 <+538>:  callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee259 <+543>:  movq   0x141a458(%rip), %rsi     ; "didEnterLayoutSublayersOfLayerForView:"
        0x1029ee260 <+550>:  movq   %rax, -0xb8(%rbp)
        0x1029ee267 <+557>:  movq   %rax, %rdi
        0x1029ee26a <+560>:  movq   %r13, %rdx
        0x1029ee26d <+563>:  callq  *%r15
        0x1029ee270 <+566>:  movq   0x1463c29(%rip), %rax     ; UIView._imminentLayoutSubviewsCount
        0x1029ee277 <+573>:  incq   (%r13,%rax)
        0x1029ee27c <+578>:  btq    $0x35, (%r13,%rbx)
        0x1029ee283 <+585>:  movb   %r12b, -0xd0(%rbp)
        0x1029ee28a <+592>:  jb     0x1029ee2be               ; <+644>
        0x1029ee28c <+594>:  xorl   %eax, %eax
        0x1029ee28e <+596>:  movq   %rax, -0xc8(%rbp)
        0x1029ee295 <+603>:  xorl   %eax, %eax
        0x1029ee297 <+605>:  movq   %rax, -0xc0(%rbp)
        0x1029ee29e <+612>:  jmp    0x1029ee34e               ; <+788>
        0x1029ee2a3 <+617>:  orq    $0x100, %rdx              ; imm = 0x100 
        0x1029ee2aa <+624>:  movq   %rcx, (%r13,%rbx)
        0x1029ee2af <+629>:  movq   %rdx, 0x8(%r13,%rbx)
        0x1029ee2b4 <+634>:  movq   %rax, 0x10(%r13,%rbx)
        0x1029ee2b9 <+639>:  jmp    0x1029ee968               ; <+2350>
        0x1029ee2be <+644>:  movq   0x141844b(%rip), %rsi     ; "_layoutEngine"
        0x1029ee2c5 <+651>:  movq   %r13, %rdi
        0x1029ee2c8 <+654>:  callq  *%r15
        0x1029ee2cb <+657>:  movq   %rax, %rdi
        0x1029ee2ce <+660>:  callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee2d3 <+665>:  movq   %r15, %rcx
        0x1029ee2d6 <+668>:  movq   %rax, %r15
        0x1029ee2d9 <+671>:  movq   0x14128e0(%rip), %rsi     ; "delegate"
        0x1029ee2e0 <+678>:  movq   %r15, %rdi
        0x1029ee2e3 <+681>:  movq   %rcx, %r12
        0x1029ee2e6 <+684>:  callq  *%rcx
        0x1029ee2e8 <+686>:  movq   %rax, %rdi
        0x1029ee2eb <+689>:  callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee2f0 <+694>:  movq   %rax, %r14
        0x1029ee2f3 <+697>:  movq   %r15, %rdi
        0x1029ee2f6 <+700>:  callq  *0x109c314(%rip)          ; (void *)0x00000001050f6cc0: objc_release
        0x1029ee2fc <+706>:  testq  %r14, %r14
        0x1029ee2ff <+709>:  je     0x1029ee339               ; <+767>
        0x1029ee301 <+711>:  movq   %r14, %rax
        0x1029ee304 <+714>:  movq   %rax, -0xc0(%rbp)
        0x1029ee30b <+721>:  orb    $0x1, 0x8(%r14,%rbx)
        0x1029ee311 <+727>:  movb   $0x1, %al
        0x1029ee313 <+729>:  movq   %rax, -0xc8(%rbp)
        0x1029ee31a <+736>:  jmp    0x1029ee34b               ; <+785>
        0x1029ee31c <+738>:  movq   0x1418b95(%rip), %rsi     ; "_hostsLayoutEngine"
        0x1029ee323 <+745>:  movq   %r13, %rdi
        0x1029ee326 <+748>:  callq  *0x109c2dc(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee32c <+754>:  testb  %al, %al
        0x1029ee32e <+756>:  jne    0x1029ee19a               ; <+352>
        0x1029ee334 <+762>:  jmp    0x1029ee968               ; <+2350>
        0x1029ee339 <+767>:  xorl   %eax, %eax
        0x1029ee33b <+769>:  movq   %rax, -0xc8(%rbp)
        0x1029ee342 <+776>:  xorl   %eax, %eax
        0x1029ee344 <+778>:  movq   %rax, -0xc0(%rbp)
        0x1029ee34b <+785>:  movq   %r12, %r15
        0x1029ee34e <+788>:  leaq   0x14c9197(%rip), %rax     ; __UILogIdenticalLayouts
        0x1029ee355 <+795>:  cmpb   $0x0, (%rax)
        0x1029ee358 <+798>:  je     0x1029ee361               ; <+807>
        0x1029ee35a <+800>:  callq  0x10379b1ac               ; symbol stub for: CFAbsoluteTimeGetCurrent
        0x1029ee35f <+805>:  jmp    0x1029ee365               ; <+811>
        0x1029ee361 <+807>:  xorpd  %xmm0, %xmm0
        0x1029ee365 <+811>:  movsd  %xmm0, -0xe8(%rbp)
        0x1029ee36d <+819>:  movq   0x10(%r13,%rbx), %rax
        0x1029ee372 <+824>:  movq   (%r13,%rbx), %rcx
        0x1029ee377 <+829>:  movq   0x8(%r13,%rbx), %rdx
        0x1029ee37c <+834>:  btq    $0x36, %rcx
        0x1029ee381 <+839>:  jb     0x1029ee398               ; <+862>
        0x1029ee383 <+841>:  movabsq $0x21000000000000, %rsi   ; imm = 0x21000000000000 
        0x1029ee38d <+851>:  movq   %rcx, %rdi
        0x1029ee390 <+854>:  andq   %rsi, %rdi
        0x1029ee393 <+857>:  xorq   %rsi, %rdi
        0x1029ee396 <+860>:  jne    0x1029ee3b4               ; <+890>
        0x1029ee398 <+862>:  movabsq $0x100000000000000, %rsi  ; imm = 0x100000000000000 
        0x1029ee3a2 <+872>:  orq    %rsi, %rcx
        0x1029ee3a5 <+875>:  movq   %rdx, 0x8(%r13,%rbx)
        0x1029ee3aa <+880>:  movq   %rcx, (%r13,%rbx)
        0x1029ee3af <+885>:  movq   %rax, 0x10(%r13,%rbx)
        0x1029ee3b4 <+890>:  movq   0x1419245(%rip), %rsi     ; "_viewControllerToNotifyOnLayoutSubviews"
        0x1029ee3bb <+897>:  movq   %r13, %rdi
        0x1029ee3be <+900>:  callq  *%r15
        0x1029ee3c1 <+903>:  movq   %rax, %rdi
        0x1029ee3c4 <+906>:  callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee3c9 <+911>:  movq   %rax, %r14
        0x1029ee3cc <+914>:  movq   0x1419b35(%rip), %rsi     ; "_lastNotifiedTraitCollection"
        0x1029ee3d3 <+921>:  movq   %r14, %rdi
        0x1029ee3d6 <+924>:  callq  *%r15
        0x1029ee3d9 <+927>:  movq   %rax, %rdi
        0x1029ee3dc <+930>:  callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee3e1 <+935>:  movq   %rax, -0xf0(%rbp)
        0x1029ee3e8 <+942>:  testq  %r14, %r14
        0x1029ee3eb <+945>:  je     0x1029ee4da               ; <+1184>
        0x1029ee3f1 <+951>:  movq   0x141a2c8(%rip), %rsi     ; "_canSkipTraitsAndOverlayUpdatesForViewControllerToNotifyOnLayoutResetState:"
        0x1029ee3f8 <+958>:  movl   $0x1, %edx
        0x1029ee3fd <+963>:  movq   %r13, %rdi
        0x1029ee400 <+966>:  callq  *0x109c202(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee406 <+972>:  testb  %al, %al
        0x1029ee408 <+974>:  jne    0x1029ee467               ; <+1069>
        0x1029ee40a <+976>:  movq   0x141a2b7(%rip), %rsi     ; "_updateTraitsIfNecessary"
        0x1029ee411 <+983>:  movq   %r14, %rdi
        0x1029ee414 <+986>:  callq  *%r15
        0x1029ee417 <+989>:  movq   0x141a2b2(%rip), %rsi     ; "_viewDelegateContentOverlayInsetsAreClean"
        0x1029ee41e <+996>:  movq   %r13, %rdi
        0x1029ee421 <+999>:  callq  *%r15
        0x1029ee424 <+1002>: testb  %al, %al
        0x1029ee426 <+1004>: je     0x1029ee457               ; <+1053>
        0x1029ee428 <+1006>: movq   0x1419139(%rip), %rsi     ; "navigationController"
        0x1029ee42f <+1013>: movq   %r14, %rdi
        0x1029ee432 <+1016>: callq  *0x109c1d0(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee438 <+1022>: movq   %rax, %rdi
        0x1029ee43b <+1025>: callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee440 <+1030>: movq   %r15, %r12
        0x1029ee443 <+1033>: movq   %rax, %r15
        0x1029ee446 <+1036>: movq   %r15, %rdi
        0x1029ee449 <+1039>: callq  *0x109c1c1(%rip)          ; (void *)0x00000001050f6cc0: objc_release
        0x1029ee44f <+1045>: testq  %r15, %r15
        0x1029ee452 <+1048>: movq   %r12, %r15
        0x1029ee455 <+1051>: je     0x1029ee467               ; <+1069>
        0x1029ee457 <+1053>: movq   0x141908a(%rip), %rsi     ; "_updateContentOverlayInsetsFromParentIfNecessary"
        0x1029ee45e <+1060>: movq   %r14, %rdi
        0x1029ee461 <+1063>: callq  *0x109c1a1(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee467 <+1069>: movq   0x141a26a(%rip), %rsi     ; "willSendViewWillLayoutSubviewsToViewControllerOfView:"
        0x1029ee46e <+1076>: movq   -0xb8(%rbp), %r12
        0x1029ee475 <+1083>: movq   %r12, %rdi
        0x1029ee478 <+1086>: movq   %r13, %rdx
        0x1029ee47b <+1089>: callq  *%r15
        0x1029ee47e <+1092>: movq   0x141a25b(%rip), %rsi     ; "viewWillLayoutSubviews"
        0x1029ee485 <+1099>: movq   %r14, %rdi
        0x1029ee488 <+1102>: callq  *%r15
        0x1029ee48b <+1105>: movq   0x141a256(%rip), %rsi     ; "didSendViewWillLayoutSubviewsToViewControllerOfView:"
        0x1029ee492 <+1112>: movq   %r12, %rdi
        0x1029ee495 <+1115>: movq   %r13, %rdx
        0x1029ee498 <+1118>: callq  *%r15
        0x1029ee49b <+1121>: movq   0x141a24e(%rip), %rsi     ; "_embeddedDelegate"
        0x1029ee4a2 <+1128>: movq   %r14, %rdi
        0x1029ee4a5 <+1131>: movq   %r15, %r12
        0x1029ee4a8 <+1134>: callq  *%r15
        0x1029ee4ab <+1137>: movq   %rax, %rdi
        0x1029ee4ae <+1140>: callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee4b3 <+1145>: movq   %rax, %r15
        0x1029ee4b6 <+1148>: testq  %r15, %r15
        0x1029ee4b9 <+1151>: je     0x1029ee4ce               ; <+1172>
        0x1029ee4bb <+1153>: movq   0x141a236(%rip), %rsi     ; "viewControllerViewWillLayoutSubviews:"
        0x1029ee4c2 <+1160>: movq   %r15, %rdi
        0x1029ee4c5 <+1163>: movq   %r14, %rdx
        0x1029ee4c8 <+1166>: callq  *0x109c13a(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee4ce <+1172>: movq   %r15, %rdi
        0x1029ee4d1 <+1175>: callq  *0x109c139(%rip)          ; (void *)0x00000001050f6cc0: objc_release
        0x1029ee4d7 <+1181>: movq   %r12, %r15
        0x1029ee4da <+1184>: movq   %r14, -0xd8(%rbp)
        0x1029ee4e1 <+1191>: movq   0x1419588(%rip), %rsi     ; "_presentationControllerToNotifyOnLayoutSubviews"
        0x1029ee4e8 <+1198>: movq   %r13, %rdi
        0x1029ee4eb <+1201>: callq  *0x109c117(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee4f1 <+1207>: movq   %rax, %rdi
        0x1029ee4f4 <+1210>: callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee4f9 <+1215>: movq   %rax, %r14
        0x1029ee4fc <+1218>: testq  %r14, %r14
        0x1029ee4ff <+1221>: je     0x1029ee511               ; <+1239>
        0x1029ee501 <+1223>: movq   0x141a1f8(%rip), %rsi     ; "_containerViewWillLayoutSubviews"
        0x1029ee508 <+1230>: movq   %r14, %rdi
        0x1029ee50b <+1233>: callq  *0x109c0f7(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee511 <+1239>: movq   -0xd8(%rbp), %rax
        0x1029ee518 <+1246>: movq   %r14, -0xf8(%rbp)
        0x1029ee51f <+1253>: orq    %r14, %rax
        0x1029ee522 <+1256>: je     0x1029ee576               ; <+1340>
        0x1029ee524 <+1258>: movabsq $0x800000000, %rax        ; imm = 0x800000000 
        0x1029ee52e <+1268>: andq   0x8(%r13,%rbx), %rax
        0x1029ee533 <+1273>: je     0x1029ee576               ; <+1340>
        0x1029ee535 <+1275>: movq   0x141280c(%rip), %rsi     ; "traitCollection"
        0x1029ee53c <+1282>: movq   %r13, %rdi
        0x1029ee53f <+1285>: callq  *%r15
        0x1029ee542 <+1288>: movq   %rax, %rdi
        0x1029ee545 <+1291>: callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee54a <+1296>: movq   %r15, %r12
        0x1029ee54d <+1299>: movq   %rax, %r15
        0x1029ee550 <+1302>: movq   0x14199b9(%rip), %rsi     ; "_processDidChangeRecursivelyFromOldTraits:toCurrentTraits:forceNotification:"
        0x1029ee557 <+1309>: xorl   %r8d, %r8d
        0x1029ee55a <+1312>: movq   %r13, %rdi
        0x1029ee55d <+1315>: movq   -0xf0(%rbp), %rdx
        0x1029ee564 <+1322>: movq   %r15, %rcx
        0x1029ee567 <+1325>: callq  *%r12
        0x1029ee56a <+1328>: movq   %r15, %rdi
        0x1029ee56d <+1331>: movq   %r12, %r15
        0x1029ee570 <+1334>: callq  *0x109c09a(%rip)          ; (void *)0x00000001050f6cc0: objc_release
        0x1029ee576 <+1340>: movq   %r13, %rdi
        0x1029ee579 <+1343>: callq  0x10348aa40               ; _UILayoutEngineSolutionIsInRationalEdgesConsultingDelegate
        0x1029ee57e <+1348>: testb  %al, %al
        0x1029ee580 <+1350>: je     0x1029ee5a6               ; <+1388>
        0x1029ee582 <+1352>: movq   0x141a17f(%rip), %rsi     ; "_usesLayoutEngineHostingConstraints"
        0x1029ee589 <+1359>: movq   %r13, %rdi
        0x1029ee58c <+1362>: callq  *0x109c076(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee592 <+1368>: testb  %al, %al
        0x1029ee594 <+1370>: je     0x1029ee5a6               ; <+1388>
        0x1029ee596 <+1372>: movq   0x14196e3(%rip), %rsi     ; "_resetLayoutEngineHostConstraints"
        0x1029ee59d <+1379>: movq   %r13, %rdi
        0x1029ee5a0 <+1382>: callq  *0x109c062(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee5a6 <+1388>: movq   0x141a163(%rip), %rsi     ; "willSendLayoutSubviewsToView:"
        0x1029ee5ad <+1395>: movq   -0xb8(%rbp), %rbx
        0x1029ee5b4 <+1402>: movq   %rbx, %rdi
        0x1029ee5b7 <+1405>: movq   %r13, %rdx
        0x1029ee5ba <+1408>: callq  *%r15
        0x1029ee5bd <+1411>: movq   %r15, %r12
        0x1029ee5c0 <+1414>: movq   0x1463969(%rip), %r15     ; UIView._layoutSubviewsCount
        0x1029ee5c7 <+1421>: incq   (%r13,%r15)
        0x1029ee5cc <+1426>: movq   0x1412465(%rip), %rsi     ; "layoutSubviews"
        0x1029ee5d3 <+1433>: movq   %r13, %rdi
        0x1029ee5d6 <+1436>: callq  *%r12
        0x1029ee5d9 <+1439>: decq   (%r13,%r15)
        0x1029ee5de <+1444>: movq   0x14638bb(%rip), %rax     ; UIView._imminentLayoutSubviewsCount
        0x1029ee5e5 <+1451>: decq   (%r13,%rax)
        0x1029ee5ea <+1456>: movq   0x141a127(%rip), %rsi     ; "didSendLayoutSubviewsToView:"
        0x1029ee5f1 <+1463>: movq   %rbx, %rdi
        0x1029ee5f4 <+1466>: movq   %r13, %rdx
        0x1029ee5f7 <+1469>: callq  *%r12
        0x1029ee5fa <+1472>: movq   0x141a11f(%rip), %rsi     ; "_wantsReapplicationOfAutoLayoutWithLayoutDirtyOnEntry:"
        0x1029ee601 <+1479>: movzbl -0xd0(%rbp), %edx
        0x1029ee608 <+1486>: movq   %r13, %rdi
        0x1029ee60b <+1489>: movl   %edx, -0xdc(%rbp)
        0x1029ee611 <+1495>: callq  *%r12
        0x1029ee614 <+1498>: testb  %al, %al
        0x1029ee616 <+1500>: je     0x1029ee628               ; <+1518>
        0x1029ee618 <+1502>: movq   0x1419a29(%rip), %rsi     ; "_updateConstraintsAsNecessaryAndApplyLayoutFromEngine"
        0x1029ee61f <+1509>: movq   %r13, %rdi
        0x1029ee622 <+1512>: callq  *0x109bfe0(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee628 <+1518>: xorpd  %xmm0, %xmm0
        0x1029ee62c <+1522>: leaq   -0x140(%rbp), %r15
        0x1029ee633 <+1529>: movapd %xmm0, 0x30(%r15)
        0x1029ee639 <+1535>: movapd %xmm0, 0x20(%r15)
        0x1029ee63f <+1541>: movapd %xmm0, 0x10(%r15)
        0x1029ee645 <+1547>: movapd %xmm0, (%r15)
        0x1029ee64a <+1552>: movq   0x1412eaf(%rip), %rsi     ; "subviews"
        0x1029ee651 <+1559>: movq   %r13, -0xd0(%rbp)
        0x1029ee658 <+1566>: movq   %r13, %rdi
        0x1029ee65b <+1569>: movq   0x109bfa6(%rip), %rax     ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee662 <+1576>: movq   %rax, %rbx
        0x1029ee665 <+1579>: callq  *%rbx
        0x1029ee667 <+1581>: movq   %rax, %rdi
        0x1029ee66a <+1584>: callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee66f <+1589>: movq   %rax, %r14
        0x1029ee672 <+1592>: movq   0x14125b7(%rip), %rsi     ; "countByEnumeratingWithState:objects:count:"
        0x1029ee679 <+1599>: leaq   -0xb0(%rbp), %rcx
        0x1029ee680 <+1606>: movl   $0x10, %r8d
        0x1029ee686 <+1612>: movq   %r14, %rdi
        0x1029ee689 <+1615>: movq   %r15, %rdx
        0x1029ee68c <+1618>: callq  *%rbx
        0x1029ee68e <+1620>: movq   %rax, %r12
        0x1029ee691 <+1623>: testq  %r12, %r12
        0x1029ee694 <+1626>: je     0x1029ee70a               ; <+1744>
        0x1029ee696 <+1628>: leaq   -0x140(%rbp), %rax
        0x1029ee69d <+1635>: movq   0x10(%rax), %rax
        0x1029ee6a1 <+1639>: movq   (%rax), %r13
        0x1029ee6a4 <+1642>: movq   0x1418e65(%rip), %rbx     ; "_updateSafeAreaInsets"
        0x1029ee6ab <+1649>: xorl   %r15d, %r15d
        0x1029ee6ae <+1652>: movq   -0x130(%rbp), %rax
        0x1029ee6b5 <+1659>: cmpq   %r13, (%rax)
        0x1029ee6b8 <+1662>: je     0x1029ee6c2               ; <+1672>
        0x1029ee6ba <+1664>: movq   %r14, %rdi
        0x1029ee6bd <+1667>: callq  0x10379c778               ; symbol stub for: objc_enumerationMutation
        0x1029ee6c2 <+1672>: movq   -0x138(%rbp), %rax
        0x1029ee6c9 <+1679>: movq   (%rax,%r15,8), %rdi
        0x1029ee6cd <+1683>: movq   %rbx, %rsi
        0x1029ee6d0 <+1686>: callq  *0x109bf32(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee6d6 <+1692>: incq   %r15
        0x1029ee6d9 <+1695>: cmpq   %r12, %r15
        0x1029ee6dc <+1698>: jb     0x1029ee6ae               ; <+1652>
        0x1029ee6de <+1700>: movl   $0x10, %r8d
        0x1029ee6e4 <+1706>: movq   %r14, %rdi
        0x1029ee6e7 <+1709>: movq   0x1412542(%rip), %rsi     ; "countByEnumeratingWithState:objects:count:"
        0x1029ee6ee <+1716>: leaq   -0x140(%rbp), %rdx
        0x1029ee6f5 <+1723>: leaq   -0xb0(%rbp), %rcx
        0x1029ee6fc <+1730>: callq  *0x109bf06(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee702 <+1736>: movq   %rax, %r12
        0x1029ee705 <+1739>: testq  %r12, %r12
        0x1029ee708 <+1742>: jne    0x1029ee6a4               ; <+1642>
        0x1029ee70a <+1744>: movq   %r14, %rdi
        0x1029ee70d <+1747>: callq  *0x109befd(%rip)          ; (void *)0x00000001050f6cc0: objc_release
        0x1029ee713 <+1753>: movq   -0xd8(%rbp), %r13
        0x1029ee71a <+1760>: testq  %r13, %r13
        0x1029ee71d <+1763>: movq   -0xd0(%rbp), %r15
        0x1029ee724 <+1770>: je     0x1029ee7e5               ; <+1963>
        0x1029ee72a <+1776>: movq   0x1419ff7(%rip), %rsi     ; "willSendViewDidLayoutSubviewsToViewControllerOfView:"
        0x1029ee731 <+1783>: movq   -0xb8(%rbp), %r14
        0x1029ee738 <+1790>: movq   %r14, %rdi
        0x1029ee73b <+1793>: movq   %r15, %rdx
        0x1029ee73e <+1796>: movq   0x109bec3(%rip), %r12     ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee745 <+1803>: callq  *%r12
        0x1029ee748 <+1806>: movq   0x1419fe1(%rip), %rsi     ; "viewDidLayoutSubviews"
        0x1029ee74f <+1813>: movq   %r13, %rdi
        0x1029ee752 <+1816>: callq  *%r12
        0x1029ee755 <+1819>: movq   0x1419fdc(%rip), %rsi     ; "didSendViewDidLayoutSubviewsToViewControllerOfView:"
        0x1029ee75c <+1826>: movq   %r14, %rdi
        0x1029ee75f <+1829>: movq   %r15, %rdx
        0x1029ee762 <+1832>: callq  *%r12
        0x1029ee765 <+1835>: movq   0x1419f84(%rip), %r14     ; "_embeddedDelegate"
        0x1029ee76c <+1842>: movq   %r13, %rdi
        0x1029ee76f <+1845>: movq   %r14, %rsi
        0x1029ee772 <+1848>: callq  *%r12
        0x1029ee775 <+1851>: movq   %rax, %rdi
        0x1029ee778 <+1854>: callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee77d <+1859>: movq   %rax, %rbx
        0x1029ee780 <+1862>: movq   %rbx, %rdi
        0x1029ee783 <+1865>: callq  *0x109be87(%rip)          ; (void *)0x00000001050f6cc0: objc_release
        0x1029ee789 <+1871>: testq  %rbx, %rbx
        0x1029ee78c <+1874>: je     0x1029ee7bb               ; <+1921>
        0x1029ee78e <+1876>: movq   %r13, %rdi
        0x1029ee791 <+1879>: movq   %r14, %rsi
        0x1029ee794 <+1882>: callq  *%r12
        0x1029ee797 <+1885>: movq   %rax, %rdi
        0x1029ee79a <+1888>: callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee79f <+1893>: movq   %rax, %rbx
        0x1029ee7a2 <+1896>: movq   0x1419f97(%rip), %rsi     ; "viewControllerViewDidLayoutSubviews:"
        0x1029ee7a9 <+1903>: movq   %rbx, %rdi
        0x1029ee7ac <+1906>: movq   %r13, %rdx
        0x1029ee7af <+1909>: callq  *%r12
        0x1029ee7b2 <+1912>: movq   %rbx, %rdi
        0x1029ee7b5 <+1915>: callq  *0x109be55(%rip)          ; (void *)0x00000001050f6cc0: objc_release
        0x1029ee7bb <+1921>: movq   %r15, %rdi
        0x1029ee7be <+1924>: movq   0x1419f5b(%rip), %rsi     ; "_wantsReapplicationOfAutoLayoutWithLayoutDirtyOnEntry:"
        0x1029ee7c5 <+1931>: movl   -0xdc(%rbp), %edx
        0x1029ee7cb <+1937>: callq  *0x109be37(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee7d1 <+1943>: testb  %al, %al
        0x1029ee7d3 <+1945>: je     0x1029ee7e5               ; <+1963>
        0x1029ee7d5 <+1947>: movq   0x141986c(%rip), %rsi     ; "_updateConstraintsAsNecessaryAndApplyLayoutFromEngine"
        0x1029ee7dc <+1954>: movq   %r15, %rdi
        0x1029ee7df <+1957>: callq  *0x109be23(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee7e5 <+1963>: movq   -0xf8(%rbp), %r13
        0x1029ee7ec <+1970>: testq  %r13, %r13
        0x1029ee7ef <+1973>: je     0x1029ee801               ; <+1991>
        0x1029ee7f1 <+1975>: movq   0x1419f50(%rip), %rsi     ; "containerViewDidLayoutSubviews"
        0x1029ee7f8 <+1982>: movq   %r13, %rdi
        0x1029ee7fb <+1985>: callq  *0x109be07(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee801 <+1991>: xorl   %eax, %eax
        0x1029ee803 <+1993>: callq  0x10348a567               ; _UIViewShowAlignmentRects
        0x1029ee808 <+1998>: testb  %al, %al
        0x1029ee80a <+2000>: movq   0x14633a7(%rip), %r12     ; UIView._viewFlags
        0x1029ee811 <+2007>: jne    0x1029ee82b               ; <+2033>
        0x1029ee813 <+2009>: movq   0x1459626(%rip), %rdi     ; (void *)0x0000000103e6e788: UIView
        0x1029ee81a <+2016>: movq   0x1419c97(%rip), %rsi     ; "_toolsDebugAlignmentRects"
        0x1029ee821 <+2023>: callq  *0x109bde1(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee827 <+2029>: testb  %al, %al
        0x1029ee829 <+2031>: je     0x1029ee848               ; <+2062>
        0x1029ee82b <+2033>: movq   0x1419c8e(%rip), %rsi     ; "_alignmentDebuggingOverlayCreateIfNecessary:"
        0x1029ee832 <+2040>: movl   $0x1, %edx
        0x1029ee837 <+2045>: movq   %r15, %rdi
        0x1029ee83a <+2048>: callq  *0x109bdc8(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee840 <+2054>: movq   %rax, %rdi
        0x1029ee843 <+2057>: callq  0x10379c832               ; symbol stub for: objc_unsafeClaimAutoreleasedReturnValue
        0x1029ee848 <+2062>: movq   0x14595f1(%rip), %rdi     ; (void *)0x0000000103e6e788: UIView
        0x1029ee84f <+2069>: movq   0x1419c7a(%rip), %rsi     ; "_toolsDebugColorViewBounds"
        0x1029ee856 <+2076>: callq  *0x109bdac(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee85c <+2082>: testb  %al, %al
        0x1029ee85e <+2084>: je     0x1029ee87d               ; <+2115>
        0x1029ee860 <+2086>: movq   0x1419c71(%rip), %rsi     ; "_colorViewBoundsOverlayCreateIfNecessary:"
        0x1029ee867 <+2093>: movl   $0x1, %edx
        0x1029ee86c <+2098>: movq   %r15, %rdi
        0x1029ee86f <+2101>: callq  *0x109bd93(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee875 <+2107>: movq   %rax, %rdi
        0x1029ee878 <+2110>: callq  0x10379c832               ; symbol stub for: objc_unsafeClaimAutoreleasedReturnValue
        0x1029ee87d <+2115>: movq   0x14595bc(%rip), %rdi     ; (void *)0x0000000103e6e788: UIView
        0x1029ee884 <+2122>: movq   0x1419ec5(%rip), %rsi     ; "_toolsDebugShouldDetectClippedViews"
        0x1029ee88b <+2129>: callq  *0x109bd77(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee891 <+2135>: testb  %al, %al
        0x1029ee893 <+2137>: je     0x1029ee8a5               ; <+2155>
        0x1029ee895 <+2139>: movq   0x1419ebc(%rip), %rsi     ; "_detectAndHandleClippedView"
        0x1029ee89c <+2146>: movq   %r15, %rdi
        0x1029ee89f <+2149>: callq  *0x109bd63(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee8a5 <+2155>: cmpb   $0x0, -0xc8(%rbp)
        0x1029ee8ac <+2162>: je     0x1029ee8f3               ; <+2233>
        0x1029ee8ae <+2164>: movq   0x1417e5b(%rip), %rsi     ; "_layoutEngine"
        0x1029ee8b5 <+2171>: movq   %r15, %rdi
        0x1029ee8b8 <+2174>: movq   0x109bd49(%rip), %rax     ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee8bf <+2181>: movq   %rax, %r14
        0x1029ee8c2 <+2184>: callq  *%r14
        0x1029ee8c5 <+2187>: movq   %rax, %rdi
        0x1029ee8c8 <+2190>: callq  0x10379c7ea               ; symbol stub for: objc_retainAutoreleasedReturnValue
        0x1029ee8cd <+2195>: movq   %rax, %rbx
        0x1029ee8d0 <+2198>: movq   0x14196a9(%rip), %rsi     ; "performPendingChangeNotifications"
        0x1029ee8d7 <+2205>: movq   %rbx, %rdi
        0x1029ee8da <+2208>: callq  *%r14
        0x1029ee8dd <+2211>: movq   %rbx, %rdi
        0x1029ee8e0 <+2214>: callq  *0x109bd2a(%rip)          ; (void *)0x00000001050f6cc0: objc_release
        0x1029ee8e6 <+2220>: movq   -0xc0(%rbp), %rax
        0x1029ee8ed <+2227>: andb   $-0x2, 0x8(%rax,%r12)
        0x1029ee8f3 <+2233>: andb   $-0x2, 0x7(%r15,%r12)
        0x1029ee8f9 <+2239>: leaq   0x14c8bec(%rip), %rax     ; __UILogIdenticalLayouts
        0x1029ee900 <+2246>: cmpb   $0x0, (%rax)
        0x1029ee903 <+2249>: je     0x1029ee922               ; <+2280>
        0x1029ee905 <+2251>: callq  0x10379b1ac               ; symbol stub for: CFAbsoluteTimeGetCurrent
        0x1029ee90a <+2256>: subsd  -0xe8(%rbp), %xmm0
        0x1029ee912 <+2264>: movq   0x1419e47(%rip), %rsi     ; "_validateLayoutHashHasChangedWithLayoutTime:"
        0x1029ee919 <+2271>: movq   %r15, %rdi
        0x1029ee91c <+2274>: callq  *0x109bce6(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee922 <+2280>: movq   0x1419e3f(%rip), %rsi     ; "willExitLayoutSublayersOfLayerForView:"
        0x1029ee929 <+2287>: movq   -0xb8(%rbp), %r14
        0x1029ee930 <+2294>: movq   %r14, %rdi
        0x1029ee933 <+2297>: movq   %r15, %rdx
        0x1029ee936 <+2300>: callq  *0x109bccc(%rip)          ; (void *)0x00000001050f9940: objc_msgSend
        0x1029ee93c <+2306>: movq   0x109bccd(%rip), %rbx     ; (void *)0x00000001050f6cc0: objc_release
        0x1029ee943 <+2313>: movq   %r13, %rdi
        0x1029ee946 <+2316>: callq  *%rbx
        0x1029ee948 <+2318>: movq   -0xf0(%rbp), %rdi
        0x1029ee94f <+2325>: callq  *%rbx
        0x1029ee951 <+2327>: movq   -0xd8(%rbp), %rdi
        0x1029ee958 <+2334>: callq  *%rbx
        0x1029ee95a <+2336>: movq   -0xc0(%rbp), %rdi
        0x1029ee961 <+2343>: callq  *%rbx
        0x1029ee963 <+2345>: movq   %r14, %rdi
        0x1029ee966 <+2348>: callq  *%rbx
        0x1029ee968 <+2350>: movq   0x109b359(%rip), %rax     ; (void *)0x000000010a39e070: __stack_chk_guard
        0x1029ee96f <+2357>: movq   (%rax), %rax
        0x1029ee972 <+2360>: cmpq   -0x30(%rbp), %rax
        0x1029ee976 <+2364>: jne    0x1029ee98a               ; <+2384>
        0x1029ee978 <+2366>: addq   $0x118, %rsp              ; imm = 0x118 
        0x1029ee97f <+2373>: popq   %rbx
        0x1029ee980 <+2374>: popq   %r12
        0x1029ee982 <+2376>: popq   %r13
        0x1029ee984 <+2378>: popq   %r14
        0x1029ee986 <+2380>: popq   %r15
        0x1029ee988 <+2382>: popq   %rbp
        0x1029ee989 <+2383>: retq   
        0x1029ee98a <+2384>: callq  0x10379c32e               ; symbol stub for: __stack_chk_fail

