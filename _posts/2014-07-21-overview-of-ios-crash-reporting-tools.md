---
layout: post
title: Overview of iOS Crash Reporting Tools
date: 2014-07-21 11:34
author: ray
comments: true
categories: [Blog]
tags: [Android, crash, IOS]
---
Believe it or not, developers are not perfect, and every once in a while you might have a (gasp!) bug in your app.

You will try your best to ship your apps with no bugs in them, but more often than not you realise afterwards that a bug has slipped through the net. Sometimes such bugs result in crashes, which no user likes to encounter.
<!--more-->
Never fear though, for there are many excellent iOS crash reporting tools to help you out! They can detect crashes and log details so you can investigate later, and even send you an email summary of when crashes occur and how often.

However, you may wonder which of these tools is the best? Read on to find out!

&nbsp;
<h2>Introduction</h2>
You’ve spent months working on your newest application, you’ve spent weeks beta testing it, you’ve published it on the App Store, and the positive reviews are starting to roll in. Everything looks great!

But then one user posts a one-star review noting the worst thing possible — the app crashed. Yikes! Nothing is worse than hearing about crashes from the user community.

Although it’s nearly impossible to test for every possible scenario, you can make the triage and troubleshooting portion of your job a little easier by using crash reporting tools in your apps. In fact, they are possibly the most important tool you can make use of to improve your application. Even if you test it thoroughly before the submission to the store, chances are you may have forgotten to check a particular sequence or scenario which leads to a crash.

From a user perspective, a crash is more than just an annoyance. When the application just stops working without any feedback, this often results in lost data and unhappy users. Installing a crash reporting system in your app allows you to obtain all the information you need to be able to fix these annoyances.

<a href="http://cdn5.raywenderlich.com/wp-content/uploads/2013/05/Fotolia_35452506_XS.jpg"><img src="http://cdn5.raywenderlich.com/wp-content/uploads/2013/05/Fotolia_35452506_XS.jpg" alt="App crashes are like death and taxes - they happen. But it's up to you how to handle them!" width="322" height="373" /></a>

App crashes are like death and taxes – they happen. But it’s up to you how to handle them!

When it comes to troubleshooting, the more information that is collected — such as the OS version, the device type, and the application version — the more details you have to track down the source of the bug and fix it. That way, you can turn each crash incident into an opportunity to improve your app!

In this two-part series, you’ll get an overview of the different options of crash reporting tools available to developers. You’ll learn about various free and commercial tools, as well as the various pros and cons of each tool. By the end of the first part, you’ll be able to make an informed choice as to the best tool for your needs. In the second part, you will learn how to get started with each crash reporting service and tie it into your app.

Sound good? Then let’s take a behind-the-scenes look at what goes into a crash reporting tool.
<h2>Crash Reporting Basics</h2>
A crash reporting tool is actually a combination of two components: a reporting library and a server-side collector. You can think of these items in terms of a restaurant: the reporting library is the kitchen and the collector is the waiter. The developer — i.e. you — plays the part the customer.

You can have the best organized kitchen in the world, but you’ll never eat anything if there isn’t a waiter to bring food to the table. Likewise, your waiter can provide excellent service, but if the food is not well prepared, your experience will not be a good one!

As a great restaurant should include a great kitchen and great waiters, a crash reporting tool should include a great reporting library and a great server-side collector. The role of the reporting library is to prepare the details about a crash. The role of the server-side component is to collect the crash data and present it in a meaningful way.

<!--more-->

&nbsp;
<h2>Symbolication</h2>
In iOS crash reporting, there’s one more thing that’s crucial to understanding your crash reports – symbolication. But what is symbolication? You can think of symbolication as the chef in the kitchen, who transforms raw ingredients like bananas, ice cream and rum into something meaningful and tasty such as <a href="http://en.wikipedia.org/wiki/Bananas_Foster">Bananas Foster</a>.

A crash report contains a stack trace of every thread that was running when the application terminated. This trace is similar to the traces you see in the debugger when paused, although a debugger’s trace will include both instance names and methods (often referred as symbols). In comparison, when an app is built for release, usually these symbol names are stripped from the binary. This means that crash reports coming into your crash reporter service contain strange looking hexadecimal addresses instead of symbol names.

You will no doubt have at least one crash log on your device, so to see what these hexadecimal addresses look like, find such a log like this:
<ol>
	<li>Connect your device to your Mac.</li>
	<li>Start Xcode and open the Organizer (CMD+Shift+2).</li>
	<li>Find your device on the left and select the “Device Logs” item as shown in the screenshot below:</li>
</ol>
<img title="Device in organizer" src="http://cdn1.raywenderlich.com/wp-content/uploads/2013/02/phone.png" alt="A device as listed in the organizer of Xcode" width="194" height="118" />

A list of crash reports will show up, unless your device is brand new and you haven’t used it much. Select one application, and have a look to the right. You should see text displayed similar to the following:

<img title="An example of Crash" src="http://cdn2.raywenderlich.com/wp-content/uploads/2013/02/hexaCrash.png" alt="An example of Crash" width="699" height="256" />

Notice how there are some symbols, for example <code>-[__NSArrayI objectAtIndex:]</code>, and some hexadecimal addresses, for example <code>0x000db142 0xb1000 + 172354</code>. This is what is known as a partially symbolicated crash log. The top of the stack trace is where the crash occurred, and going down the list indicates which other methods were called to get to where the crash occurred.

The reason that it’s a “partly” symbolicated log is because Xcode is able to symbolicate just the system components like UIKit and CoreFoundation (lines 6 to 18 in the screenshot above). But crashes aren’t usually generated by system libraries, instead they have their roots in the code you write. For example, line 2 indicates that the code path that lead to the crash went through the <code>objectAtIndex:</code> method on <code>NSArray</code>.

However, take a look at line 3 in the screenshot above. What’s the meaning of this?
<div>
<table>
<tbody>
<tr id="p336691">
<td id="p33669code1">
<pre>0x000db142 0xb1000 + 172354</pre>
</td>
</tr>
</tbody>
</table>
</div>
This is an example of an un-symbolicated portion of the log. All this tells us is that the crash occurred at the instruction at memory address <code>0x000db142</code> which is equal to <code>0xb1000 + 172354</code>. The reason for it telling us that seemingly obscure bit of math is that <code>0xb1000</code> is the start address of the main portion of the app. The 172354 is the important bit as it is the offset into the app.
<div>

<em>Note:</em> All these numbers relate to byte offsets within the memory addressable to the CPU. When an application is run, the binary is loaded into memory and read instruction by instruction, by the CPU. The offset where the program is loaded, in this case 0xb1000, is randomised as a minimal form of security against exploits.

</div>
But, that’s not terribly useful for debugging purposes. A raw crash report contains what you need to understand the <i>cause</i> of the crash, but you can’t fully comprehend it unless you dive a little deeper and match it up with the source code through a process known as “symbolication”. This is the process of turning these raw numbers into symbols and even pinpointing the exact line of code.
<h2>The Symbolication Process</h2>
In the case of iOS, this requires two things. The first is the exact application binary that caused the crash. The second is the dSYM file created during the compilation of the binary.

How do you get a hold of these two items? Well, if you are using the “Build and Archive” command in Xcode then you are already storing them! In Xcode, you can see these in the “Organizer” window. To see for yourself, open Xcode then go to <em>Window\Organizer</em>. Then select the “Archives” tab where you will see a list of your apps on the left and on the right you’ll see a list of the archives for the selected app.

However, if you haven’t been using “Build and Archive”, or don’t have any applications to choose from, simply create a dummy app to see where Xcode stores these files.
<div>

<em>Note:</em> If you’re familiar with the Organizer, feel free to skip this section and move on to the section titled “Find that dSYM!”

</div>
To create a dummy app. Start up Xcode, go to <em>File\New\New Project</em>, choose the <em>iOS\Application\Empty Application</em> template, and click <em>Next</em>, as shown in the screenshot below:

<a href="http://www.raywenderlich.com/33669/overview-of-ios-crash-reporting-tools-part-1/create_empty_application" rel="attachment wp-att-36867"><img src="http://cdn5.raywenderlich.com/wp-content/uploads/2013/04/create_empty_application-700x471.png" alt="create_empty_application" width="650" /></a>

On the next page, feel free to put in any name for the application. The details don’t matter because this is just a throwaway app to create an archive with a binary and dSYM in it. Click <em>Next</em>, then choose a location to save your project. When Xcode presents the workspace, select <em>iOS Device</em> as the compilation target, as shown below:

<img title="Scheme in Xcode" src="http://cdn5.raywenderlich.com/wp-content/uploads/2013/02/scheme.png" alt="Scheme in Xcode" width="436" height="73" />

Finally, select <em>Product\Archive</em>, as shown below:

<img title="Archive action in Xcode" src="http://cdn2.raywenderlich.com/wp-content/uploads/2013/02/archive.png" alt="Archive action in Xcode" width="212" height="179" />

This will generate an archive for your application, which can then be found in the Organizer.
<h2>Find that dSYM!</h2>
Go to the Organizer, by selecting <em>Window\Organizer</em>, select the <em>Archives</em> tab and then find either an archive for one of your own apps or the test app you made if you followed the previous section. In case you’re struggling to find the archive, here’s a screenshot of what the Organizer looks like:

<img title="Archived app in the Organizer" src="http://cdn5.raywenderlich.com/wp-content/uploads/2013/02/archivedApps-e1361874512820.png" alt="Archived app in the Organizer" width="693" height="455" />

Right-click on your choice of archive, and choose <em>Show in finder</em>. This will open up a Finder window with a file whose extension is .xcarchive. This is not a file, per se, but rather a folder.
<div>

<em>Note:</em>Just like archives, lots of “files” on Mac OS X and iOS are actually folders. Examples include applications (.app), iPhoto libraries (.photolibrary) and code frameworks (.framework).

</div>
To inspect the archive, first open a new terminal window. To open a terminal window, open a new Finder window, navigate to your Applications folder, then navigate to the Utilities subfolder, and finally run the Terminal application. Or you could simply type “terminal” into Spotlight and press return when it’s found Terminal. :]

Once Terminal is open, drag the .xcarchive file into the terminal window and you’ll see the complete path to the folder. Press <em>Control-A</em> to jump to the start of the path and type: <code>"cd "</code> (note that there is a space after the “cd”).

Next, press enter. You’ll notice that the terminal prompt (the bit before where the cursor sits) now has the name of the .xcarchive before the dollar sign.

Next, type the following command: <code>"find . -type d"</code>. You should see some output similar to the following:
<div>
<table>
<tbody>
<tr id="p336692">
<td id="p33669code2">
<pre>$ find . -type d
.
./Products
./Products/Applications
./Products/Applications/breezi.app
./Products/Applications/breezi.app/_CodeSignature
./Products/Applications/breezi.app/en.lproj
./dSYMs
./dSYMs/breezi.app.dSYM
./dSYMs/breezi.app.dSYM/Contents
./dSYMs/breezi.app.dSYM/Contents/Resources
./dSYMs/breezi.app.dSYM/Contents/Resources/DWARF</pre>
</td>
</tr>
</tbody>
</table>
</div>
The command you just typed is using the “find” program, which does exactly what it sounds like – it finds things! The “.” means “search in the current directory” and the “-type d” means “find things that are directories”. In the resulting list of files, notice the <code>.dSYM</code> directory, along with the <code>.app</code> directory. Those are what you’ll need to finish the symbolication process and tie the crash report to your code.

