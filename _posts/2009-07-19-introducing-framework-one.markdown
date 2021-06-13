---
layout: post
title: "Introducing Framework One"
date: 2009-07-19 19:26:21 -0700
comments: true
categories: [releases, fw1]
author: Sean Corfield
---
The first question on everyone's mind is probably: why on earth would you create and publish yet another MVC application framework?<!-- more -->

I've become increasingly dissatisfied with the application frameworks out there. They're big, bloated, complex and, with one or two exceptions, they require XML configuration in some way or other. A discussion on the recently created [ColdFusion OO mailing list](http://groups.google.com/group/coldfusionoo) (started in the wake of the verbal sparring match between Brian Kotek and Hal Helms), saw me admit this... ColdFusion's simplicity was being buried in the complexity of application frameworks and there was no simple on-ramp. Even Fusebox, the mainstay of ColdFusion application frameworks has become far too much for simple applications.

The second question is probably: why did you choose that name?

Years ago, I worked with a programming language called PL/1 (Programming Language One) which was intended to be the "one language to rule them all", so to speak. I wanted to create a very small, simple framework. Something that involved only one core file if possible. Once I'd built it, I needed a name for my one file framework... FW/1 seemed like a good pun.

The third question is going to be: why should I care and why should I use it?

If you're happy with an existing framework, keep using and feel free to ignore FW/1. If you haven't picked a framework yet because they all seem so complex, maybe FW/1 is for you. If, like me, you're tired of framework bloat, try FW/1 and give me your feedback.

I'll be posting some examples as blog posts as well as building out the project wiki over the coming weeks. I built FW/1 in a few hours. It's a single CFC. It's 400 lines of code - including comments. It's entirely convention based. It supports controllers (CFCs), services (CFCs), views (.cfm) and layouts (.cfm). URL/form actions take the form section.item and FW/1 will look for section.cfc (controller and service) and section/item.cfm (view and layouts). It optionally supports your favorite bean factory (ColdSpring, Lightwire and any other IoC container that implements getBean() and containsBean() methods). If you use a bean factory, it will autowire your controllers and services - or your controllers and/or services can be managed by the bean factory.
