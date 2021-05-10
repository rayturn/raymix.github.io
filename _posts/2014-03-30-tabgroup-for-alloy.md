---
layout: post
title: TabGroup for Alloy
date: 2014-03-30 13:48
author: ray
comments: true
categories: [Blog]
---
<h2>Overview</h2>
This is a widget for the <a href="http://www.appcelerator.com">Appcelerator</a> <a href="http://projects.appcelerator.com/alloy/docs/Alloy-bootstrap/index.html">Alloy</a> MVC framework which provides an iOS + Android TabGroup with enhanced customisation options.

The idea was to come up with something that could "just work" on both iOS and Android, and sit in the same position at the bottom of the screen for both, and wouldn't require too much integration changes.
<!--more-->
<h3><a href="https://github.com/jasonkneen/com.jasonkneen.tabgroup#note-this-is-very-much-work-in-progress-and-im-tweaking-as-i-use-this-in-current-projects" name="note-this-is-very-much-work-in-progress-and-im-tweaking-as-i-use-this-in-current-projects"></a>NOTE: This is very much "work in progress" and I'm tweaking as I use this in current projects</h3>
<a href="https://raw.github.com/jasonkneen/images/master/tabGroup/ios.png" target="_blank"><img src="https://raw.github.com/jasonkneen/images/master/tabGroup/ios.png" alt="TabGroup" /></a><a href="https://raw.github.com/jasonkneen/images/master/tabGroup/android.png" target="_blank"><img src="https://raw.github.com/jasonkneen/images/master/tabGroup/android.png" alt="TabGroup" /></a><a href="https://raw.github.com/jasonkneen/images/master/tabGroup/android2.png" target="_blank"><img src="https://raw.github.com/jasonkneen/images/master/tabGroup/android2.png" alt="TabGroup" /></a>
<h2><a href="https://github.com/jasonkneen/com.jasonkneen.tabgroup#latest" name="latest"></a>Latest</h2>
<ul>
	<li>Merged some pull requests to squash some bugs and add focus events</li>
	<li>Avoid using navBarHidden in Android Windows or ANY settings that may cause your tab windows to go heavyweight as you'll get unexpected behavoir. If you need to hide the title bar in Android, use the platform theme file to do that.</li>
</ul>
In 3.2 of Titanium, all Windows are Heavyweight on Android - a workaround is to put
<div>
<pre>&lt;property name="ti.android.useLegacyWindow" type="bool"&gt;true&lt;/property&gt;</pre>
</div>
in the TiApp.xml
<h2><a href="https://github.com/jasonkneen/com.jasonkneen.tabgroup#features" name="features"></a>Features</h2>
<ul>
	<li>Easy to add to existing XML</li>
	<li>Works with existing Window definitions</li>
	<li>Easy to customise</li>
	<li>Works on Android, iOS</li>
</ul>
<h3><a href="https://github.com/jasonkneen/com.jasonkneen.tabgroup#wishlist" name="wishlist"></a>Wishlist</h3>
<ul>
	<li>replace iOS titlebar with custom version with support for left, right buttons</li>
	<li>hide / show tabs dynamically, refresh</li>
	<li>custom, oversized tab buttons e.g. middle button</li>
	<li>click and hold - vertical (tweetbot style) sub tabs</li>
</ul>
<h2></h2>
<!--more-->
<h2></h2>
<h2><a href="https://github.com/jasonkneen/com.jasonkneen.tabgroup#quick-start" name="quick-start"></a>Quick Start</h2>
<ul>
	<li><a href="https://github.com/jasonkneen/com.jasonkneen.tabgroup">Download the latest version</a> of the widget as a ZIP file.</li>
	<li>Unzip the folder to your project under <code>app/widgets/com.jasonkneen.tabgroup</code>.</li>
	<li>Add the widget as a dependency to your <code>app/config.json</code> file:</li>
</ul>
<div>
<pre>"dependencies": {
    "com.jasonkneen.tabgroup":"1.0"
}</pre>
</div>
<ul>
	<li>Add the widget as the last item in the view container you want to apply dynamic styles too, so your main window or view, just above the closing Window or View tags.</li>