It’s the .dSYM that’s really the most important bit. Recall from earlier that a release build has all the symbols stripped from the binary, which is why crash reports from release builds contain only hexadecimal addresses. The .dSYM contains all the information required to convert back into symbols through the process of symbolication.

Xcode has a method of symbolication built into it and it’ll generally do a good job of making sure any crash log you ever see has been symbolicated. It uses spotlight to find the appropriate .dSYM file for the crash log in question, but sometimes it can’t find it (maybe because you didn’t have it on your computer at the time as the build was made on a build server). In these cases, you can force Xcode to re-symbolicate once you have the .dSYM copied to your computer. To force Xcode to re-symbolicate a crash you can do so in the Organizer. Go to the Organizer again, select the “Devices” tab and then select “Device Logs” on the left. Then select the crash log that needs re-symbolicating and press the button at the bottom of the screen:

<img src="http://cdn3.raywenderlich.com/wp-content/uploads/2013/05/Resymbolicate-button.png" alt="Resymbolicate button" width="257" height="82" />

It’s not just Xcode that uses the .dSYM to symbolicate crash reports. It’s also required by crash reporting tools. As you’ll see throughout the reviews of the various crash reporting tools, the way you get this file to the crash reporting tool is different, but it’s always required for symbolication. There’s no point having crash logs if you can’t read them after all!
<h2>Making a Case for Crash Reporting</h2>
It’s an <em>incredibly</em> clunky process to collect crash logs manually and since a user will probably not have Xcode installed, they would have to use iTunes to access the crash logs. While not a technique I recommend asking your users to do, here’s how you’d do it in the absence of crash reporting tools:
<ol>
	<li>Connect the device to the Mac, synch it via iTunes and navigate to the logs folder, which is different on each operating system:
<ul>
	<li><em>OS X</em>: ~/Library/Logs/CrashReporter/MobileDevice/(your iPhone’s name)/(your app name)</li>
	<li><em>Windows XP</em>: C:\Documents and Settings\Application Data\Apple computer\Logs\CrashReporter\(your iPhone’s name)\(your app name)</li>
	<li><em>Windows Vista</em>: C:\Users\AppData\Roaming\Apple computer\Logs\CrashReporter\MobileDevice\(your iPhone’s name)\(your app name)</li>
</ul>
</li>
	<li>Look for files with the extension of “.crash”.</li>
	<li>Ask the user to send those files to you.</li>
	<li>Once you have the files, open the Organizer in Xcode and select “Device Logs” on the top left, as shown below:<img title="Device Logs item in Organizer" src="http://cdn5.raywenderlich.com/wp-content/uploads/2013/02/devLog.png" alt="Device Logs item in Organizer" width="188" height="96" /></li>
	<li>At the bottom select import to add the log to Xcode and, if necessary because Xcode hasn’t noticed the log yet, trigger a re-symbolication as explained earlier.</li>
</ol>
Phew! Can you imagine any user volunteering to do this for you?

This solution is pretty tedious, to say the least, and it requires the active involvement of your users. That might be OK when you’re in the beta testing phase, and your testers expect a bit of weirdness with the app. However, once you’re in production don’t expect that your users will be happy to go through all that work to give you a crash report!

Interestingly enough, Apple helps out and collects crash reports for applications that are published through the App Store. However, it only happens for users who opted to automatically send diagnostic data to Apple. If you have an app available through, the App Store login to <a href="http://itunesconnect.apple.com">iTunes Connect</a>, select “Manage Your Applications”, choose one of your applications and the click the blue button “View details”, as shown below:

<img title="Application as Shown on the App Store" src="http://cdn5.raywenderlich.com/wp-content/uploads/2013/02/appStore.png" alt="Application as Shown on the App Store" width="479" height="272" />

This will open a detailed view of your app’s properties. At the top right there is a button “Crash Reports”, as shown in the following screenshot:

<img title="Locating Crash Reports on the App Store" src="http://cdn2.raywenderlich.com/wp-content/uploads/2013/02/appStore2-700x201.png" alt="Locating Crash Reports on the App Store" width="700" height="201" />

This section will contain the crash logs collected by Apple. In my experience, these aren’t collected as frequently as with other crash reporting tools, probably because not all that many people have automatic sending of diagnostic reports turned on. Plus, they still require some manual work from developers, like importing and symbolicating.

The smart developer, who would rather spend time writing code than manually retrieving and symbolicating crash dumps, would prefer a process like this:
<ol>
	<li>Release an application (either in beta or in production) which is able to capture and store crash logs.</li>
	<li>The application periodically (or even immediately!) sends crash reports to a server which collects and organizes them for you.</li>
	<li>The developer logs in to the server, finds the crash reports already symbolicated, and can easily find the root cause of the issue.</li>
</ol>
It’s not a far-fetched dream — tools like this do exist in the real world! However, there are quite a few to choose from. Some are free and open source, while others are commercial products. Of the many available, I’ve reviewed some of the most prominent offerings:
<ul>
	<li>Crashlytics</li>
	<li>Crittercism</li>
	<li>Bugsense</li>
	<li>TestFlight</li>
	<li>HockeyApp</li>
</ul>
Have a read through the reviews to see which one provides the particular solutions you’re looking for in your app. I have my own favorites, but I’d love to hear your thoughts on your personal choices in the comments section!
<h2>Crashlytics</h2>
Recently bought by Twitter, Crashlytics is pretty famous in the iOS community. It’s used by well-known companies such as Path and Yammer. It is a full-stack service, meaning that the framework provides both client-side and server-side parts. At the moment, Crashlytics supports only iOS, although the website does indicate that Android support is coming soon.

<em>Crashlytics Setup and Dashboard</em>

Once you have logged into the Crashlytics site, you’ll be prompted to download a Mac application. This application will guide you through the process of configuring your iOS application to work with the Crashlytics service. The application works like a wizard. It first asks you to pick an Xcode project on your disk. Next, it will install the Crashlytics framework, and finally, it will add a step to the building phase of your project.

Any crashes captured will appear on the Crashlytics back-end. Just login, select the application you’re interested in, and you’ll see something like this:

<img title="The Dashboard on Crashlytics" src="http://cdn2.raywenderlich.com/wp-content/uploads/2013/02/crashlytics-700x376.png" alt="The Dashboard on Crashlytics" width="700" height="376" />

The data is definitely well-organized. At a glance, you can see the issues reported, number of crashes, and the number of users affected. On the right, a graph shows the distribution of crashes over time. Each crash is classified by application version to avoid any confusion.

<em>Crashlytics Reports</em>

Crashlytics displays the list of crashes at the bottom, already symbolicated, and you can immediately see the line generating the crash. In the example above, you can quickly see that line 343 in SMEngine.m was where the crash occurred.

If you click on a crash entry, further details are displayed, as shown in the screenshot below:

<img title="A Log Report on Crashlytics" src="http://cdn2.raywenderlich.com/wp-content/uploads/2013/02/crashlytics2-e1361875571818.png" alt="A Log Report on Crashlytics" width="700" height="990" />

At the top of the report, there are contextual details about the average environment where the crash occurred, like the free space on the device, which is handy if you are caching data on disk; RAM, which is handy if you are caching data in memory; and whether the device is jailbroken, along with other juicy details.

At the bottom, you’ll see a stack trace with the sequence of calls that occurred right before the app crashed. As noted, it is very likely some of your code is causing the crash, so you should look for the names of your classes and methods. In this example, the crash is in <code>getHourlyForecast</code>, which is called by <code>parseResultForConnection</code>, which in turn is called by <code>connectionDidFinishLoading</code>.

Once you have fixed the bug and deployed the new version of the app, you can mark the issue as closed, using the control shown below:

<img title="Close an Issue on Crashlytics" src="http://cdn5.raywenderlich.com/wp-content/uploads/2013/02/crashlytics3-700x148.png" alt="Close an Issue on Crashlytics" width="700" height="148" />

Once an issue is closed, it won’t appear again in the list of crashes that require your attention.

<em>Crashlytics 3rd Party Integration</em>

You can integrate Crashlytics with third party bug trackers and project management tools, including the following:
<ul>
	<li><a href="http://campfirenow.com">Campfire</a></li>
	<li><a href="http://www.atlassian.com/software/jira/overview">JIRA</a></li>
	<li><a href="http://www.pivotaltracker.com">Pivotal Tracker</a></li>
	<li><a href="http://www.redmine.org">Redmine</a></li>
	<li><a href="http://www.pagerduty.com">PagerDuty</a></li>
</ul>
Simply enter your credentials for the above services, and Crashlytics servers will forward crash log data to the appropriate service. Cool! :]

For those developers who love to craft their own solution, you can even set up a web hook. This is a custom URL where Crashlytics will send you data in JSON format, and you’re then free to manipulate the data as you like.

The logging functions that Crashlytics provides are worth mentioning as well. Once you have configured your project for Crashlytics, you can quickly set up a macro to log events, actions and most importantly, the values of variables, which can be helpful while hunting for the source of a crash.

This is the equivalent of using <code>NSLog</code> statements, except instead of logging to the console, the lines are sent to Crashlytics. How many times have you wished you could see the console output as your customers use your app in the real world? That’s a reality now! :]

You can also set up notifications via email to receive messages about crashes either as they are received, or as a daily email digest.

<em>Crashlytics Usage Tiers</em>

Previously, Crashlytics had two usage tiers: a free tier, as well as a paid enterprise tier. However, the enterprise tier is now free as well. Thanks Twitter!

To go back to the restaurant analogy, Crashlytics is a good restaurant with a proficient kitchen (symbolication). The food is good and the wait staff are attentive (browsability and usability of logs on the server). Unfortunately, there is not much variety in the menu (just iOS). There is also a wait list, much like a very good restaurant, which means you may have to wait to be seated. However the wait time at the moment doesn’t appear to be very long and you are often seated within minutes.
<h2>Crittercism</h2>
Crittercism is another full-stack tool to keep track of your crash logs. It has been adopted by companies such as Netflix, Eventbrite and Linkedin. It provides support for iOS, as well as Android, HTML5 and Windows 8 (which is in beta at the moment).

<em>Crittercism Setup and Dashboard</em>

Setup is pretty simple. You create an account, create an application on the server, download an SDK, add it to your project, and initialize it. Then you’re ready to rock!

The screenshot below shows the dashboard view of Crittercism, along with a symbolicated crash log:

<img title="An example of Crittercism log" src="http://cdn4.raywenderlich.com/wp-content/uploads/2013/02/crittercism1-e1361877259704.png" alt="An example of Crittercism log" width="700" height="677" />

With Crittercism, not every detail is available at first glance. For example, to view the application version or contextual data, you have to navigate through the tabbed menu in the middle of the page.

