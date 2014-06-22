---
layout: post
title: "FW/1 Release 1.1 Available!"
date: 2010-06-11 19:02:00 -0700
comments: true
categories: releases
---
The latest stable release of Framework One is now available for [download](https://github.com/framework-one/fw1/releases/tag/v1.1) from github! The previous stable release is still available on the tags page ([download FW/1 1.0](https://github.com/framework-one/fw1/releases/tag/v1.0)).<!-- more -->

This release features a large number of enhancements suggested by the community including:

* A skeleton application
* The list of keys in `populate()` can contain spaces for improved readability
* The delimiter for subsystems can be configured (the default is still `:`)
* You can override the behavior when a view is missing by defining your own `onMissingView()` handler
* The name of the application is automatically generated allowing you to omit `this.name` assignment in `Application.cfc`
* Controller calls can be queued up in `setupApplication()`, `setupSession()` and `setupRequest()` via the `controller()` API - `doController()` is officially deprecated
* `populate()` now works with CFCs that use generated setters or `onMissingMethod()` via the new `trustKeys` argument - and `onPopulateError()` to trap any errors that arise
* Sample applications now work on installations with a non-empty context root (except for a couple clearly marked as such)
* Skinning is possible via a new override point: `customizeViewOrLayoutPath()`
* When you queue up `service()` calls, you can now provide a struct of additional arguments so you add arguments that are not in the request context
* The list of extensions and file paths that are ignored by FW/1 is now configurable (so you can easily allow certain parts of your application to operate outside the framework)
* A new API `getConfig()` returns a readonly copy of the framework's configuration structure which may be useful in controllers
* You can now override the view conventions easily in a controller via the new `setView()` API
* The action arguments is now consistently optional in all of the getters for subsystem / section / item combinations
* An example of security / access control with FW/1 has been added as a variant of the user manager
* `buildURL()` and `redirect()` can generate SES URLs via new FW/1 configuration settings: `generateSES` and `SESOmitIndex`
* `buildURL()` and `redirect()` now all embedded query strings to make URL generation easier, as well as allowing control over which name/value pairs are folded into SES URLs vs regular query string format
 
