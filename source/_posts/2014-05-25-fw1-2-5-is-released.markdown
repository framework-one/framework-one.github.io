---
layout: post
title: "FW/1 2.5 is released!"
date: 2014-05-25 23:13:39 -0700
comments: true
categories: releases
---
This is a migration release to pave the way for breaking changes in Release 3.0. All examples have been updated to latest best practices and now use cfcscript exclusively. Examples use DI/1 0.5.0 to manage all beans and services (as `framework.ioc`), and no longer rely on start/end actions or the `service()` method.<!-- more -->

As always, FW/1 can be downloaded from the [FW/1 page on RIAForge](http://fw1.riaforge.org). Release 2.5 is now the latest stable release of this framework, as it approaches its fifth birthday!

For a full list of all tickets closed in Release 2.5: https://github.com/framework-one/fw1/issues?milestone=14&page=1&state=closed

Migration from 2.2.1
---
The `service()` call has been deprecated, as have start/end action items. Global access to `rc` in `Application.cfc` has also been deprecated. If you just drop 2.5 into your setup and you rely on these features, you'll get exceptions explaining how to enable these features for backward compatibility, namely add the following to your framework configuration:

    enableGlobalRC = true,
    suppressServiceQueue = false

The ability to enable the implicit service calls is still present via:

    suppressImplicitService = false

but, like the other two options, defaults to disallowing the deprecated feature.

If you enable these deprecated features, you will no longer get exceptions using them, but you will see deprecation warnings in your application server's console log. This is to remind you to update your code in preparation for 3.0 later this year!

_Please note that Release 3.0 will completely remove these backward compatibility options - and the associated deprecated features. In addition, `org.corfield.framework` will move to `framework.one` in Release 3.0, alongside `framework.ioc`._