The “breadcrumbs” feature of Crittercism allows you to place log statements throughout your code to get contextual information about what happened before the crash, as shown below:

<img title="Log statements on crittercism" src="http://cdn1.raywenderlich.com/wp-content/uploads/2013/02/crittercism2-700x359.png" alt="Log statements on crittercism" width="700" height="359" />
<div>

<em>Note</em>: Breadcrumbs is a paid enterprise feature of Crittercism, but it’s included in the trial period if you want to get a feel for how it works.

</div>
Like other tools, you can mark a crash log as “known” or “solved”. Finally, the SDK includes the possibility to schedule a “Rate My App” popup via the backend, with the ability to customize when the popup appears, as well as the message displayed.

<em>Crittercism Incident Mapping</em>

Another interesting feature of Crittercism is the map that shows you where your logs were recorded, as shown below:

<img title="Distribution on log on a map on Crittercism" src="http://cdn3.raywenderlich.com/wp-content/uploads/2013/02/crittercism3-700x334.png" alt="Distribution on log on a map on Crittercism" width="700" height="334" />

Personally, I have some concern about this, since the user is never asked for permission to calculate a user’s current position. You can use your own judgment call on this.

You can receive email notifications when a crash log is uploaded to the server, and you can set up alarms to receive SMS or email messages when crash counts pass a given threshold.

<em>Crittercism 3rd Party Integration</em>

As far as third party integration goes, you can hook Crittercism up to <a href="http://www.helpshift.com/">HelpShift</a> or <a href="https://www.uservoice.com">Uservoice</a>. Both are customer support help-desk applications.

<em>Crittercism Usage Tiers</em>

There are three pricing tiers for Crittercism at the moment, as shown below:

<a href="http://www.raywenderlich.com/33669/overview-of-ios-crash-reporting-tools-part-1/pricing-tiers" rel="attachment wp-att-40130"><img src="http://cdn5.raywenderlich.com/wp-content/uploads/2013/05/pricing-tiers-700x258.png" alt="Pricing Tiers" width="700" height="258" /></a>

When you sign up, you are offered a 30-day free trial which includes enterprise level features like breadcrumbs and phone support.

The price of the full-tier enterprise solution isn’t published — you’ll have to contact Crittercism directly to get enterprise pricing after the trial expires.

Crittercism is an interesting restaurant. The menu is very rich (offering iOS, Android, HTML5 and Windows 8 in beta) but the wait staff is not really friendly (requiring manual upload of dSYM files, and the usability of the website isn’t the best).
<h2>Bugsense</h2>
Bugsense is another full-stack service, used by big companies such as Samsung, Intel and Groupon. It supports iOS, Android, Windows 8, Windows Phone and HTML5.

<em>Bugsense Setup and Dashboard</em>
The setup is pretty similar to other platforms: create an account, create an application, download an SDK, include it in your project, and set the API key.

The Bugsense dashboard for an application looks like this:

<a href="http://www.raywenderlich.com/33669/overview-of-ios-crash-reporting-tools-part-1/bug_sense_dashboard" rel="attachment wp-att-40139"><img src="http://cdn4.raywenderlich.com/wp-content/uploads/2013/05/bug_sense_dashboard-700x377.png" alt="bug_sense_dashboard" width="700" height="377" /></a>

If you click one of the logs, you are presented with a detailed view, like so:

<a href="http://www.raywenderlich.com/33669/overview-of-ios-crash-reporting-tools-part-1/crash-detail" rel="attachment wp-att-40142"><img src="http://cdn3.raywenderlich.com/wp-content/uploads/2013/05/crash-detail-700x385.png" alt="Crash Detail Page" width="700" height="385" /></a>

You can see the class that caused the crash — in this case, <code>NSInvalidArgumentException</code> — the function generating it, and the corresponding line of code (SMViewController.m:26).

<em>Bugsense Crash Reports</em>

The log presentation is chock-full of detail, and the user interface allows the ability to customize which attributes are displayed. It also displays the number of times a crash has occurred. As usual, you can mark issues as resolved.

One of the downsides of Bugsense is that you have to manually upload the dSYM file of your build to allow symbolication on the server-side. You <i>can</i> configure the app to run symbolication right on the device, although the documentation discourages it because you don’t actually get full information such as code line numbers.

To keep track of the application’s context during run-time, you can drop breadcrumbs into your code at key points and they will be uploaded to the server, together with data about the crash. You can also use events to perform this functionality, but they still look pretty similar to breadcrumbs.

<em>Bugsense Push Notifications</em>

An interesting feature is “Fix notifications”. This feature allows you to send your users a push notification to indicate that an upgrade is available for the app. This is pretty handy when you have resolved a bug and released a new version, but some of your users is still running an old version.

<em>Bugsense 3rd Party Integration</em>

The backend of Bugsense can be integrated with <a href="http://www.atlassian.com/software/jira/overview">JIRA</a> to push data about crash logs. Like other tools, you will receive email notifications when the platform receives a crash log.

<em>Bugsense Usage Tiers</em>

There are four pricing tiers for Bugsense as shown below:

<a href="http://www.raywenderlich.com/33669/overview-of-ios-crash-reporting-tools-part-1/pricing_details" rel="attachment wp-att-40144"><img src="http://cdn1.raywenderlich.com/wp-content/uploads/2013/05/pricing_details-510x500.png" alt="Pricing Details" width="510" height="500" /></a>

Bugsense is a very nice restaurant. The menu is really varied (iOS, Android, Windows 8, Windows Phone and HTML5) and well organized. The wait staff is not always impeccable, but given the other great features of the restaurant, it’s something you can manage to live with.
<h2>TestFlight</h2>
TestFlight was born as a tool to manage the distribution of beta releases. Over time the developers added many more features, like action logging and crash reporting. It’s like a restaurant that has a bar or a club to entertain the client before or after dinner.

TestFlight has been adopted by companies such as Adobe, Instagram and tumblr to manage over-the-air deployment, tracking and crash reporting. Both iOS and Android are supported at present.

<em>TestFlight Setup and Dashboard</em>

Getting started with TestFlight is similar to other crash reporting services: create an account, create an application, download and integrate the SDK, and finally, setup an application key.

On the server-side, you have to produce a build of your app, upload the <code>.ipa</code> file of your distribution, at which point you can ping your testers to download the new build. This can either be done directly through the TestFlight website, or through the <a href="http://testflightapp.com/desktop/">supplied Mac app</a>. This app also determines when Xcode’s archive process has been completed, then prompts you to upload the new build.

The dashboard on the server-side looks like this:

<img title="Dashboard on testflight" src="http://cdn2.raywenderlich.com/wp-content/uploads/2013/02/testflight-700x439.png" alt="Dashboard on testflight" width="700" height="439" />

A handy menu on the left allows you to browse different aspects of your build like sessions, checkpoints crashes (similar to breadcrumbs), and user feedback.

<em>TestFlight User Feedback</em>

TestFlight provides a feedback view in your application to collect feedback from your testers. The feedback view appears to users like this:

<img title="Feedback view on TestFlight" src="http://cdn2.raywenderlich.com/wp-content/uploads/2013/02/testflight-feedback.png" alt="Feedback view on TestFlight" width="320" height="480" />

<em>TestFlight Crash Reports</em>

A crash log on the web server looks like this:

<img title="A Crash log on TestFlight" src="http://cdn4.raywenderlich.com/wp-content/uploads/2013/02/tf-700x332.png" alt="A Crash log on TestFlight" width="700" height="332" />

As noted previously, you can log when a user reaches a critical point in your app, such as opening a particular view. This provides you with contextual information which is useful when hunting for the root cause of a crash.

<em>TestFlight 3rd Party Integration</em>

At the moment there is no integration with third party tools, although you can hook up with the upload API to automate the upload phase.

<em>TestFlight Usage Tiers</em>

The TestFlight service is free at the moment; there’s no separate free or paid tiers. As for the restaurant analogy — this restaurant/bar/club is an interesting place to hang out if you want to spend the whole night in one spot. The menu is pretty limited (iOS and Android only), and the neighborhood is bit desolate (there’s no integration with third-party tools).
<h2>HockeyApp</h2>
HockeyApp is pretty well known in the indie developer world — probably because it’s made by indie developers! :]. It supports iOS, Android, MacOS, and Windows Phone. Like TestFlight, it’s a restaurant with perks, not just food. In fact, besides crash reporting and just like TestFlight, it includes distribution management.

<em>HockeyApp Setup and Dashboard</em>

Once you’ve created an account you are invited to create an app and download the corresponding SDK. The setup is pretty standard: import a framework, set up the API key and you are ready to go. Alternatively, you can opt for the more complicated approach of including the full source to the framework instead. This is good to know that option exists, incase you find a bug in the framework and desperately need the fix before the HockeyApp team fix it themselves. That’s a rare occurrence, but it’s good to know that the option exists!

Here’s what the HockeyApp dashboard looks like:

<img title="Dashboard on hockeyapp" src="http://cdn1.raywenderlich.com/wp-content/uploads/2013/02/hockeyapp-e1361882100809.png" alt="Dashboard on hockeyapp" width="699" height="543" />

The dashboard gives you an overview of your builds with a handy menu at the top to show different sections of the dashboard along with a nice graph of various stats at the bottom.

The companion desktop application detects when the archive procedure has finished and prompts you to upload the new build, including the dSYM. Once you’ve manually uploaded the dSYM file to the server, you can select the Crashes tab, pick a log and you’ll see a symbolicated crash, as below:

<img title="A crash log on hockeyapp" src="http://cdn2.raywenderlich.com/wp-content/uploads/2013/02/hockeyapp2-700x440.png" alt="A crash log on hockeyapp" width="700" height="440" />

The HockeyApp back-end provides the ability to perform advanced searches through your logs, using criteria like “show all of the crashes that happened on iOS6 but not on an iPad”.

<em>HockeyApp Crash Uploads</em>

A unique feature is the ability to upload crashes you receive via email from your users. This is made possible by the fact that the HockeyApp crash log format is similar to the format adopted by Apple. You can also log events, much like using breadcrumbs, and attach them to a crash log via a simple API.

<em>HockeyApp 3rd Party Integration</em>

The integration with third-party tools is quite rich. HockeyApp integrates with the following tools and services, among others:
<ul>
	<li><a href="http://www.atlassian.com/software/jira/overview">JIRA</a></li>
	<li><a href="https://github.com">GitHub</a></li>
	<li><a href="http://www.mantisbt.org">Mantis</a></li>
	<li><a href="http://lighthouseapp.com">Lighthouse</a></li>
	<li><a href="http://www.redmine.org">Redmine</a></li>
	<li><a href="http://trac.edgewall.org">Trac</a></li>
	<li><a href="http://basecamp.com">Basecamp</a></li>
	<li><a href="http://www.pivotaltracker.com">Pivotal tracker</a></li>
</ul>
Like all other crash reporting tools, you can set the frequency and type of email notifications that you receive from HockeyApp.

<em>HockeyApp Usage Tiers</em>

HockeyApp boasts four pricing tiers at the moment, with discounts available if you sign up for yearly subscriptions:

