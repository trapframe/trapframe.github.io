---
layout: post
title: Windows kernel debugging / WDK, the beginning...
date: 2013-11-12 22:28:15.000000000 +01:00
---


If you want to get in the driver development area and/or kernel debugging there is a few must read.

First know the OS architecture, you don't want to write drivers without this knowledge, and in this area the reference is "Windows internals" :

* Windows Internals part 1 & 2 (ISBN-10: **0735648735** & **0735665877**)


Then there is the "DDK" books , i know these are old ones, but yet, imho, the best one still available:


* Windows NT Device Driver Development (ISBN-10: **0976717522**)

* Developing Windows NT Device Drivers: A Programmer's Handbook (ISBN-10: **0201695901**)
* The Windows 2000 Device Driver Book: A Guide for Programmers (ISBN-10: **0130204315**)

The first one is probably the best book available to introduce device drivers development.

The second one is also very good but more on the hardware side

The last one is about WDM (Windows Driver Model), and should be read last, first master basic driver development then go to WDM, really...

Next, you'll need the [WDK](http://msdn.microsoft.com/en-us/windows/hardware/hh852365.aspx) , the new name for "DDK", it contains everything you need to develop & debug drivers, including a **VERY** good documentation and numerous samples, and it's free :)

I recommend to install the WDK 7.1.0 as this is the last one available to build drivers for XP/2003 if you need too and still being to compile and deploy for newer Windows versions.
Anyway you can install several WDK side by side if needed.

However i would not advise you to start driver development using the new KMDF framework and Visual Studio, they will abstract to much things and you'll end up using yet another framework without the knowledge of the plumbing...

You wan to get really serious in Windows driver development ?
Those guys, [OSR](http://www.osronline.com/), might be the best teachers available, they even teach the newcomers in the Microsoft kernel team :)
Don't read me wrong, there is some incredible smart people in the MS team, but knowing and teaching is a different thing...

They also offer some cool tools and a free subscription to their famous "NT Insider", a free magazine about windows driver development and debugging with some must read articles.





