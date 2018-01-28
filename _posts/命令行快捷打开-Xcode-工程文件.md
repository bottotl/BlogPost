---
title: 命令行快捷打开 Xcode 工程文件
date: 2017-12-08 13:31:34
tags:
---

环境依赖：zsh 

## 编辑 .zshrc

加入如下行

	function xc {
	  proj=$(ls -d *.xcworkspace/ 2>/dev/null)

	  if [ -n "$proj" ]; then
	    # Omit -beta if you're not using beta version
	    open -a Xcode "$proj"
	  else
	    echo "No Xcode project detected."
	  fi

	}

## 重新激活 zsh
	source .zshrc

## 使用

pod install 结束以后命令后执行 xc 自动打开 xcode 文件