<img title="Pricing tiers of hockeyapp" src="http://cdn4.raywenderlich.com/wp-content/uploads/2013/02/hockeyapp3-700x353.png" alt="Pricing tiers of hockeyapp" width="700" height="353" />

It’s worth mentioning that HockeyApp is the hosted version of the open source solution Quincykit. If you’re keen on setting up your own back-end to collect logs, check out <a href="http://quincykit.net">http://quincykit.net</a>.

HockeyApp is a nice restaurant and the organization and variety of the menu is great (supporting iOS, Android, MacOS, and Windows Phone). The neighborhood (3rd party integration) is lovely and well-populated.
<h2>Summary and Comparison Chart</h2>
I’ve provided a table below comparing the features of the tools reviewed above:

<img title="Comparison table of crash reporting tools" src="http://cdn2.raywenderlich.com/wp-content/uploads/2013/05/table.png" alt="Comparison table of crash reporting tools" width="700" height="600" />
<h2>The Bottom Line</h2>
If most of your development work is on iOS, the best crash reporting tool in my opinion is Crashlytics. It’s free, with tons of features and a very usable back-end. Moreover, the reporting process is fully automated, as you get logs on the server with no need to manually upload dSYM files for each release. One drawback is that it doesn’t manage distribution of your app.

If you’re looking for a complete service cross-platform choice, I’d suggest Bugsense, because of the usability of its dashboard. Be aware, though — the cheaper tiers only retain data for a short period, ranging from 7 to 30 days.

However, all of the services reviewed above are still valid alternatives, depending on the features and usability you’re looking for.

In the second part of this article, I will show how to get started with each of these services, integrate it with your app, and give you a tour of the various crash logs and other features.

Thanks for joining me for the second part of this two-part series on crash reporting services!

The <a href="http://www.raywenderlich.com/33669/overview-of-ios-crash-reporting-tools-part-1">first part</a> introduced you to the architecture of crash reporting services, including storage, symbolication, and server-side management. As well, I provided a basic overview and comparison chart of the most popular crash reporting services today.

In this second part, I’ll take you through the steps to get started with each service covered in the last article: Crashlytics, Crittercism, Bugsense, TestFlight and HockeyApp.

&nbsp;

Throughout this article, you will work with a very simple iPhone application that just contains a table view, as shown in the screenshot below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/App.png"><img title="The sample App for this Tutorial" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/App.png" alt="The sample App for this Tutorial" width="320" height="480" /></a>

This particular app has been constructed to generate crash events. That’s bad news in the real word — but for our purposes, it will serve perfectly!
<h2>Getting Started</h2>
The app provides a pizza menu with two functions:
<ol>
	<li>Swipe to delete a pizza from the menu.</li>
	<li>Scroll to the bottom to load more pizzas.</li>
</ol>
The code for our pizza application can be found <a title="Starter project for this tutorial" href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashy-starter.zip">here</a>.

Download the project, then build and run in the simulator. To cause a crash, swipe to delete a row, or simply scroll to the bottom of the list.
<div>

<em>Note</em>: The sample app is designed to run on iOS6 and above. If you are testing on an pre iOS6 device, you will need to disable AutoLayout.

To do so, open <em>SMViewController.xib</em>, select the File Inspector, then uncheck “Use AutoLayout”, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/05/disable_autolayout.png"><img title="disable_autolayout.png" src="http://www.raywenderlich.com/wp-content/uploads/2013/05/disable_autolayout.png" alt="Disable AutoLayout" width="600" height="266" border="0" /></a>

Now when your app crashes, you’ll really mean for it to happen! :]

</div>
Although I know it’s killing you not to fix the bugs, don’t! You want the application to crash so that a reporting tool can help you to identify the source of the problem.

Here are a list of the crash reporting services that I’ll cover in this article:
<ul>
	<li>Crashlytics (<a href="https://www.crashlytics.com/" target="_blank">www.crashlytics.com)</a></li>
	<li>Crittercism (<a href="https://www.crittercism.com/" target="_blank">www.crittercism.com</a>)</li>
	<li>Bugsense (<a href="http://www.bugsense.com/" target="_blank">www.bugsense.com</a>)</li>
	<li>TestFlight (<a href="https://www.testflightapp.com/" target="_blank">www.testflightapp.com</a>)</li>
	<li>HockeyApp (<a href="http://www.hockeyapp.net/" target="_blank">www.hockeyapp.net</a>)</li>
</ul>
Keep in mind that there is a waiting period for Crashlytics where it may take several days for Crashlytics to contact you. However, if it does take a long time then no fear because Crashlytics has generously provided RayWenderlich.com with <a href="http://try.crashlytics.com/?s=magnolia">a special link</a> that will help you jump the queue. Try that and see if you get in the VIP door a little faster! :]

Before continuing, make sure you create an account with each of the crash reporting frameworks and download each of the crash reporting SDKs.

Since some crash reporting tools require a unique identifier to function correctly, make sure to change the sample project’s bundle identifier to something of your own design, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/05/change_bundle_identifier.png"><img title="change_bundle_identifier.png" src="http://www.raywenderlich.com/wp-content/uploads/2013/05/change_bundle_identifier.png" alt="Change bundle identifier" width="600" height="358" border="0" /></a>

Once you have a set a custom bundle identifier, make sure to make a backup copy of the project. Each framework has their own installation instructions so it is best to use a “vanilla” project with each demo.

In order to test out these crash reporting frameworks, you will need to run the app on a real device. This requires an Apple developer account as well as a provisioned device. If you are a new to iOS development, you can learn how to create a developer account, as well as provisioning your own device in <a href="http://www.raywenderlich.com/8003/how-to-submit-your-app-to-apple-from-no-account-to-app-store-part-1">this tutorial</a>.
<div>

<em>Note</em>: During this tutorial, do not run the application on the device through Xcode. Xcode will intercept the crash, then open the lldb debugger shell as usual. You won’t get any crash reports if Xcode intercepts the crash event!

To make all the examples below work, you have to build and run the application, then click the stop button on Xcode. This way you will have the latest version installed on the the device.

Once that is done, you can launch the app on the device itself, and then crash it all you want!

All the crashes on your iOS device will be caught and sent to the server component of the service that you have integrated into the app. Crash reports are usually sent to the server the next time you start the app, so the steps to follow to generate a crash report on the server are as follows:
<ol>
	<li>Build and run on Xcode.</li>
	<li>Press the stop button.</li>
	<li>Run the app on your iOS device.</li>
	<li>Make the app crash.</li>
	<li>Run the app again.</li>
</ol>
</div>
As well, make sure the device has connectivity via wi-fi or 3G so that the crash reports can be sent. Now, let’s get crashing! :]
<h1>Crashlytics</h1>
Recently bought by Twitter, Crashlytics is pretty famous in the iOS community. It’s used by well-known companies such as Path and Yammer. It is a full-stack service, meaning that the framework provides both client-side and server-side parts.

At the moment, Crashlytics supports only iOS, although the website does indicate that Android support is coming soon.
<h2>Crashlytics — Configuring the Project</h2>
Once you have logged into Crashlytics, you will be prompted to download a Mac OSX application that will help you to set up your first project.

To save you time, here’s the <a href="https://www.crashlytics.com/download/mac">direct link</a> to the Mac application. When running, the application appears in the menu bar on the top right as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics1.png"><img title="Crashlytics in the menu bar" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics1.png" alt="Crashlytics in the menu bar" width="188" height="23" /></a>

Click on the icon and enter your credentials when prompted. If this is your first time using Crashlytics, you should see an empty list of applications and your account name at the bottom, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics2.png"><img title="Crashlytics after login" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics2.png" alt="Crashlytics after login" width="333" height="553" /></a>

Click on the <em>New App</em> button on the right. This will show you the list of recent projects opened in Xcode, like so:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics3.png"><img title="Recent Projects in Crashlytics" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics3.png" alt="Recent Projects in Crashlytics" width="353" height="561" /></a>

If your app is not listed, you’ll need to perform the following steps:
<ol>
	<li>Click the “Other” button at the bottom to open a Finder window.</li>
	<li>Select the .xcodeproj file and click “Open”.</li>
	<li>Select your project from the list in Crashlytics and hit “Next”.</li>
</ol>
This will automatically open the project in Xcode and ask you to add a script to the Build Phase. From this point onward, the Crashlytics app will walk you through each step of installation process. Follow the instructions, and when you’re done, continue with the next step of the tutorial.
<h2>Crashlytics — Running the App</h2>
Now that you’ve completed the setup portion, you are now ready to try out Crashlytics. Run the application on your iOS device, swipe on a cell and tap <em>Delete</em>. The application will crash, as expected.

Restart the app and wait about a minute to make sure the crash report has propagated to the Crashlytics server. Then take a peek at the website to see if the crash report is live. You should also receive an email notification about a new crash in your application.

The new issue on the back-end should look like the following:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics10.png"><img title="First issue on Crashlytics" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics10-700x282.png" alt="First issue on Crashlytics" width="700" height="282" /></a>

At first glance, you will see the guilty line of code (<em>SMViewController.m</em> at line 80), the number of users affected, and the number of occurrences of this crash. Launch the app and make it crash again. You’ll now see the number of crashes of the same issue increase.

Click the row of that issue and you’ll see the extended report, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics11.png"><img title="Detailed view of the report on Crashlytics" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics11-e1362321061492.png" alt="Detailed view of the report on Crashlytics" width="699" height="1001" /></a>

Looking at the crash log, it’s clear that you need to counterbalance the deletion of a cell with the deletion of its corresponding element in the array populating the table view. To fix this bug, open <em>SMViewController.m</em> and modify <code>tableView:commitEditingStyle:forRowAtIndexPath:</code> to look like the following code:
<div>
<table>
<tbody>
<tr id="p340501">
<td id="p34050code1">
<pre>- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSIndexPath_Class/">NSIndexPath</a> *)indexPath
{
    if (editingStyle == UITableViewCellEditingStyleDelete) {
        [pizzaOrder removeObjectAtIndex:indexPath.row];
        [tableView deleteRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationFade];
    }
}</pre>
</td>
</tr>
</tbody>
</table>
</div>
After you are done, build and run, push the app to your device, and test the application again. Once you have determined that you have fixed the issue, return to the Crashlytics crash report then set the issue status as closed, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/05/crashlytics_close_bug.png"><img title="crashlytics_close_bug.png" src="http://www.raywenderlich.com/wp-content/uploads/2013/05/crashlytics_close_bug.png" alt="Crashlytics close bug" width="600" height="510" border="0" /></a>

On to the second bug!

Crashlytics allows you to add log statements to your code to help track down bugs. Open <em>SMViewController.m</em> and add the following import statement at the top of the file:
<div>
<table>
<tbody>
<tr id="p340502">
<td id="p34050code2">
<pre>#import &lt;Crashlytics/Crashlytics.h&gt;</pre>
</td>
</tr>
</tbody>
</table>
</div>
Next, modify <code>tableView:willDisplayCell:forRowAtIndexPath:</code> in <em>SMViewController.m</em> as follows.

