---
layout: post
title: "FW/1 The Year Ahead"
date: 2013-11-02 23:42:41 -0700
comments: true
categories: roadmap
---
With FW/1 Version 2.2 just around the corner - after a long time in incubation - and FW/1 itself being almost four and a half years old, it's a good time to look ahead at what's in store.<!-- more -->

When FW/1 was first conceived, it was intended to be drop-dead simple and help on-board developers who were new to MVC and new to frameworks and also new to OOP. It leveraged conventions very heavily to encourage simple controller logic and delegation to a service layer for heavy lifting. That was the conceptual justification for the implicit service calls in the 1.x version and the ability to split controller methods in two - start/end-item - to wrap the automatic call to the service layer.

While that conceptual framework served its purpose admirably, users very quickly grew out of it and needed to start managing service calls more directly. That's why implicit service calls were no longer the default in 2.0 (although you could turn them back on). Even with that change, many users find the queuing of service calls confusing, even tho' controller calls are also queued (although that's mostly invisible to users).

In version 2.5, scheduled for early January 2014, FW/1 will begin the move away from queuing services by deprecating the `service()` call and requiring a configuration setting to enable its use in your application. In version 3.0, scheduled for release just after cf.Objective() 2014, the `service()` call will be removed. Along with that, the start/end-item calls will be deprecated in 2.5 and removed in 3.0, since they were only introduced in the first place to create the queued services workflow!

This means that users will need to manage services themselves and of course I recommend using a Dependency Injection framework for that (or at least using some sort of object factory as a bare minimum). Accordingly, DI/1 will have a higher profile in FW/1 2.5 and the two frameworks will be officially bundled together in 3.0. FW/1 will continue to support any bean factory that provides the `containsBean()` and `getBean()` API such as ColdSpring (WireBox uses a slightly different API but I plan to provide an adapter for it in 2.5).

Also as part of 3.0, the framework CFC itself will be renamed and the `/org/corfield` structure removed. The default path will be `/framework/one.cfc` so your `Application.cfc` will have `extends="framework.one"` by default. In 2.5, DI/1 will have adopted this pattern as `/framework/ioc.cfc`, but since 2.5 will still be backward compatible with 2.2 (after you've added the compatibility setting in `Application.cfc`), I don't want to force renaming or reorganizing on users until 3.0.

Finally, as part of 3.0, the entire repository will be restructured to better reflect what is considered "best practices" in terms of where you install things and what lives in your webroot (only web-accessible assets!). This will make it easier to get started with the FW/1 skeleton application as a "best practice" out-of-the-box experience.

Note that the DI/1 and AOP/1 repositories will remain active but DI/1 versions will be in lockstep with FW/1 from 3.0 onward, and development will be conducted as part of the FW/1 repository, with releases being merged to the DI/1 repository. Once AOP/1 reaches a similar level of maturity, it will likely follow the same trajectory.
