---
layout: page
title: "Change Log for FW/1 and Friends"
date: 2016-11-27 16:40
comments: false
sharing: false
footer: true
---
_This is documentation for the upcoming 4.1 release. For the current release, see [this documentation](/documentation/)._

The following changes are part of FW/1 4.1, DI/1 1.3.0, and cfmljure 1.2.0.

Summary
---
The 4.1 release is intended to be a minor maintenance release over 4.0.

Breaking Changes
---

* None so far.

Enhancements
---

* [460](https://github.com/framework-one/fw1/issues/460) - New framework option `missingview` can specify an action to take on a `FW1.viewNotFound` exception, rather than the default `error` action.
* [457](https://github.com/framework-one/fw1/issues/457) - In mixed CFML / Clojure applications, dependencies that might break your CFML Servlet container are no longer loaded (e.g., `servlet-api` JARs). You could often get away with doing this, but sometimes you could get errors and they were often very obscure.
* [455](https://github.com/framework-one/fw1/issues/455) - In mixed CFML / Clojure applications, you can now use `framework.one` (from FW/1 for Clojure) to simplify how you render data, redirect, etc. The `cljcontroller.cfc` shim now understands both the original data format (where you just did `(assoc rc :render {:type :json :data expr})`) and the native FW/1 data format (from `(fw1/render-json rc expr)`). This allows controllers to be shared between mixed CFML / Clojure applications and standalone FW/1 for Clojure applications.
* [454](https://github.com/framework-one/fw1/issues/454) - In mixed CFML / Clojure applications, Clojure namespaces that could conflict with CFML beans are suppressed (turn on `debug : true` to see this in the console log). Previously, such Clojure namespaces could supplant CFML beans and the errors produced would be obscure at best.
* [452](https://github.com/framework-one/fw1/issues/452) - `baseURL` with trailing `/` no longer causes `//` to appear in URLs (when calling `buildURL()`).

Bug Fixes
---

* [458](https://github.com/framework-one/fw1/issues/458) - If an exception occurs during bean discovery, and an application's error handling causes any DI/1 method to be invoked, it's likely that `discoverBeans()` will be run a second time. Previously, that caused beans to be loaded twice and, if you had `omitDirectoryAliases : true`, you would get a new exception (that bean names were not unique) which masked the original exception. Now, if an exception occurs during initialization, bean discovery is considered complete, which should allow the original exception to propagate (even if your exception handling then crashes and burns!).
* [456](https://github.com/framework-one/fw1/issues/456) - `onError()` now resets `setLayout()` so that if an error occurs in a subsystem, the error handling is rendered correctly, using only layouts from the error action.