&nbsp;
<blockquote>
<div>- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSIndexPath_Class/">NSIndexPath</a> *)indexPath { if (indexPath.row == pizzaOrder.count-1) { CLS_LOG(@"posting notification"); [Crashlytics setIntValue:pizzaOrder.count forKey:@"numberOfPizzas"]; [[<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSNotificationCenter_Class/">NSNotificationCenter</a> defaultCenter] postNotificationName:LOAD_MORE_NOTIFICATION object:nil]; } }</div></blockquote>
<em>CLS_LOG</em> is a macro provided by the Crashlytics framework that enables remote logging. In debug builds, it acts just like NSLog, passing strings to the console. For release builds, the log is sent along with the crash reports and optimized to be as fast as possible. In fact, Crashlytics’ own <a href="http://support.crashlytics.com/knowledgebase/articles/92519-how-do-i-use-logging-" target="_blank">documentation</a> boasts a 10x improvement over using regular <code>NSLog()</code> calls.

The method also uses custom logging which allows you to store information in key-value form. Think of this as a type of NSDictionary on the web server: you set the values of keys in the code, and you can read the values on the server when hunting down a bug.

In this case you are using <em>setIntValue:forKey</em>, but there are other methods available to store objects, floats and booleans as well. You can find more details about custom logging in the <a href="http://support.crashlytics.com/knowledgebase/articles/92520-how-do-i-use-custom-keys-" target="_blank">Crashlytics knowledgebase</a>.

Run the application in Xcode and then stop it. Run it again on your device, scroll to the bottom, allow the app to crash, and restart the application after the crash.

The new issue will look like this on the Crashlytics dashboard:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics12.png"><img title="Second issue on Crashlytics" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics12-700x282.png" alt="Second issue on Crashlytics" width="700" height="282" /></a>

It is clear that this is due to the lack of the method definition <code>loadMore</code> in <em>SMWebEngine.m</em>. If you were to add the definition for this method, and the crash would disappear. Although that’s not the point of this tutorial, so don’t worry about actually implementing it!

Notice that the detailed view of each issue provides information about the device like iOS version, and free space on disk and in memory. All these details might help you when you are hunting down the root cause of a crash.

Click on “More details…” in the middle of the page. This will show even more details like the type of device, orientation, plus the keys and log statements that you have spread throughout your code, as shown in the following screenshot:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics14.png"><img title="Logs and keys on Crashlytics" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics14-700x399.png" alt="Logs and keys on Crashlytics" width="700" height="399" /></a>

This way you can have a path of breadcrumbs that can help to hunt down the cause of the crash. It is important to note that logs and keys are sent to the server as attachments to a crash log, so you won’t see them if there aren’t any crashes.

Crashlytics, unlike other services like TestFlight, is not a generic remote logging application, but rather a true crash reporting application. So even though your app may be full of logging statements, if the app never crashes, you’ll never see any of your logs!

One useful feature for beta testing is the ability to ask your users to provide some data to identify themselves. Crashlytics provides three ways to attach this data to a crash log:
<div>
<table>
<tbody>
<tr id="p340504">
<td id="p34050code4">
<pre>[Crashlytics setUserIdentifier:@”123456”];
[Crashlytics setUserName:@”cesarerocchi”];
[Crashlytics setUserEmail:@”cesare@mailaddress.com”];</pre>
</td>
</tr>
</tbody>
</table>
</div>
This way, you can identify who is experiencing a crash, and contact that tester to get more details.

Finally, you can configure some logging features on the back-end. Each application has a settings section that you can open by clicking the gear button, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics15.png"><img title="App settings on Crashlytics" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics15.png" alt="App settings on Crashlytics" width="365" height="106" /></a>

From here you can disable reports from a specific version, or request a user for permission before sending data about the report. When you enable it (and you should!), you can customize the message as shown in the screenshot below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics16.png"><img title="Privacy on Crashlytics" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crashlytics16.png" alt="Privacy on Crashlytics" width="628" height="563" /></a>
<h2>Crashlytics — Summing It Up</h2>
This concludes the guided tour of Crashlytics. If you are just targeting the iOS market, I highly recommend that you consider Crashlytics. The service is very quick to upload crashes, there are no hassles with dSYM files, and the back-end service is quite intuitive.
<h1>Crittercism</h1>
Crittercism is another full-stack tool to keep track of your crash logs. It has been adopted by companies such as Netflix, Eventbrite and Linkedin. It provides support iOS, as well as Android, HTML5 and Windows 8 (which is in beta at the moment).
<h2>Crittercism — Configuring the Project</h2>
To get started with Crittercism, there are a few steps to follow:
<ol>
	<li>Register an application.</li>
	<li>Download and import the SDK.</li>
	<li>Configure the Xcode project.</li>
</ol>
After you login to Crittercism, assuming it’s your first time, you will end up at the following screen:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit1.png"><img title="First step in Crittercism" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit1-700x429.png" alt="First step in Crittercism" width="700" height="429" /></a>

Tap the big blue button on the top left. You’ll be presented with the following screen:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit2.png"><img title="Creating the first app on Crittercism" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit2-e1362321800174.png" alt="Creating the first app on Crittercism" width="700" height="615" /></a>

This will ask you to assign a name; for this project, put “crashy”. Then select the <em>iOS</em> platform, and don’t bother to invite any collaborators. You’re just testing the application, so choose <em>No</em> for the question “In App Store?”. When you’re done, click the big blue <em>Register App</em> button at the bottom.

Next you will be prompted to download the most recent version of the SDK (which is 3.5.1 at the time of this writing). Here’s the link to the <a href="https://www.crittercism.com/downloads/ios">download page</a>, just in case.

Open a new, unfixed copy of the crashy-starter project in Xcode, unzip the Crittercism SDK, drag and drop the folder named <em>CrittercismSDK</em> onto the root of our project, and make sure “Copy items into destination group’s folder” is checked.

Go to the <em>Build Phases</em> tab of the target. Open the <em>Link Binary with Libraries</em> section by clicking on it. You’ll notice that <code>libCrittercism</code> is already linked. Add the SystemConfiguration framework and QuartzCore by clicking on the plus sign, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit3.png"><img title="Framework linking in Crittercism" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit3-700x258.png" alt="Framework linking in Crittercism" width="700" height="258" /></a>

Next, you need to find the App ID. This is the ID for Crittercism and has nothing to do with Apple’s application ID. Head to <a href="https://www.crittercism.com/developers">the Crittercism dashboard</a>, select your application from the list, and click the settings tab on the left. You will see a section that looks like this:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit4.png"><img title="API key on Crittercism" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit4.png" alt="API key on Crittercism" width="519" height="287" /></a>

This provides a portion of code to copy and paste into <code>application:didFinishLaunchingWithOptions:</code> of <em>SMAppDelegate.m</em>. First, add the following import statement at the top of <em>SMAppDelegate.m</em>:
<div>
<table>
<tbody>
<tr id="p340505">
<td id="p34050code5">
<pre>#import "Crittercism.h"</pre>
</td>
</tr>
</tbody>
</table>
</div>
Now add the Crittercism supplied code to the top of <code>application:didFinishLaunchingWithOptions:</code> :
<div>
<table>
<tbody>
<tr id="p340506">
<td id="p34050code6">
<pre>[Crittercism enableWithAppID:@"&lt;YOUR_CRITTERCISM_APP_ID&gt;"];</pre>
</td>
</tr>
</tbody>
</table>
</div>
And that’s it, your project is configured to work with Crittercism!
<h2>Crittercism — Uploading the dSYM</h2>
Unlike Crashlytics, you have to manually upload your dSYM file to the server. To find the dSYM file, follow the instructions in the <a href="http://www.raywenderlich.com/33669/overview-of-ios-crash-reporting-tools-part-1#symbolication">first part</a> of the tutorial in the Symbolication section.

Recall from the <a href="http://www.raywenderlich.com/33669/overview-of-ios-crash-reporting-tools-part-1">first part</a> of this tutorial that a dSYM is actually a directory. For this reason, you’ll need to zip your dSYM folder first before uploading it to Crittercism. To upload your zipped dSYM file use the tab <em>Upload dSYMs</em> in the “Settings” section of your app, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit5.png"><img title="Upload dSYM on Crittercism" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit5-700x320.png" alt="Upload dSYM on Crittercism" width="700" height="320" /></a>
<div>

<em>Note</em>: If you are still having problems uploading your dSYM file, check out this <a href="http://www.youtube.com/watch?v=i2hv35MisgM" target="_blank">YouTube video</a> that screencasts the process, step by step.

</div>
Once you have uploaded the dSYM folder, run your application, either on your simulator or on your device, and the console should print the following message:
<div>
<table>
<tbody>
<tr id="p340507">
<td id="p34050code7">
<pre>Crittercism successfully initialized.</pre>
</td>
</tr>
</tbody>
</table>
</div>
Let’s start by tracking down the first bug.
<h2>Crittercism — Running the App</h2>
As you did before, run the application without Xcode, attempt to delete a cell, and restart the application. Now visit the <em>Crash Reports</em> section of your application on the back-end. You should see your crash report as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit6.png"><img title="Dashboard on Crittercism" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit6-700x368.png" alt="Dashboard on Crittercism" width="700" height="368" /></a>

The dashboard will show an overview of your application with a big graph at the top and the list of issues at the bottom. Just pretend you don’t know the what the source of the bug is, and click the issue in the list to inspect it. You’ll see a detailed report similar to the following:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit7.png"><img title="A crash report on Crittercism" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit7-e1362322220278.png" alt="A crash report on Crittercism" width="699" height="830" /></a>

The “Reason” column should lead you to think there is an issue with the table view. You might see more details, but the log is not yet symbolicated, so you don’t know which file and line of code is responsible for the crash.

You’ll need to add the log to a queue to be symbolicated, by clicking the <em>Add to Symbolication Queue</em> link highlighted in the screenshot above. Once it has been symbolicated, you will see the decoded thread of the crash, which indicated an issue on line 80 in file <em>SMViewController.m</em>, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit8.png"><img title="Decoded log on Crittercism" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit8-700x229.png" alt="Decoded log on Crittercism" width="700" height="229" /></a>

One shortcoming of Crittercism is that I couldn’t find a way to queue crash logs automatically as they get uploaded. If you’ve found a way around this, please let me know in the comments!

If you select the “Users” tab, you’ll notice that Crittercism automatically generates IDs to identify users, as shown in the following screenshot:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit9.png"><img title="User ids on Crittercism" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/crit9-700x168.png" alt="User ids on Crittercism" width="700" height="168" /></a>

You can now mark the issue as resolved using the drop down menu on the left side of the graph as shown below:

<img title="resolve_crittercism_bug.png" src="http://www.raywenderlich.com/wp-content/uploads/2013/05/resolve_crittercism_bug.png" alt="Resolve Crittercism Bug" width="600" height="267" border="0" />

