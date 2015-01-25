---
layout: post
title: "FW/1 Version 2.1 Released"
date: 2013-01-05 14:23:53 -0700
comments: true
categories: [releases, fw1]
---
FW/1 2.1 is now the official master release.

The develop branch has been merged to master and the 'v2.1' tag has been added. Work on version 2.2 will soon start on the develop branch.<!-- more -->

Version 2.1 has been a long time in the works. Version 2.0 was released in December 2011, just over a year ago. So what has been going on in Framework One Land since then? You can play along at home by reading [the commit list for the master branch](https://github.com/seancorfield/fw1/commits/master)...

January 2012 saw version 2.0.1 appear with a fix for `buildUrl("")` which was broken just before the 2.0 gold release and caught by Seb Duggan. Thank you. January also saw a patch from Daryl Banttari to improve the performance of `expandPath()` by caching, similar to how `fileExists()` was handled.

February brought the start of FW/1's MXUnit-based test suite with some stellar work by Ryan Anklam. Ryan continued that work in March, June and July - thank you! March brought a pull request from John Whish to improve application startup behavior in the face of errors in user code. In June, Matt Quackenbush contributed a nice improvement to `redirect()`: support for HTTP status codes. In July, Ryan Anklam contributed to `populate()`, providing the ability to handle child objects automatically, and Giancarlo Gomez enhanced the regex support in routes. In August, I fixed a bug in autowiring bean factories when using subsystems which was identified by Marco Betschart, and then Marcin Szczepanski substantially expanded the test suite throughout August, September and October - much appreciated! In October, I fixed a bug in how `setView()` interfered with error handling, identified by Alex Purice, and then I worked on the test suite and I test-drove the development of the new "environment control" feature. Peter Boughton spotted that `setLayout()` did not work inside `onMissingView()` so I test-drove a fix for that. I added the `unhandledExceptionCaught` option so you can choose to rely on FW/1 for handling errors in requests that would otherwise be 'unhandled' by the framework (thanks to 'stubotnik' for that suggestion). Dave Ferguson identified a nasty bug in ColdFusion 10's WebSocket implementation that was causing him problems with FW/1 - which I fixed by copying `CGI` variables to the `request` scope. I cleaned up `request` scope usage and further improved FW/1's robustness in the face of errors at startup, due to user code problems. In November, Richard Herbert found a case where FW/1 failed if `session` scope was disabled (now fixed). December was then a busy month with the addition of the new tracing / debugging facility, ensuring HTTP status code 500 is set when an unhandled exception is reported, and various small bug fixes and code cleanup in the new features added. Thanx there to Nando Breiter, Marcin Szczepanski and Chris Blackwell for finding those bugs!

Thank you everyone for your contributions to FW/1 - I really appreciate the assistance in making the framework better for its community! As can be seen above, it really is a team effort.

In summary then, version 2.1 contains:

* Environment Control support.
* Application tracing.
* `populate()` now handles child objects automatically.
* Improved regex support in routes.
* Addition of a nice test suite, making it easier to test-drive the development of new features, as well as ensure fewer regression bugs sneak in.
* `request` scope cleanup (and removal of several `request` variables that had previously been marked as deprecated.
* Lots of small improvements to the robustness of application startup in the face of user code problems, making it easier to debug those problems, and several enhancements to error handling in general.

You can [download FW/1 2.1 at RIAForge](http://fw1.riaforge.org/).

 
