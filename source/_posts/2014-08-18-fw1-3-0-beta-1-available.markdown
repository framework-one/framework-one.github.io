---
layout: post
title: "FW/1 3.0 Beta 1 available"
date: 2014-08-18 11:21:49 -0700
comments: true
categories: [releases, fw1, di1]
---
Just over four weeks ago, I released [FW/1 3.0 Alpha 1](http://framework-one.github.io/blog/2014/07/20/fw1-3-0-alpha-1-available-for-testing/) and declared it _feature complete_. There were some big changes in that release and, in particular, some long-standing features were removed (after being deprecated in FW/1 2.5) and some recently-introduced features were also deprecated. Today I am releasing the first Beta version which includes bug fixes and usability enhancements, focusing primarily on DI/1 and AOP/1.<!-- more -->

You can [download FW/1 3.0 Beta 1 from Github](https://github.com/framework-one/fw1/releases/tag/v3.0-beta1) and that release page has a link to [the complete list of closed tickets in FW/1 3.0 Beta 1](https://github.com/framework-one/fw1/issues?milestone=13&page=1&state=closed) and [closed tickets in DI/1 that are for FW/1 3.0](https://github.com/framework-one/di1/issues?milestone=1&page=1&state=closed). Note that those issues are only fixed in the FW/1 repository, not the DI/1 repository, but they will be backported later on.

DI/1 and AOP/1 come of age
---

As indicated above, the focus of Beta 1 has been on cleaning up DI/1 and AOP/1 to get them to a "1.0" release status as part of FW/1 3.0. Going forward, the DI/1 and AOP/1 repositories will only get updated with released versions and will be stripped down to minimal examples for those who wish to use them standalone. Future development (including issues and test cases etc) will all be done in the FW/1 repository.

DI/1 has been enhanced in this release to provide some features that will help developers migrating from ColdSpring (or who are looking for some of ColdSpring's more advanced features in DI/1).

AOP/1 has been rewritten to better integrate with DI/1 and ensure that injected beans are intercepted (the `beanProxy.cfc` needed only minor tweaks and it does the heavy lifting of interception).

Here is the full list of changes in DI/1 and AOP/1 (since Alpha 1):

* Dotted path deduction rewritten / improved [di1/#61](https://github.com/framework-one/di1/issues/61). There were a number of situations where DI/1 was unable to figure out a dotted component path for CFCs identified through relative folder paths, especially outside the primary webroot of an application. This should be addressed now!
* New option `omitDirectoryAliases` - default `false` [di1/#64](https://github.com/framework-one/di1/issues/64). Set this `true` if you want to suppress the directory-based aliases that DI/1 creates (e.g., `beans/user.cfc` => `userBean`). This will enforce uniqueness of bean names (since the suffix will no longer differentiate beans with the same name in different folders).
* IIS web server mapping case sensitivity [di1/#65](https://github.com/framework-one/di1/issues/65). A bug fix for an annoying edge case caused by IIS being case sensitive in a situation that caused hard-to-debug errors from DI/1.
* AOP/1 now handles intercepted methods that return null [#264](https://github.com/framework-one/fw1/issues/264). Self-explanatory.
* DI/1 now accepts a load listener as part of its configuration [#273](https://github.com/framework-one/fw1/issues/273). This was added to allow load listeners to be added easily via `diConfig` when using FW/1. A load listener is the recommended way to declare new beans, add aliases and additional beans and set up factory beans/methods (see next item).

The following DI/1 setups are now equivalent:

    var bf1 = new framework.ioc( "..." );
    bf1.onLoad( myListener );
    
    var bf2 = new framework.ioc( "...", { loadListener = myListener } );

* DI/1 now supports factory beans / factory methods [#274](https://github.com/framework-one/fw1/issues/274). The new API `factoryBean()` allows you to specify that a bean should be obtained from another bean - the _factory bean_ - by calling a specified method - the _factory method_ - with optionally specified arguments, and optional bean value overrides (similar to the `declareBean()` API).

Here are some examples:

    bf.factoryBean( "a1Bean", "myFactory", "theMethod" );
    // a1Bean = bf.getBean("myFactory").theMethod();
    bf.factoryBean( "a2Bean", "yourFactory", "createIt", [ "dsn" ] );
    // a2Bean = bf.getBean("yourFactory").createIt( bf.getBean("dsn") );
    bf.factoryBean( "a3Bean", "warehouse", "builder", [ "dsn", "config" ],
        { dsn = "myDB" } );
    // a3Bean = bf.getBean("warehouse").builder( "myDB", bf.getBean("config") );

* DI/1 now supports a post-injection _init-method_ like ColdSpring [#275](https://github.com/framework-one/fw1/issues/275). A new configuration option `initMethod` allows you to specify a no-argument method that DI/1 should attempt to call on all managed beans after all of their dependencies have been injected. This allows beans to perform additional configuration that requires access to their injected dependencies, which cannot be performed in a constructor. You can thank Daniel Budde II for this feature being added (and for spurring me to finally add factory beans/methods which I'd been thinking about for a while)!
* AOP/1 now intercepts injected beans [#277](https://github.com/framework-one/fw1/issues/277). This was the rewrite of AOP/1 to hook into DI/1's `resolveBean()` method via a new `setupInitMethod()` extension point (which will make additional extensions to DI/1 easier). Previously AOP/1 only intercepted beans obtained directly via `getBean()`. Thank you to Daniel Budde II for identifying this issue and providing a test case to help debug and verify the new behavior.
* DI/1 no longer resolves beans multiple times [#279](https://github.com/framework-one/fw1/issues/279). This was a performance issue but previously harmless. With the enhancements to AOP/1, this introduced several bugs (hopefully all fixed now!).

FW/1 bug fixes and enhancements
---

Here are the changes in FW/1 itself since Alpha 1:

* Controllers are reloaded after the bean factory is updated [#276](https://github.com/framework-one/fw1/issues/276). Previously, FW/1 cached controllers and recreating the bean factory was not sufficient to pick up changes in controller files. Now, when you call `setBeanFactory()`, the controller cache is cleared so controllers will be reloaded and any changes will be picked up (regardless of whether controllers are managed by FW/1 or DI/1).

The road to gold
---

The next milestone should be Release Candidate 1 and only bug fixes are likely to be considered at this point, no new features or enhancements, unless they are required to make the Beta feature set fully usable. If all goes well, RC1 should be released in about 3-4 weeks, and the _gold_ 3.0 release about 3-4 weeks after that (late September / early October).

Note that `org.framework.corfield` is a deprecated path for FW/1 - it has moved to `framework.one` - and whilst it is supported during the 3.0 prerelease builds, it will be removed in the gold release. Similarly, as noted in the Alpha 1 blog post, `getRC()` and `getRCValue()` are deprecated and will also be removed in the gold release.
