---
layout: post
title: "FW/1 3.0 Alpha 1 available for testing!"
date: 2014-07-20 18:14:24 -0700
comments: true
categories: releases
---
I consider FW/1 3.0 to be _feature complete_ at this point so I am releasing Alpha 1 for testing. I expect people to run into a few bugs - this release has some big changes in it, compared to the 2.x release stream - and it's possible that new feature requests will crop up during alpha testing, but everything I wanted to change is in place.<!-- more -->

You can [download FW/1 3.0 Alpha 1 from Github](https://github.com/framework-one/fw1/releases/tag/v3.0-alpha1) and that release page has a link to [the complete list of closed tickets in FW/1 3.0 Alpha 1](https://github.com/framework-one/fw1/issues?milestone=13&page=1&state=closed) although I'm going to summarize the most important ones in this blog post.

Features Removed
---

First and foremost, some features that have been part of FW/1 from the early days have been removed. These features were deprecated in FW/1 2.5 as a migration path so I would strongly advise anyone still on FW/1 2.2.1 (or earlier) to upgrade to 2.5 in preparation for the (breaking) changes in 3.0!

These features include the `service()` API call and the `start*()` and `end*()` item handlers within controllers, as well as global references to `rc` (where it was not passed as an argument or made available in a view automatically). You can read more about the deprecation (and now removal) of these features in [the release announcement for FW/1 2.5](http://framework-one.github.io/blog/2014/05/25/fw1-2-5-is-released/) on this blog. Management of services via a bean factory, with property-based injection, and direct invocation has long been considered a much better way to interact with services than using the "service queue" that FW/1 originally provided.

In addition, the recently added `getRC()` and `getRCValue()` API calls - added in FW/1 2.5 during the deprecation of global references to `rc` - have been deprecated _and will be removed in the final FW/1 3.0 release_. They were hastily added and they were unnecessary. In this alpha release, their use will trigger an exception explaining what to use instead. You can add:

    framework.enableLegacyRCAccessors = true

to your configuration while you update your code (this will suppress the exception but still write a message to your application server's console output - just as the deprecation process did in FW/1 2.5).

Automatic Bean Factory Usage
---

The other big change in this 3.0 release is that DI/1 (and AOP/1) is fully integrated. FW/1 itself moves from `org.corfield.framework` to `framework.one` alongside `framework.ioc` (DI/1), `framework.aop` (AOP/1), and some helper CFCs. `org.corfield.framework` still exists but will issue a deprecation warning if it is used. It will be removed in the final 3.0 release.

You can still place the FW/1 CFCs anywhere you want but if you move DI/1, you'll need to tell FW/1 where to find it - see below.

Previously, it was expected that you create a bean factory in your `Application.cfc` `setupApplication()` function and call FW/1's `setBeanFactory()` API to tell the framework about it. For some time, he conventional path to your **Model** CFCs has been `/model/beans` for your transient beans (domain objects) and `/model/services` for your singleton services (and perhaps `/model/gateways` for any data gateways, although those could just as easily live in your services folder too). That means you nearly always had the following code in `setupApplication()`:

    var bf = new framework.ioc( "model" );
    setBeanFactory( bf );

Or, if you also managed your controllers this way, you may have had:

    var bf = new framework.ioc( "model,controllers" );
    bf.addBean( "fw", this );
    setBeanFactory( bf );

The `addBean()` call ensures that the bean factory knows `fw` is an alias for your bean factory so it will be available to any controller `init( any fw )` methods when they are constructed.

If you use subsystems, you probably had something similar in your `setupSubsystem()` function (and hopefully you set the default bean factory as a parent for each subsystem bean factory).

Now, FW/1 does this for you automatically. There are new configuration options to control the details, but the default cases _should just work_ and you can remove your bean factory creation code from your `setupApplication()` function. Those options are:

* `diEngine` - the type of the dependency injection engine: FW/1 knows about **"di1"**, **"aop1"**, and **"wirebox"**. The default is **"di1"**. You can also specify **"none"** to suppress the automatic bean factory machinery and **"custom"** if you want to tell FW/1 to use your own bean factory (see below). Note that ColdSpring is deliberately _not supported_ as it is no longer maintained by anyone and has not been updated in years.
* `diComponent` - the default location of the bean factory CFC. For DI/1, this is `framework.one`; for AOP/1, this is `framework.aop`; and for WireBox, this is `framework.WireBoxAdapter`. If you move these files elsewhere, or setup a different mapping for them, set `diComponent` to that new location. If `diEngine` is **"custom"**, you can set `diComponent` to the dotted path of your bean factory for FW/1 to use it automatically.
* `diLocations` - the set of folders that DI/1, AOP/1, or WireBox will scan for CFCs. The default is **"model,controllers"** - note the relative paths! If you have these folders elsewhere (i.e., not relative to the application root), then you'll need to specify `diLocations`, e.g., as `"/model,/controllers"` or `"/myapp/model,/myapp/controllers"` or something similar.
* `diConfig` - additional configuration passed to DI/1, AOP/1, WireBox, or your custom bean factory. Specifically, this is the second argument to the constructor for DI/1 or AOP/1, and the `properties` argument to the constructor for WireBox, or the single argument to the constructor for your own bean factory. By default, it is an empty struct.

Additional Features
---

In addition to the two major changes listed above, there are a number of minor enhancements compared to FW/1 2.5:

* `isUnhandledRequest( string targetPath )` - a new API that you can override to tell FW/1 not to handle certain requests. By default, this returns **true** for certain file extensions and certain paths, as specified by the `unhandledExtensions` and `unhandledPaths` configuration values but you can choose to override this completely, or still call `super.isUnhandledRequest(targetPath)` and add additional conditions of your own.
* `redirectCustomURL( string uri, string preserve = 'none', statusCode = '302' )` - a new API that uses `buildCustomURL()` to construct a URL for a redirect.
* `buildCustomURL()` - now supports variable substitution: if `:varname` is present in the URI passed in and `rc.varname` exists and is a simple value, then that value will be substituted into the returned URL. To avoid confusion with subsystem paths, `:varname` will only be recognized if it follows one of: `/`, `?`, `=`, `&`.
* `setLayout()` - now accepts an optional second argument, a **boolean**, that let's you tell FW/1 to automatically suppress any further layouts. This removes the need to specify `request.layouts = false` in your layout file.
* Both DI/1 and FW/1 now try very hard to avoid attempting to autowire FW/1 itself (or the Application.cfc based on it, which acts as a global controller in a FW/1 application).
