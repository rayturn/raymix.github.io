---
layout: post
title: apk 资源提取及 dex 文件反编
date: 2014-02-28 11:39
author: ray
comments: true
categories: [Blog]
tags: [Android, apk, apktool, dex, dex2jar]
---
1. 提取apk中的图片、XML配置、语言资源等文件

首先下载最新的 <a href="https://code.google.com/p/android-apktool/downloads/list" target="_blank">apktool</a>

按照官方说明进行安装。

执行：
<blockquote>apktool d -f &lt;name&gt;.apk &lt;target&gt;</blockquote>
另外有一个图形化工具叫：Androidfby

2.dex文件反编

相关工具： <a href="http://code.google.com/p/dex2jar/downloads/list" target="_blank">dex2jar</a>，<a href="http://code.google.com/p/innlab/downloads/list" target="_blank">jdgui</a> 最新版本请见 <a href="http://java.decompiler.free.fr/?q=jdgui" target="_blank">官方</a>