If you’d like to collect more specific data about users, the Crittercism class also provides the following methods:
<div>
<table>
<tbody>
<tr id="p340508">
<td id="p34050code8">
<pre>[Crittercism setUsername:(<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/">NSString</a> *)username];

[Crittercism setEmail:(<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/">NSString</a> *)email];

[Crittercism setGender:(<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/">NSString</a> *)gender];

[Crittercism setAge:(int)age];

[Crittercism setValue:(<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/">NSString</a> *)value forKey:(<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/">NSString</a> *)key];</pre>
</td>
</tr>
</tbody>
</table>
</div>
An interesting feature is the ability to log handled exceptions on the server as follows:
<div>
<table>
<tbody>
<tr id="p340509">
<td id="p34050code9">
<pre>@try {
        [<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSException_Class/">NSException</a> raise:NSInvalidArgumentException
                    format:@"Argument must be a string"];
        } @catch (<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSException_Class/">NSException</a> *exc) {
            [Crittercism logHandledException:exc]
        }</pre>
</td>
</tr>
</tbody>
</table>
</div>
Crittercism also offers breadcrumb logging for enterprise accounts and an assortment of other features.
<h2>Crittercism — Summing it Up</h2>
This concludes the how-to portion on Crittercism. The Crittercism framework also supports Android, Windows 8 and HTML5.

I’ve found that the user interface for the back-end is a bit complex, requiring a few extra clicks to get to the required information. It’s a bit clumsy to manually upload the dSYM for each build, but overall, it’s a solid framework.
<h2>Okay Class — Time for a Break!</h2>
Okay, it’s time for a short break from your crash reporting studies. Answer the following tricky question.
<div>
<table border="0" align="center" bgcolor="FFFFFF">
<tbody>
<tr>
<th>Solution Inside: Is divide-by-zero an operation that causes a crash on ARM?</th>
<th><a id="spoilerDiv65eb8001_action" href="http://www.raywenderlich.com/34050/overview-of-ios-crash-reporting-tools-part-2?utm_source=tuicool">Show</a></th>
</tr>
<tr>
<td colspan="2"></td>
</tr>
</tbody>
</table>
<div>
<table border="0" frame="box" align="center" bgcolor="FFFFFF">
<tbody>
<tr>
<th></th>
<td colspan="2"></td>
</tr>
<tr>
<td colspan="2"></td>
</tr>
</tbody>
</table>
</div>
</div>
<h1>BugSense</h1>
Bugsense is another full-stack service, used by big companies such as Samsung, Intel and Groupon. It supports iOS, Android, Windows 8, Windows Phone and HTML5.
<h2>Bugsense — Configuring the Project</h2>
Once you are logged in to Bugsense, you will end up in the dashboard which should be empty if this is your first time using Bugsense. Click the “Add New Project” button, enter a name for your project, select <em>iOS</em> for Technology, <em>Testing</em> for the stage, then click <em>Submit</em>, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/bugsense1.png"><img title="Creating an Application on Bugsense" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/bugsense1.png" alt="Creating an Application on Bugsense" width="538" height="470" /></a>

The website will then show a dashboard with no data, since you have not yet integrated the framework. When you first visit your project page, a welcoming lightbox will greet you which will contain your API key, as so:

<img title="bugsense_api_key.png" src="http://www.raywenderlich.com/wp-content/uploads/2013/05/bugsense_api_key1.png" alt="Bugsense API Key" width="600" height="468" border="0" />

Copy down this key as you’ll be needing it later!

Unfortunately there is no direct link to download the SDK, so you will have to click the <em>Docs</em> item in the top right bar of the dashboard. This will open the generic documentation page, from which you have to select iOS. You will end up at <a href="https://www.bugsense.com/docs/ios" target="_blank">this link</a>. This page should contain a URL to download the iOS SDK (which is version 3.2 at the time of this writing).

Unzip the downloaded ZIP file and it will create a folder named <em>BugSense-iOS.framework</em>. Next, open a new copy of the starter project and go to the <em>Build Phases</em> tab. As previously, expand <em>Link Binary With Libraries</em>, click the <em>+</em> button, choose <em>Add other</em>, and select the unzipped <em>BugSense-iOS.framework</em> folder. Also add <em>SystemConfiguration</em> and <em>libz.dylib</em>. The link section should now look like the following:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/bugsense3.png"><img title="Framework linking on bugsense" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/bugsense3.png" alt="Framework linking on bugsense" width="645" height="166" /></a>

Switch to the <em>Build Settings</em> tab and make sure that “Strip Debug Symbols During Copy” and “Strip Linked Product” are both set to YES as shown here:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/bugsense4.png"><img title="More project settings on bugsense" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/bugsense4-700x362.png" alt="More project settings on bugsense" width="700" height="362" /></a>

Now you just need to insert a few hooks to integrate your app with Bugsense. Insert the following statement at the top of <em>SMAppDelegate.h</em>:
<div>
<table>
<tbody>
<tr id="p3405010">
<td id="p34050code10">
<pre>#import &lt;BugSense-iOS/BugSenseController.h&gt;</pre>
</td>
</tr>
</tbody>
</table>
</div>
Next, open <em>SMAppDelegate.m</em> and add the following code to the beginning of <code>application:didFinishLaunchingWithOptions:</code>, using the API key you copied down earlier:
<div>
<table>
<tbody>
<tr id="p3405011">
<td id="p34050code11">
<pre>[BugSenseController sharedControllerWithBugSenseAPIKey:@"YOUR API KEY"];</pre>
</td>
</tr>
</tbody>
</table>
</div>
Now you’re ready to test Bugsense with your first bug!
<h2>Bugsense — Running the App</h2>
Just like in the previous sections, build and archive your project, install the application (.ipa file) on the device, and zip the dSYM file for later upload. Once the application is installed, run it, make it crash by deleting a row, and restart the app. Head over to the dashboard of Bugsense and open your project. You’ll see the following report:

<img title="bugsense_user_interface.png" src="http://www.raywenderlich.com/wp-content/uploads/2013/05/bugsense_user_interface.png" alt="Bugsense User Interface" width="600" height="251" border="0" />

A nice graph shows the number of crashes per day. Underneath is the list of error logs — just one in this case. You can immediately see the generic reason behind this issue; click the issue to open the detailed report as shown below:

<img title="symbolicate.png" src="http://www.raywenderlich.com/wp-content/uploads/2013/05/symbolicate.png" alt="Symbolicate" width="600" height="151" border="0" />

The stack trace as shown is complete, but has not yet been symbolicated. Click the <em>Symbolicate</em> link on the top right. In this case, it will fail to symbolicate the trace because the dSYM is not yet on the server, and the button will turn green with a label of “Upload Symbols”.

Click the <em>Upload Symbols</em> button, and you will be redirected to the settings section of the application, where you can upload your zipped dSYM folder, as shown below:

<img title="upload_dsym_files.png" src="http://www.raywenderlich.com/wp-content/uploads/2013/05/upload_dsym_files.png" alt="Upload dSYM Files" width="600" height="283" border="0" />

Once you have uploaded your dSYM files, head back to the detailed view of the error log and again click the <em>Symbolicate</em> link. This time you will see that line 7 is decoded, clearly showing the line responsible for the crash:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/bugsense8.png"><img title="Symbolicated crash on bugsense" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/bugsense8.png" alt="Symbolicated crash on bugsense" width="685" height="62" /></a>

Below the crash log, there is also a handy table showing lots of details about the crash. This table can be customized using the gear icon on the right to show many more pieces of data like locale, milliseconds elapsed from the start of the application, data about the mobile carrier and WiFi status, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/bugsense9.png"><img title="Report details on bugsense" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/bugsense9-700x95.png" alt="Report details on bugsense" width="700" height="95" /></a>

Personally, I am a bit concerned about the UDID, which has been deprecated since iOS5.

To mark a crash as resolved, return to the application dashboard, select the crash in question, then press the button labeled <em>Resolve Selected</em>, as below:

<img title="resolve_open_bug.png" src="http://www.raywenderlich.com/wp-content/uploads/2013/05/resolve_open_bug.png" alt="Resolve Open Bug" width="600" height="91" border="0" />

You can also provide the number of the release that includes the fix by clicking the <em>Settings</em> link in the navigation bar then selecting <em>Fix Notifications</em>. You can even personalize the message to be sent via push notification like so:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/bugsense11.png"><img title="Setting notifications on bugsense" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/bugsense11-700x471.png" alt="Setting notifications on bugsense" width="700" height="471" /></a>

Bugsense also supports breadcrumbs, but they are only accessible in higher paid plans.
<h2>Bugsense — Summing it Up</h2>
This ends the how-to portion of Bugsense. The platform is quite feature-rich and supports Windows8, Windows Phone and HTML5.

A desktop application to automate the upload of dSYMs would definitely be appreciated, as well as the automatic symbolication of crash logs when they are received on the server.
<h1>TestFlight</h1>
TestFlight was born as a tool to manage the distribution of beta releases. Over time the developers added many more features, like action logging and crash reporting.

TestFlight has been adopted by companies such as Adobe, Instagram and tumblr to manage over-the-air deployment, tracking and crash reporting. iOS is the only platform supported at the moment.
<h2>TestFlight — Configuring the Project</h2>
As mentioned in the previous part of this article, TestFlight is a bit more than a crash reporting system, for it also allows you to recruit testers and distribute your test builds. Once you have logged in, head to <a href="https://testflightapp.com/sdk/download/">the SDK download page</a> to download the SDK. The current version at the time of this writing is 1.2.4.

Open a new copy of the starter project, unzip the TestFlight SDK, drag the entire folder onto the root of the project, and make sure “Copy items into destination folder (if needed)” is checked.

In the target settings, select the <em>Build Phases</em> tab and then open the <em>Link Binary With Libraries</em> section. <em>libTestFlight.a</em> should appear in the list of linked libraries. Also click the “+” button and add the <em>libz.dylib</em> as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf2.png"><img title="Framework linking in TestFlight" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf2.png" alt="Framework linking in TestFlight" width="641" height="151" /></a>

Select the <em>Build Settings</em> tab and select “NO” for the following flags:
<ul>
	<li>Deployment Postprocessing</li>
	<li>Strip Debug Symbols During Copy</li>
	<li>Strip Linked Product</li>
</ul>
Compile the application using Cmd+B to check if everything is ok. Next you’ll need a particular token to make your app work with TestFlight. To get this token, go to your <a href="https://testflightapp.com/dashboard/team/edit/" target="_blank">TestFlight team dashboard</a> and copy the team token displayed. That’s the key you’ll need to initialize the framework in the code.

Open <em>SMAppDelegate.m</em> and add the following import:
<div>
<table>
<tbody>
<tr id="p3405012">
<td id="p34050code12">
<pre>#import "TestFlight.h"</pre>
</td>
</tr>
</tbody>
</table>
</div>
Insert the following code at the beginning of <code>application:didFinishLaunchingWithOptions:</code> in <em>SMAppDelegate.m</em>:
<blockquote>
<div>[TestFlight takeOff:@"YOUR TOKEN HERE"]; ///&lt; Replacing "YOUR TOKEN HERE" with the token you found on your team dashboard</div></blockquote>
Now you’re ready to test the application against the first bug. I suggest you download and install the companion desktop application before doing this, which you can download from <a href="https://testflightapp.com/desktop/" target="_blank">here</a>. This app is smart enough to detect when you have archived an application and offers to upload both the .ipa file and the dSYM. This will save you some pointing and clicking on the TestFlight website.

If you’ve never set up a binary for adhoc distribution, TestFlight has provided some <a href="http://help.testflightapp.com/customer/portal/articles/829857" target="_blank">detailed instructions</a> to get you started.

If you use the desktop application (which I highly suggest you do!), follow the wizard’s instructions and at the end you’ll be prompted to copy out the share URL. That’s the URL of the new build you’ve just submitted — you can send this link to your testers.

To install the app to your device, simply open that URL on your device. It’s that easy! :]. Run the app, crash it by swiping and hitting “Delete”, and restart the app. Now check to see if your crash report shows up on the web back-end by heading to <a href="https://testflightapp.com/dashboard/builds/">the TestFlight dashboard</a>. If all went as expected, you should see something similar to the following:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf3.png"><img title="A build on TestFlight" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf3-700x65.png" alt="A build on TestFlight" width="700" height="65" /></a>
<div>

<em>Note:</em> Crash logs sometimes take a while before showing up on the web interface. TestFlight support indicated that these issues will be resolved in the next release of the SDK.

</div>
This view already provides some interesting data about your build – the number of crashes, feedback and total installations. In your case, you should see one install and one crash. Click the <em>Latest build</em> link to get the detailed report, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf4.png"><img title="Activity view on TestFlight" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf4-700x316.png" alt="Activity view on TestFlight" width="700" height="316" /></a>

A menu on the left gives you an overview of the data related to the build, such as the number of session and the total number of crashes. In the middle section you can see the list of users and on the far right, a summary of the recent activity performed on the application by users.

Tap on the <em>Crashes</em> tab on the left and expand the log using the little arrow on the right. You’ll see the complete stack trace, as below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf.png"><img title="A detailed crash log on TestFlight" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf-700x332.png" alt="A detailed crash log on TestFlight" width="700" height="332" /></a>

Another interesting feature in TestFlight is the ability to add checkpoints. Checkpoints are much like breadcrumbs or events – you spread some log statements in key places in your code to get a better idea of what happened before a crash.

To place a checkpoint in your code, use the method <code>passCheckpoint</code>.

Open <em>SMViewController.m</em>, import the TestFlight framework:
<div>
<table>
<tbody>
<tr id="p3405014">
<td id="p34050code14">
<pre>#import "TestFlight.h"</pre>
</td>
</tr>
</tbody>
</table>
</div>
Now insert the following code at the end of <code>viewDidLoad</code>:
<div></div>
<blockquote>[TestFlight passCheckpoint:[<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/">NSString</a> stringWithFormat:@"view loaded with %i pizzas", pizzaOrder.count]];</blockquote>
<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf6.png">
<img title="Checkpoint on TestFlight" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf6-700x332.png" alt="Checkpoint on TestFlight" width="700" height="332" /></a>Create a new build and distribute it. Install it on your device, run it, scroll down the table view to make it crash, and restart the app to allow the sending of logs to the server. Now open the dashboard, select the new build and click the <em>Checkpoints</em> tab on the left to see the list of checkpoints. You’ll see something similar to the screenshot below:

If you click the <em>Crashes</em> tab, the second bug report will appear:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf7.png"><img title="The second bug on TestFlight" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf7-700x76.png" alt="The second bug on TestFlight" width="700" height="76" /></a>

TestFlight offers a feature which is especially nice during beta testing – collecting feedback from your users directly in your app! Open <em>SMViewController.m</em>, and the following import statement to the top of the file:
<div>
<table>
<tbody>
<tr id="p3405016">
<td id="p34050code16">
<pre>#import "TestFlight.h"</pre>
</td>
</tr>
</tbody>
</table>
</div>
Now modify <em>tableView:commitEditingStyle:forRowAtIndexPath:</em> as follows:
<blockquote>
<div>- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSIndexPath_Class/">NSIndexPath</a> *)indexPath</div>
<div>{</div>
<div>[TestFlight openFeedbackView];</div>
<div>}</div></blockquote>
Create a new build and upload it. Open the app, swipe, and tap to delete. Once the delete button is tapped, a view controller like the following will appear:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf8.png"><img title="Feedback view on TestFlight" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf8.png" alt="Feedback view on TestFlight" width="320" height="568" /></a>

From here the user can send you direct feedback about the current build. You can see the messages sent by users in the “Feedback” section of your build, as so:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf9.png"><img title="Feedback on TestFlight" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/tf9-700x227.png" alt="Feedback on TestFlight" width="700" height="227" /></a>
<h2>TestFlight — Summing it Up</h2>
TestFlight is a complete service that covers distribution, crash reporting and remote logging. However, I have noticed that sometimes the dSYM is not uploaded successfully by the desktop application and you’ll have to re-upload it using the website.
<h1>HockeyApp</h1>
HockeyApp is pretty well known in the indie developer world — probably because it’s made by indie developers! :]

HockeyApp supports iOS, Android, MacOS, and Windows Phone. HockeyApp, like TestFlight, is more than a crash reporting tool. It also allows you to manage the distribution of builds to beta testers, as well as providing a platform to collect feedback.
<h2>HockeyApp — Configuring the Project</h2>
Once you have logged into the site, you will be greeted by an empty dashboard like so:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h1.png"><img title="HockeyApp empty dashboard" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h1-700x219.png" alt="HockeyApp empty dashboard" width="700" height="219" /></a>

You can create a new app using the web site, but using the Desktop application is far easier. Here’s the steps to create your new app with HockeyApp:
<ol>
	<li>Create an API token.</li>
	<li>Download and install the desktop app.</li>
	<li>Configure your project with the API token.</li>
	<li>Configure your Xcode project.</li>
	<li>Archive the project.</li>
</ol>
First, click your account name on the top right and select “API Tokens”, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h2.png"><img title="Create API token on HockeyApp" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h2.png" alt="Create API token on HockeyApp" width="236" height="305" /></a>

Next, you can set the access privileges to the platform in the dialog presented below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h3.png"><img title="API token settings on HockeyApp" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h3.png" alt="API token settings on HockeyApp" width="302" height="306" /></a>

Leave the settings at their defaults (All Apps and Full Access) so that the API key will give you what is effectively “root access” to the platform. Click “Create” and the API token will appear. Copy it and save it somewhere.

Unfortunately there is no direct download link for the desktop application, but the link should be available in the “Installation” section on this <a href="http://support.hockeyapp.net/kb/how-tos-faq/how-to-upload-to-hockeyapp-on-a-mac" target="_blank">knowledge base page</a>.

Run the app and paste the API token in the Preferences pane, as shown here:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h4.png"><img title="Configuring the HockeyApp desktop application" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h4.png" alt="Configuring the HockeyApp desktop application" width="517" height="277" /></a>

How cool would it be if after every archive was built in Xcode, the HockeyApp desktop app opened automatically, ready to upload the new build? Well, you can do that!

Copy your unmodified starter project and open it in Xcode. Select the root of the project, then select <em>Product/Scheme/Edit Scheme</em> (or CMD+&lt;), expand the <em>Archive</em> action, and finally select <em>Post-actions</em>, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h5.png"><img title="Adding a custom action to the archive phase" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h5-700x397.png" alt="Adding a custom action to the archive phase" width="700" height="397" /></a>

Click the <em>+</em> button on the bottom left, select <em>New Run Script Action</em> and paste the following command into the field shown below:
<div>
<table>
<tbody>
<tr id="p3405018">
<td id="p34050code18">
<pre>open -a HockeyApp "${ARCHIVE_PATH}"</pre>
</td>
</tr>
</tbody>
</table>
</div>
<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h6.png"><img title="New action after archive" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h6.png" alt="New action after archive" width="692" height="469" /></a>

Click OK to save. Now archive your current project. The desktop application should detect the archive and show the following dialog:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h7.png"><img title="The desktop app of HockeyApp" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h7.png" alt="The desktop app of HockeyApp" width="507" height="496" /></a>

As you can see, the desktop application has automatically detected the applicationID, version number and has located both .ipa and .dSYM files to be uploaded on the server. Make sure “Download Allowed” is checked, and click <em>Upload</em> to send it to the server.

Open the dashboard and click on the build you just uploaded, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h8.png"><img title="The new build on HockeApp" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h8-700x316.png" alt="The new build on HockeApp" width="700" height="316" /></a>

Notice that users and devices have been automatically detected. Neat! Copy the “App ID” and save it somewhere. You’ll need it to configure the crash reporting system. Now the application is ready to be distributed. You’ll find the download phase to be quite streamlined as well.

Head to <a href="http://config.hockeyapp.net/" target="_blank">http://config.hockeyapp.net/</a> on your iPhone and tap the “Install” link. This will ask you to install the HockeyApp profile and register your device. It will also install a webclip that will appear as an application icon on your device. Once that’s complete, you’ll be able to see the list of applications available for your device, as below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h9.png"><img title="Build distribution on the iPhone" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h9.png" alt="Build distribution on the iPhone" width="320" height="568" /></a>

Tap your app to install it and check that it runs correctly.

So far you have been dealing just with the distribution part. It’s time to integrate crash reporting. Head to <a href="http://hockeyapp.net/releases/">http://hockeyapp.net/releases/</a> to download the client SDK for iOS (the current version as of this writing is 3.0). Download the binary version and unzip it. Drag and drop the folder <em>HockeySDK.embeddedframework</em> onto your Xcode project and make sure “Copy items into destination group’s folder” is checked.

Select the root in <em>Project Navigator</em>, select <em>Project</em> and in the <em>Info</em> tab set <em>Configurations</em> to “HockeySDK” as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h10.png"><img title="Project configuration for HockeApp" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h10-700x229.png" alt="Project configuration for HockeApp" width="700" height="229" /></a>

Open <em>SMAppDelegate.h</em> and add the following import to the top of the file:
<div>
<table>
<tbody>
<tr id="p3405019">
<td id="p34050code19">
<pre>#import &lt;HockeySDK/HockeySDK.h&gt;</pre>
</td>
</tr>
</tbody>
</table>
</div>
You now need to implement the HockeyApp protocols in the delegate. Open <em>SMAppDelegate.h</em> and modify the delegate interface as follows:
<blockquote>
<div>@interface SMAppDelegate : UIResponder &lt;UIApplicationDelegate,BITHockeyManagerDelegate, BITUpdateManagerDelegate, BITCrashManagerDelegate&gt;</div>
<div>...</div>
<div>@end</div></blockquote>
Next, open <em>SMAppDelegate.m</em> and add the following code to the beginning of <code>application:didFinishLaunchingWithOptions:</code>, using the App ID you saved earlier:
<blockquote>
<div>[[BITHockeyManager sharedHockeyManager] configureWithIdentifier:@"YOUR APP ID" delegate:self];</div>
<div>[[BITHockeyManager sharedHockeyManager] startManager];</div></blockquote>
Add the following method to <em>SMAppDelegate.m</em>:
<blockquote>
<pre>- (<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/">NSString</a> *)customDeviceIdentifierForUpdateManager:(BITUpdateManager *)updateManager {
#ifndef CONFIGURATION_AppStore
  if ([[UIDevice currentDevice] respondsToSelector:@selector(uniqueIdentifier)])
    return [[UIDevice currentDevice] performSelector:@selector(uniqueIdentifier)];
#endif
  return nil;
}</pre>
</blockquote>
<div></div>
All of this code will allow you to collect data about the user installations only when the build is <i>not</i> targeting the App Store; that is, when you are in the beta testing phase. To distinguish your new app version from the old version, update the app version in <em>APPNAME-Info.plist</em>. Now you are ready to test your first crash report! First, archive your application and use the desktop application to upload it to the server.
<div>

<em>Note:</em> If you have already uploaded a version of the app to HockeyApp, you will need to increase the version number of the app. To do so, click on the root project. Select “crashy” underneath “Targets”, click the “Summary” tab, then increase your version number, as so:

<img title="increase_version_number.png" src="http://www.raywenderlich.com/wp-content/uploads/2013/05/increase_version_number.png" alt="Increase Version Number" width="600" height="252" border="0" />

</div>
On your device, delete the previous version of your pizza app, and open the web clip that was previously installed to your device. Alternately, you can visit <a href="https://rink.hockeyapp.net/apps">https://rink.hockeyapp.net/apps</a> to do the same thing. There you will see the new release.

Install the new app version, run it, make it crash by deleting a row, and restart the app to allow the crash reports to be uploaded to the sever. After the restart, the application will show an alert view to ask if you want to send a crash report. To do so, tap <em>Send Report</em>. Now visit the HockeyApp dashboard and select your build. It will appear like the following:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h11.png"><img title="The new build on HockeyApp" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h11-700x232.png" alt="The new build on HockeyApp" width="700" height="232" /></a>

There are two versions of the app, as expected, and the newest version shows a crash. Click the row corresponding to the newest version and click the <em>Crashes</em> tab at the top, as shown below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h12.png"><img title="The first crash log on HockeyApp" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h12-700x182.png" alt="The first crash log on HockeyApp" width="700" height="182" /></a>

As you can see the crash report has already been symbolicated and shows the line responsible for the crash. Click it and it will show the details of the stack trace, as below:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h13.png"><img title="Details of the first bug on HockeyApp" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h13-e1362390880938.png" alt="Details of the first bug on HockeyApp" width="699" height="737" /></a>

Once you have fixed the bug, you can close it using the status drop-down at the top left. For the next crash, you will implement some logging. HockeyApp does not include a built-in logger so you will need to add one. You will use the well-known <a href="https://github.com/robbiehanson/CocoaLumberjack" target="_blank">CocoaLumberjack</a>. Download the zip file from GitHub, unzip it, and drag and drop the folder named “Lumberjack” onto the project root.

Now open <em>SMAppDelegate.m</em> and add the following import statements at the top:
<div>
<table>
<tbody>
<tr id="p3405023">
<td id="p34050code23">
<pre>#import "DDLog.h"
#import "DDASLLogger.h"
#import "DDTTYLogger.h"
#import "DDFileLogger.h"</pre>
</td>
</tr>
</tbody>
</table>
</div>
Directly below the import statements in <em>SMAppDelegate.m</em>, add the following code:
<div>
<table>
<tbody>
<tr id="p3405024">
<td id="p34050code24">
<pre>@interface SMAppDelegate ()
@property (nonatomic, strong) DDFileLogger *fileLogger;
@end</pre>
</td>
</tr>
</tbody>
</table>
</div>
Still working in <em>SMAppDelegate.m</em>, add the following code to <code>application:didFinishLaunchingWithOptions:</code> just before the initialization code for HockeyApp:
<div>
<table>
<tbody>
<tr id="p3405025">
<td id="p34050code25">
<pre>_fileLogger = [[DDFileLogger alloc] init];
_fileLogger.maximumFileSize = (1024 * 64);
_fileLogger.logFileManager.maximumNumberOfLogFiles = 1;
[_fileLogger rollLogFile];
[DDLog addLogger:_fileLogger];

[DDLog addLogger:[DDASLLogger sharedInstance]];
[DDLog addLogger:[DDTTYLogger sharedInstance]];</pre>
</td>
</tr>
</tbody>
</table>
</div>
Now add the following two methods to <em>SMAppDelegate.m</em>:
<blockquote>
<pre>- (<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/">NSString</a> *) getLogFilesContentWithMaxSize:(NSInteger)maxSize {
  <a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSMutableString_Class/">NSMutableString</a> *description = [<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSMutableString_Class/">NSMutableString</a> string];

  <a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSArray_Class/">NSArray</a> *sortedLogFileInfos = [[_fileLogger logFileManager] sortedLogFileInfos];
  NSUInteger count = [sortedLogFileInfos count];

  for (NSInteger index = count - 1; index &gt;= 0; index--) {
    DDLogFileInfo *logFileInfo = [sortedLogFileInfos objectAtIndex:index];

    <a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSData_Class/">NSData</a> *logData = [[<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSFileManager_Class/">NSFileManager</a> defaultManager] contentsAtPath:[logFileInfo filePath]];
    if ([logData length] &gt; 0) {
      <a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/">NSString</a> *result = [[<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/">NSString</a> alloc] initWithBytes:[logData bytes]
                                                  length:[logData length]
                                                encoding: NSUTF8StringEncoding];

      [description appendString:result];
    }
  }

  if ([description length] &gt; maxSize) {
    description = (<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSMutableString_Class/">NSMutableString</a> *)[description substringWithRange:NSMakeRange([description length]-maxSize-1, maxSize)];
  }

  return description;
}

- (<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/">NSString</a> *)applicationLogForCrashManager:(BITCrashManager *)crashManager {
    <a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/">NSString</a> *description = [self getLogFilesContentWithMaxSize:5000];
    if ([description length] == 0) {
        return nil;
    } else {
        return description;
    }
}</pre>
</blockquote>
<div></div>
The first method retrieves logged contents from a local file, while the second method implements a delegate method for the crash manager of HockeyApp.

The application is now set up to collect logs locally using the Lumberjack framework, and to send them to the server via the HockeyApp framework.

Open <em>SMViewController.m</em> and add the following import:
<div>
<table>
<tbody>
<tr id="p3405027">
<td id="p34050code27">
<pre>#import "DDLog.h"</pre>
</td>
</tr>
</tbody>
</table>
</div>
In <em>SMViewController.m</em> add the following statement to the end of <code>viewDidLoad</code>:
<div>
<table>
<tbody>
<tr id="p3405028">
<td id="p34050code28">
<pre>DDLogVerbose(@"SMViewController did load");</pre>
</td>
</tr>
</tbody>
</table>
</div>
This small logging statement will tell us if the view loaded successfully or not. But as it stands, you won’t actually be able to see that log line because the log level has not been set. Add the following code directly after the <code>#import</code> statements in <em>SMViewController.m</em>.
<div>
<table>
<tbody>
<tr id="p3405029">
<td id="p34050code29">
<pre>static const int ddLogLevel = LOG_LEVEL_VERBOSE;</pre>
</td>
</tr>
</tbody>
</table>
</div>
I generally set the logging level to verbose, since I prefer to have as many details as possible when debugging.

Finally, modify <code>tableView:willDisplayCell:forRowAtIndexPath:</code> to reflect the code below:
<blockquote>
<pre>- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSIndexPath_Class/">NSIndexPath</a> *)indexPath {
    if (indexPath.row == pizzaOrder.count-1) {
        DDLogError(@"willDisplayCell - pizzas are %i", pizzaOrder.count);
        [[<a href="http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSNotificationCenter_Class/">NSNotificationCenter</a> defaultCenter] postNotificationName:LOAD_MORE_NOTIFICATION
                                                            object:nil];
    }
}</pre>
</blockquote>
<div></div>
This will log the number of pizzas before the notification is posted.
<div>

<em>Note:</em> <code>DDLogError</code> is the only macro that works synchronously, so you can call it even right before a crash. All the other methods such as <code>DDLogVerbose</code> and <code>DDLogInfo</code> are asynchronous, so they might not do their job if they are called immediately before a crash.

</div>
Now update the application version, archive it and let the desktop application upload it to the server. Delete the old version on the device and install the new one using the web clip.
<div>

<em>Note:</em> HockeyApp will detect older versions of a build and will prompt you to update the build from within the app itself. This happens behind the scenes once you upload a new version, so users will not have to manually visit a web address. They just need to launch the app. Pretty convenient, eh?

</div>
Run the application, scroll to the bottom to make it crash, and restart the app to send the crash report. Now visit the dashboard of your app and you will see the crash reported as such:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h14.png"><img title="The second crash log on HockeyApp" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h14-700x197.png" alt="The second crash log on HockeyApp" width="700" height="197" /></a>

Return to the HockeyApp dashboard. Click on the crash to get to the detailed view, then click <em>Crash Logs</em> in the tabs above. This will show the raw log. You should see three tabs below the crashes. Click the <em>Description</em> tab and lo and behold, there are your log statements:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/03/h15.png"><img title="Log statements on HockeyApp" src="http://www.raywenderlich.com/wp-content/uploads/2013/03/h15-700x319.png" alt="Log statements on HockeyApp" width="700" height="319" /></a>

Sometimes applications crash during startup. That makes it impossible to send reports to the server, because the application literally has no time to detect a previous crash and send it over-the-air. HockeyApp is the only tool that allows you to delay the initialization phase (and thus delaying the crash) and gives precedence to the “send to server” procedure so that your crash reports can be uploaded.

You can find some more details on handling startup crashes with HockeyApp <a href="http://support.hockeyapp.net/kb/how-tos-faq/how-to-handle-crashes-during-startup-on-ios" target="_blank">here</a>.
<h2>HockeyApp — Summing it Up</h2>
HockeyApp is a great tool to manage distribution, remote logging and crash reporting. A disadvantage of the service is that it does not include built-in logging. I really appreciated the automation of hooking up with the archive action of Xcode, which saves you some time and stress when you create a new build.

Sometimes the information you need is a bit hidden, such as remote logs, and requires quite a number of clicks to be displayed. I have already expressed my preference for Crashlytics if you need just crash reporting and logging, but I’d definitely consider HockeyApp if you need also distribution management.
<h2>Conclusion</h2>
For your benefit, I’ll repeat the comparison table included in the first part of the tutorial:

<a href="http://www.raywenderlich.com/wp-content/uploads/2013/05/table.png"><img title="comparison table of crash reporting tools" src="http://www.raywenderlich.com/wp-content/uploads/2013/05/table.png" alt="comparison table of crash reporting tools" width="700" height="600" /></a>

I’ll confirm the verdict that I expressed in the first tutorial: Crashlytics is an excellent solution if you need to be up and running quickly and you like a well done back-end.

If you need more than just crash reports and remote logging, and if you are targeting platforms other than iOS, HockeyApp and Bugsense will work really well for you.

&nbsp;
