---
title: CocoaPods 学习
date: 2017-03-04 11:06:48
tags: 
categories: 学习
---

[TOC]


## 项目组成

CocoaPods is composed of the following projects:

| Project | Description | Info |
| :------ | :--- | :--- |
| [CocoaPods](https://github.com/CocoaPods/CocoaPods) | The CocoaPods command line tool. | [guides](https://guides.cocoapods.org)
| [CocoaPods Core](https://github.com/CocoaPods/Core) | Support for working with specifications and podfiles. | [docs](http://docs.cocoapods.org/cocoapods_core)
| [CocoaPods Downloader](https://github.com/CocoaPods/cocoapods-downloader) |  Downloaders for various source types. |  [docs](http://docs.cocoapods.org/cocoapods_downloader/index.html)
| [Xcodeproj](https://github.com/CocoaPods/Xcodeproj) | Create and modify Xcode projects from Ruby. |  [docs](http://docs.cocoapods.org/xcodeproj/index.html)
| [CLAide](https://github.com/CocoaPods/CLAide) | A small command-line interface framework.  | [docs](http://docs.cocoapods.org/claide/index.html)
| [Molinillo](https://github.com/CocoaPods/Molinillo) | A powerful generic dependency resolver.  | [docs](http://www.rubydoc.info/gems/molinillo)
| [CocoaPods.app](https://github.com/CocoaPods/CocoaPods-app) | A full-featured and standalone installation of CocoaPods.  | [info](https://cocoapods.org/app)
| [Master Repo ](https://github.com/CocoaPods/Specs) | Master repository of specifications. | [guide](http://docs.cocoapods.org/guides/contributing_to_the_master_repo.html)

<!-- more --> 


## 学习资料
[objc系列译文（6.4）：深入理解 CocoaPods](http://ios.jobbole.com/53365/)

## 核心组件

CocoaPods是用ruby写的，并划分成了若干个Gem包。CocoaPods在解析执行过程中最重要的几个包的路径分别是：CocoaPods/CocoaPods、 CocoaPods/Core 和 CocoaPods/Xcodeproj。

### CocoaPods / CocoaPod

这是面向用户的组件，每当你执行一个pod命令时，这个组件将被激活。它包括了所有实用CocoaPods的功能，并且还能调用其他gem包来执行任务。

### CocoaPods / Core

Core gem提供了与CocoaPods相关的文件（主要是Podfile和podspecs）的处理。

### Podfile

Podfile用于配置项目所需要的第三方库。它能被高度定制，所以你可以尽可能地给它添加你想要的特性。如果您还想对Podfile了解更多的话，请查看Podfile指南(地址 http://guides.cocoapods.org/syntax/podfile.html)。

### Podspec

.podspec文件描述了一个库将怎样被添加进工程中。.podspec文件可以标识该第三方库所需要的源码文件、依赖库、编译选项，以及其他第三方库需要的配置。

### CocoaPods / Xcodeproj

这个包负责工程文件直接关系的处理。它能创建以及修改.xcodeproj文件和.xcworkspace文件。它也可以作为一个独立的包使用，当你要编写修改项目文件的脚本时，可以考虑使用CocoaPods/Xcodeproj。

---

### 流程
运行 pod install 命令

pod install的执行引发了很多操作。了解底层运行过程最简单的方式就是给pod install语句添加 –verbose 参数。现在，运行结果见附录

#### 更新 specs 仓库（Updating local specs repositories）
路径：/Users/username/.cocoapods/repos
这个目录包含了所有公有库的所有版本的 `.podspec.json` 文件。 

*创建仓库相关链接*

- [Specs and the Specs Repo](http://guides.cocoapods.org/making/specs-and-specs-repo.html): Learn about creating Podspec's and the Spec repo.
- [Getting setup with Trunk](http://guides.cocoapods.org/making/getting-setup-with-trunk.html): Instructions for creating a CocoaPods user account


**.podspec 文件**
主要包含了：

- 版本信息
- 源地址
- 依赖

#### 分析依赖（Analyzing dependencies）
根据 .podspec 去分析项目需要的所有依赖

#### 版本控制和冲突

CocoaPods 使用语义版本命名约定来解决对版本的依赖。由于冲突解决系统建立在非重大更改的补丁版本之间，这使得解决依赖关系要容易得多。举个例子，两个完全不同的第三方库同时依赖 CocoaLumberjack 。它们其中一个依赖的版本是2.3.1，而另一个则为2.3.3，解析器可以自动使用较新的版本，在这里则是2.3.3，因为这可以与2.3.1向后兼容。

但这并不总是有效。有许多第三方库还并不支持这个约定，这让解决方案变得非常复杂。

当然，总是会有一些冲突需要手工解决。如果一个第三方库依赖CocoaLumberjack 1.2.5，而另一个依赖CocoaLumberjack 2.3.1，最后只能靠调用这两个第三方库的用户来手动地决定CocoaLumberjack的版本了。

#### 加载源码

CocoaPods 执行的下一个步骤是加载源代码。每个.podspec文件都包含了源代码的索引，这些索引一般指向了一个git地址或者git tag。它们以commit SHA码的方式存储在 ~/Library/Caches/CocoaPods中。而在这些路径中创建文件则由 Core 包负责。

源代码将依照Podfile、.podspec和缓存文件的信息下载到相应的第三方库路径。

#### 生成 Pods.xcodeproj

每次 pod install 执行后并且检测到改动时，Pods.xcodeproj文件将被 Xcodeproj gem更新。如果 Pods.xcodeproj 文件不存在，则会以默认配置生成，若已存在，则 Pods.xcodeproj 会使用现有的配置。

#### 安装第三方库

当 Cocoapods 向项目中增加了一个第三方库的时候，不仅仅是将添加代码这么简单。由于每个第三方库有不同的 target，所以每次添加第三方库时，都只有几个文件被添加。每个源代码都需要：

一个包含编译选项的 .xcconfig 文件
一个同时拥有编译设置和 CocoaPods 默认配置的私有 .xcconfig 文件
编译所必须的 prefix.pch 文件
另一个编译必须的文件 dummy.m
一旦每个 pod 的 target 都完成了以上步骤，整个 Pods 的 Target 就会被创建。这增加了相同的文件，与另外几个。如果有源代码中包含了资源 bundle，向 app 的 target 中添加 bundle 的方式将写入 Pods-Resources.sh。还有一个叫Pods-environment.h 的文件，文件中含有许多检查组件是否来自 pod 的宏定义。最后，将生成两个确认文件，一个 .plist 文件，一个用于给用户查阅许可信息的 markdown 文件。

#### 写入到磁盘

直到现在，许多已完成的过程都使用的是内存中的对象。为了让这些过程的结果可重复被使用，我们需要将所有结果都记录在一个文件中。所以 Pods.xcodeproj 和另外两个非常重要的文件：Podfile.lock 和 Manifest.lock 都将被写入磁盘。

#### Podfile.lock

这是 CocoaPods 创建的最重要的文件之一。它记录了需要被安装的 pod 的每个已安装的版本。如果你想知道已安装的 pod 是哪个版本，可以查看这个文件。推荐将 Podfile.lock 文件加入到版本控制中，这有助于整个团队的一致性。

#### Manifest.lock

这是每次运行 pod install 时创建的 Podfile.lock 文件的副本。如果你见过“沙盒文件和 Podfile.lock 文件不同步”的错误，这个错误就是因 Manifest.lock 文件和 Podfile.lock 文件不一样引起。由于 Pods 所在的目录并不总在版本控制之下，这样可以保证开发者运行 app 之前都能更新他们的 pods，否则 app 可能会 crash，或者在一些不太明显的地方编译失败

#### xcproj

如果您已经依照我们的建议在系统上安装了 xcproj，它会将您的 Pods.xcodeproj 文件转换成就旧有 ASCII 格式的 plist 文件。为什么要这么做呢？因为 Xcode 所依赖和使用的 plist 在很久以前就已经不被其他软件支持了。如果没有  xcproj，你的 Pods.xcodeproj 文件将会以 XML 格式的 plist 文件存储，当你用 Xcode 打开它时，它会被改写，造成大量的文件冲突。

 

#### 运行结果

运行 pod install 的最终结果是许多文件被添加到你的工程和系统中。这个过程通常只需要几秒钟。当然没有 Cocoapods 这些事也都可以完成。只不过所花的时间就不仅仅是几秒而已了。
       

---
附录
---

#### pod install 运行结果
	pod install --verbose

将会出现以下执行结果：


	 Preparing
	[!] Unable to load a specification for the plugin `/Users/xxxx/.rvm/gems/ruby-2.2.4@global/gems/cocoapods-deintegrate-1.0.0`

	Updating local specs repositories

	Updating spec repo `master`
	$ /usr/bin/git pull --ff-only 
	From https://github.com/CocoaPods/Specs
		46384af..01239bf  master     -> origin/master
	Fast-forward
	.../!ProtoCompiler-gRPCPlugin.podspec.json         |   31 +
	
	……省略……
	
	Analyzing dependencies

	Inspecting targets to integrate
		Using `ARCHS` setting to build architectures of target `Pods`: (``)
	Finding Podfile changes
	 - AFNetworking
	 - APParallaxHeader
	 ……省略……
	 
	Fetching external sources
	-> Fetching podspec for `APParallaxHeader` from `./xxx/Vender/APParallaxHeader/APParallaxHeader.podspec`
	
	Resolving dependencies of `Podfile`
	
	Comparing resolved specification to the sandbox manifest
	  - AFNetworking
	  - APParallaxHeader
	  ……省略……
	
	Downloading dependencies
	-> Using AFNetworking (2.6.3)
	……省略……
	
	Generating Pods project
		- Creating Pods project
		- Adding source files to Pods project
		- Adding frameworks to Pods project
		- Adding libraries to Pods project
		- Adding resources to Pods project
		- Linking headers
		- Installing targetsRunning post install hooks
		 - Installing target `AFNetworking` iOS 8.0
		  - Generating Info.plist file at `Pods/Target Support Files/AFNetworking/Info.plist`
		  - Generating module map file at `Pods/Target Support Files/AFNetworking/AFNetworking.modulemap`
		  - Generating umbrella header at `Pods/Target Support Files/AFNetworking/AFNetworking-umbrella.h`
		 
		……省略……	
	- Running post install hooks
	- Writing Xcode project file to `Pods/Pods.xcodeproj`
    - Generating deterministic UUIDs
    - Writing Lockfile in `Podfile.lock`
    - Writing Manifest in `Pods/Manifest.lock`

	Integrating client project

	Integrating target `Pods` (`xxx.xcodeproj` project)
	 - Running post install hooks
	 - cocoapods-stats from `/Users/xxxx/.rvm/gems/ruby-2.2.4@global/gems/cocoapods-stats-0.6.2/lib/cocoapods_plugin.rb`

	Sending stats
      - AFNetworking, 2.6.3
      - APParallaxHeader, 0.1.6
      ……省略……	
    - cocoapods-stats from
    `/Users/xxxx/.rvm/gems/ruby-2.2.4@global/gems/cocoapods-stats-1.0.0/lib/cocoapods_plugin.rb`
    
    ……省略……	
    
    Pod installation complete! There are 20 dependencies from the Podfile and 21 total pods installed.