</ul>
<div>
<pre>&lt;Alloy&gt;
    &lt;Window&gt;
        &lt;Widget id="tabGroup" src="com.jasonkneen.tabgroup"/&gt;
    &lt;/Window&gt;
    &lt;Window title="Tab 1" id="win1"&gt;
        &lt;Label top="0"&gt;I am Window 1&lt;/Label&gt;
    &lt;/Window&gt;
    &lt;Window title="Tab 2" id="win2" &gt;
        &lt;Label top="0"&gt;I am Window 2&lt;/Label&gt;
    &lt;/Window&gt;
    &lt;Window title="Tab 3" id="win3" &gt;
        &lt;Label top="0"&gt;I am Window 3&lt;/Label&gt;
    &lt;/Window&gt;
    &lt;Window title="Tab 4" id="win4" &gt;
        &lt;Label top="0"&gt;I am Window 4&lt;/Label&gt;
    &lt;/Window&gt;
&lt;/Alloy&gt;</pre>
</div>
<ul>
	<li>Configure the widget from the controller .js file (current properties supported shown here)</li>
</ul>
<div>
<pre>$.tabGroup.configure({
    backgroundColor : "#000",
    //backgroundImage : "PATH",
    tabsAtBottom:true,    //这个参数决定tabgroup是否在bottom

    tabs : {
        backgroundSelectedColor : "#666",
        color : "#DDD",
        selectedColor : "#fff",
        font : {
            fontWeight : "normal"
        },
        selectedFont : {
            fontWeight : "bold"
        }       
    }
});

$.tabGroup.addTab({
    caption : "Downloads",
    icon : "/images/icons/grey/516-archive-box.png",
    selectedIcon : "/images/icons/white/516-archive-box.png",
    win : $.win1,
    // view : custom Ti.UI.View
    // selectedView : custom Ti.UI.View for selected state
});

$.tabGroup.addTab({
    caption : "Settings",
    icon : "/images/icons/grey/519-tools-1.png",
    selectedIcon : "/images/icons/white/519-tools-1.png",
    win : $.win2
});

$.tabGroup.addTab({
    caption : "Save",
    icon : "/images/icons/grey/522-floppy-disk.png",
    selectedIcon : "/images/icons/white/522-floppy-disk.png",
    win : $.win3
});

$.tabGroup.addTab({
    caption : "Scoot",
    icon : "/images/icons/grey/530-scooter.png",
    selectedIcon : "/images/icons/white/530-scooter.png"
});

$.tabGroup.addTab({
    caption : "Fly",
    icon : "/images/icons/grey/533-helicopter.png",
    selectedIcon : "/images/icons/white/533-helicopter.png"
});

$.tabGroup.open();</pre>
</div>
<h2><a href="https://github.com/jasonkneen/com.jasonkneen.tabgroup#quick-start-1" name="quick-start-1"></a>Quick Start</h2>
Methods &amp; Properties:-
<ul>
	<li>.configure(dictionary) - set's up properties for tabGroup, default tabs</li>
	<li>.addTab(dictionary) - adds a new tab</li>
	<li>.open() - opens the Tabgroup</li>
	<li>.getActiveTab() - returns the current tab controller</li>
	<li>.setActiveTab(n) - sets the tab to n (0 - tabcount)</li>
	<li>.activeTab - current active tab</li>
	<li>.activeTab.open(win) - open a subwindow</li>
	<li>.activeTab.close(win) - close a subwindow</li>
</ul>
Events :-

.on("focus") - will pass you the tab that has focus

(also each time you switch tabs, the window in the tab will get focus / blur events)
<h2>Download &amp; Source</h2>
<a title="https://github.com/jasonkneen/com.jasonkneen.tabgroup" href="https://github.com/jasonkneen/com.jasonkneen.tabgroup" target="_blank">https://github.com/jasonkneen/com.jasonkneen.tabgroup</a>
<h2><a href="https://github.com/jasonkneen/com.jasonkneen.tabgroup#license" name="license"></a>License</h2>
<pre>Copyright 2013 Jason Kneen

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.</pre>
