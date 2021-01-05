---
layout: post
title: "FW/1 3.0 RC 2 available"
date: 2015-02-06 21:21:49 -0700
comments: true
categories: [releases, fw1, di1]
author: Sean Corfield
---
About two weeks ago I released [FW/1 3.0 RC 1](http://framework-one.github.io/blog/2015/01/24/fw1-3-0-rc-1-available/) and the only real bug persisting at that point was related to [DI/1 dotted path deduction](https://github.com/framework-one/fw1/issues/283).

I think that bug is finally squashed, based on early testing by some users that had encountered the bug, so the second [Release Candidate build](https://github.com/framework-one/fw1/releases/tag/v3.0-rc2) of FW/1 3.0 is now available!<!-- more -->

In addition to fixing that DI/1 bug, the only other change since RC 1 is some cleanup of the `examples` folder. That means this RC is almost certainly going to be gold candidate for the final 3.0 release.

Please download and test this version and report any issues you find. I'd like to cut the final 3.0 release fairly soon, so I can merge `develop` to `master` and start planning FW/1 3.5.
