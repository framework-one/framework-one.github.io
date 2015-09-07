---
layout: post
title: "FW/1 Out of the Box"
date: 2015-09-06 23:00
comments: true
categories: [releases, fw1, commandbox]
---
FW/1 is up on [ForgeBox](http://www.coldbox.org/forgebox). ForgeBox is the npm / Maven of the CFML world. If you haven't heard about it -- or you think it's only for "Box" products -- you need to check it out! It's a repository for CFML projects that can be easily installed via [Command Box](https://www.ortussolutions.com/products/commandbox). Wait! You haven't heard of that either? Gosh, you've got some reading to do! It'll change the way you do development!

Go get it installed, then read on!<!--more-->

Now you have `box` installed, here’s how to get up and running with FW/1 easily:

    > box
    CommandBox:mydir> install fw1-commands
    ... you only need to do this once ...
    CommandBox:mydir> mkdir example
    CommandBox:mydir> cd example
    CommandBox:example> fw1 create app example basic
    CommandBox:example> install fw1
    ... installs stable 3.1.2 version ...
    CommandBox:example> start

At this point it’ll open a browser running your new skeleton FW/1 app. Happy coding!

Note that you don't need to have installed a CFML server, or set up a webroot - `box` can start a CFML server in any directory so you can get up and running quickly!

You can check out the various [FW/1 commands](https://github.com/framework-one/fw1-commands) available inside `box` but, like Ruby on Rails, they let you quickly add new controllers, views, subsystems and so on to your application. Huge kudos to Tony Junkes for contributing this to the FW/1 family of projects!

Going forward, my plan is that `install fw1` will always install the current stable (master) version — I just need to remember to keep it up to date! — and you can also install specific versions (currently `fw1-3.1.2` and `fw1-3.5.0`). The latter installs from the develop branch. Support for true multiple version installs is planned for `box`, possibly next month.

One of the things that is really nice about this is you can switch FW/1 versions as easily as:

    CommandBox:example> uninstall fw1
    CommandBox:example> install fw1-3.5.0

Now your app is running 3.5.0 (Alpha 2) instead of the stable release!

