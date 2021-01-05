---
layout: post
title: "Where can I download old versions of FW/1?"
date: 2013-05-07 14:18:12 -0700
comments: true
categories: [downloads, fw1]
author: Sean Corfield
---
I've been asked this several times recently so I figured it was worth a blog post. First of, why would anyone want older versions of the framework? Well, if they're running on Adobe ColdFusion 9.0.0 or earlier, they can't use the 2.x release stream: they're stuck on 1.x. Also, if they're currently using an old version and don't want a major upgrade, they might want a minor upgrade for a bug fix.<!-- more -->

Okay, so why haven't I blogged about this before? Truth be told, I thought it was "obvious" how to find specific legacy releases on any Github project. Apparently, it is not obvious for everyone so it is worth blogging about. Every properly managed project on Github tags every official release so that all past releases can be found on the 'tags' page. You can see [FW/1's 'tags' page](https://github.com/framework-one/fw1/tags) where you can find every release since 1.0. Unfortunately, my choice of naming for tags has not always been consistent and I forgot the 'v' prefix for a while around the release of 2.0. Oops. Unfortunately the typical naming convention for prereleases tends to sort them above their gold release versions - see [Clojure's core.logic library's tags](https://github.com/clojure/core.logic/tags) for a more striking example. At least Github provides an easy mechanism for provided tagged archive releases.

It's probably worth pointing out that downloading FW/1 directly from the [FW/1 GitHub project page](https://github.com/framework-one/fw1) will give you the latest stable release which is currently 2.1.1. That's because it downloads a ZIP file of the "master" branch from the Github site. All development is performed on the "develop" branch.
