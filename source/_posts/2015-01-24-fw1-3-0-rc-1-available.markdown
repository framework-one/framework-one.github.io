---
layout: post
title: "FW/1 3.0 RC 1 available"
date: 2015-01-24 23:21:49 -0700
comments: true
categories: [releases, fw1, di1]
---
FW/1 3.0 has been in beta testing since August 2014 and lots of people are already running in production so I figured it was time to push out the first [Release Candidate build](https://github.com/framework-one/fw1/releases/tag/v3.0-rc1).

The main focus of RC 1 has been bug fixes. Only a small number of functional enhancements have been added (notably `getEnvVar()` to retrieve the value of a system environment variable which can be useful during environment control processing).<!-- more --> Some deprecated features have been removed:

* `getRC()` and `getRCValue()` have been removed, along with the configuration flag that had been enabling them. They were a hasty addition at the end of the 2.5 cycle and they were a mistake - if you're using them, you're doing something wrong!
* `org.corfield.framework` is no longer supported - use `framework.one` instead. It was always a questionable choice of file path for the framework and I've been tempted to change it several times. The 3.0 cycle deprecated it and moved the framework CFC to the `/framework` folder, where DI/1 has lived since the 2.5 cycle. This is probably a **breaking change** unless you've been using prerelease builds of 3.0 and have already eliminated the deprecation warnings!

In addition, AOP/1 is no longer bundled with FW/1. It's not ready for primetime yet so I didn't want to include it in the 3.0 release. It got a lot of work between Alpha 1 and Beta 1 but additional bugs and some hard problems came up in testing. It's still available on the `aop1` branch if you want to experiment with it (and help find and fix more bugs in it!).

As part of the preparation for RC 1, all of the documentation has been reviewed and updated and DI/1's documentation is now [part of the main documentation site](http://framework-one.github.io/documentation/using-di-one.html). Big thanks go to Nando Breiter for bringing that across from the old wiki in the standalone DI/1 repo. Code contributors to RC 1 include: John Berquist, Ryan Guill, Cameron Childress. Thank you!

At this point, only bug fixes will be considered before FW/1 3.0 goes "gold" and given the long period of testing so far on Beta 1, that final release shouldn't be too far away.

Oh, and if you go to [FW/1's page on RIAForge](http://fw1.riaforge.org), you'll see that 3.0 RC 1 is the default download now, instead of 2.5.
