---
layout: post
title: Unity 在IOS9 下字体残缺
date: 2015-11-18 13:34
author: ray
comments: true
categories: [Blog]
tags: [IOS9, Unity]
---
升级到最新的Unity4.6.8之后，虽然Unity声称解决了字体问题，但是我们在项目中还是遇到了奇怪的问题，IOS9设备每次第一次运行时部分字体会残缺，但是在切换场景后字体恢复正常显示，IOS8的设备一切正常。

经过大量的测试后发现问题原来出在 Player Settings 中的 Graphics API选项上，之前选择了Open GL ES 2.0,修改为Automatic后问题解决。
<!--more-->