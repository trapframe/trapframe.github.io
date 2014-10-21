---
layout: post
title: Thoughts (and questions) on prefetch  and layout.ini
date: 2014-09-10 17:38:14.000000000 +02:00
---
Today, while doing some testing around prefetch entries i decided to have a closer look to the *layout.ini* file in */windows/prefetch/*.
I thought this file was a kind of plaintext version of all the informations contained in the *.pf* files, and was used by the defrag engine to optimize files content location on the disk.

I was wrong.

My mistake was not about its role or content, but about the retention period.

I've done a few tests/checks (non exhaustive) on several machines (XP / Win7 / Win8) and each time the *layout.ini*  was containing more information, and specifically **much more older** traces of binaries execution than the one present in the prefetch files (even older than the entries found in the **Application compatibility Cache**).

I am aware that the maximum number of *.pf* files is quite limited, but it appears that same limitation does not apply to the *layout.ini* entries.

How is the OS managing the entries in this file is still a mistery to me but i suspect that there is a limit to the number of entries contained in *layout.ini*, but it's higher.

For the sake of curiosity i created a script which create and run the same executable with a different name each time (test01.exe, test02.exe, ...).
Effect on the *.pf* entries was expected, most of the previous *.pf* files were deleted and i was left with a prefetch directory full of *testXX.exe* files.

On the other hand, the *layout.ini* file was still containing very old entries in addition to the new *testXX.exe* ones (i ran *"rundll32.exe advapi32.dll,ProcessIdleTasks"* in order to trigger manually the layout.ini update).

I asked some forensic buddies about this, and they were surprised, like me, about these results.

This might be a known forensic artifact i overlooked, but since it looks like it is not THAT well known i though it was worth of a blog post, even if there is still some pending questions...

One thing i learned today is that i will never perform a forensic investigation without having a look to this *layout.ini* file.
