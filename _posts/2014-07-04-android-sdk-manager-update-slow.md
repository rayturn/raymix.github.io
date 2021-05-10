---
layout: post
title: Android SDK Manager 更新慢解决
date: 2014-07-04 22:51
author: ray
comments: true
categories: [Blog]
---
1、如果是windows7，那么"开始 - 所有程序 - Android SDK Tools, 右键SDK Manager - 以管理员身份运行(A)"
2、在SDK Manager窗口中，Tools - Options, 打开Settings
1) 在Misc下选中Force https://...sources to be fetched using http://...（原来默认使用https，现在强制使用http）
3、打开hosts文件：Windows在C:\WINDOWS\system32\drivers\etc目录下，Linux/MAC用户打开/etc/hosts文件

sudo vi /etc/hosts

在文件末尾默认添加星号行内代码：
****************************************
#Google主页
203.208.46.146 www.google.com
74.125.113.121 developer.android.com
#更新的内容从以下地址下载
203.208.46.146 dl.google.com
203.208.46.146 dl-ssl.google.com

&nbsp;
