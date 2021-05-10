---
layout: post
title: QBASIC-独角戏
date: 1995-11-01 23:06
author: ray
comments: true
categories: [Works]
---
初中一年级作品 - 独角戏

95年QBASIC练习程序，播放音乐的同时显示字幕，需中文DOS环境支持。

字幕的播放速度很依赖计算机的时钟频率，当年没有网络，书籍有限，没找到如何可以精确的延时，利用死循环进行延时，speed变量控制时长。

字幕的播放速度根据曲谱中定义的时长计算。字幕以逐像素变色的方式显示。

<!--more-->

运行截图：
<code><a href="/assets/1995/11/Snip20160202_5.png" rel="attachment wp-att-126"><img class="alignnone size-medium wp-image-126" src="/assets/1995/11/Snip20160202_5-300x222.png" alt="Snip20160202_5" width="300" height="222" /></a></code>

<code>
SCREEN 12: COLOR 13: DIM a$(20), b$(20), c(20), t(20, 20): speed = 2900
a$(1) = "t200o3f4f4f4f4mnf4d4a1"
c(1) = 300
a$(2) = "o4f4e4d4c4o3a4o4c4o3a1"
c(2) = 200
a$(3) = "o4c4o3a4g4g4mng4f4g4d4"
c(3) = 200
a$(4) = "o4c4c4mnc4o3a4g4a4"
c(4) = 0
a$(5) = "g4g4mng4a4g4d4e1"
c(5) = 200
a$(6) = "t200o3f4f4f4f4mnf4d4a1"
c(6) = 200
a$(7) = "o4f4e4d4c4o3a4o4c4o3a1"
c(7) = 200
a$(8) = "o4c4o3a4g4g4mng4f4g4d4"
c(8) = 200
a$(9) = "o4c4c4mnc4o3a4g4a4"
c(9) = 200
a$(10) = "o4e4e4mne4f4e4d4d32e1p2"
c(10) = 500
a$(11) = "c4o3b4a4g4c4d4e2"
c(11) = 120
a$(12) = "a4g4a4o4c4o3c4g4e2"
c(12) = 100
a$(13) = "g4o3f4e4e4mne4d4e4b4"
c(13) = 200
b$(1) = "是谁导演这场戏"
b$(2) = "在这孤单角色里"
b$(3) = "对白总是自言自语"
b$(4) = "对手总是回忆"
b$(5) = "看不出什么结局"
b$(6) = "自始自终全是你"
b$(7) = "让我投入太彻底"
b$(8) = "故事如果注定悲剧"
b$(9) = "何苦给我美丽"
b$(10) = "演出相聚和别离"
b$(11) = "没有星星的夜里"
b$(12) = "我用泪光吸引你"
b$(13) = "既然爱你不能言语"
b$(14) = "只能微笑哭泣"
b$(15) = "让我从此忘了你"
b$(16) = "没有星星的夜里"
b$(17) = "我把往事留给你"
b$(18) = "如果一切只是演戏"
b$(19) = "要你好好看戏"
b$(20) = "心碎只是我自己"
FOR j = 1 TO 10
la = LEN(a$(j)): lb = LEN(b$(j))
FOR i = 1 TO la
i$ = LCASE$(MID$(a$(j), i, 1))
IF i$ = "c" OR i$ = "d" OR i$ = "e" OR i$ = "f" OR i$ = "g" OR i$ = "a" OR i$ = "b" THEN
i1$ = MID$(a$(j), i + 2, 1)
IF i1$ &lt;&gt; "2" AND i1$ &lt;&gt; "6" THEN
i2$ = MID$(a$(j), i + 1, 1)
ELSE
i2$ = MID$(a$(j), i + 1, 2)
END IF
a = a + 1: t(j, a) = VAL(i2$)
END IF
IF i = la THEN a = 0
NEXT i
CLS
LOCATE 20, (80 - lb) / 2: PRINT b$(j): PLAY "mb" + a$(j)
FOR s = 1 TO lb / 2
xstart = ((80 - lb) / 2 - 1) * 8
FOR x = 1 TO 16
FOR y = 1 TO 21
c = POINT((s - 1) * 16 + x + xstart - 1, 300 + y)
IF c = 13 THEN PSET ((s - 1) * 16 + x + xstart - 1, 300 + y), 15
FOR c = 1 TO speed / (t(j, s) + 1): NEXT c
NEXT y
NEXT x
NEXT s
NEXT j
END
</code>
