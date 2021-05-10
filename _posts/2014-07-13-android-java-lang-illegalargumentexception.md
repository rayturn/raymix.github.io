---
layout: post
title: "Android: java.lang.IllegalArgumentException: No config chosen"
date: 2014-07-13 09:34
author: ray
comments: true
categories: [Blog]
tags: [Android, cocos2d, cocos2d-x]
---

Simulator:

gLSurfaceView.setEGLConfigChooser(8 , 8, 8, 8, 16, 0);

Device:

gLSurfaceView.setEGLConfigChooser(5, 6, 5, 0, 16, 8);
