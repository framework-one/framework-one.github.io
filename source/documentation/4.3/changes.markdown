---
layout: page
title: "Change Log for FW/1 and Friends"
date: 2020-12-21 08:00
comments: false
sharing: false
footer: true
---
The following changes are part of FW/1 4.3.

Summary
---
The 4.3 release is very a minor maintenance release over 4.2.
 
Enhancements
---

* [514](https://github.com/framework-one/fw1/issues/514) Add optional third flag to `injectProperties()` method to allow ignoring of undefined properties in the target bean

Bug Fixes
---

* [518](https://github.com/framework-one/fw1/pull/518) aop1 cannot intercept if base name in dottedPath is same as another bean
* [513](https://github.com/framework-one/fw1/pull/513) Remove locking from framework one global controller retrieval

Other Changes
---

* [516](https://github.com/framework-one/fw1/pull/516) Add adobe 2018 and specify distribution to test runners