---
layout: page
title: "Change Log for FW/1 and Friends"
date: 2016-07-12 10:15
comments: false
sharing: false
footer: true
---
_This is documentation for the upcoming 4.0 release. For the current release, see [this documentation](/documentation/)._

The following changes are part of FW/1 4.0, DI/1 1.2.0, and cfmljure 1.1.0.

Summary
---
The focus of the 4.0 release is on improving REST support. Improvements include:

* JSON-encoded and URL-encoded POST / PUT body support.
* Controllers have easy access to HTTP headers.
* Builder syntax for `renderData()` result elements.
* Support for user-supplied rendering functions.
* Integrated support for HTTP `OPTIONS` verb.
* Per-resource error handling.
* Setting status text (in addition to status code) in HTTP responses.
* Wildcard HTTP method support.

In addition, DI/1 has had a number of enhancements, including the addition of a builder syntax for programmatically declaring beans.

Breaking Changes
---

* [400](https://github.com/framework-one/fw1/issues/400) - By default, `property` declarations that contain a `type` or `default` are now ignored for autorwiring. In earlier versions of FW/1 (DI/1), such `property` declarations would have been treated as dependencies and autowired: you could override that behavior for _typed_ properties by specifying `omitTypedProperties : true` in your configuration. That is now the default behavior. In addition a new `omitDefaultedProperties` setting has been added, also defaulted to `true`, which is what tells FW/1 (DI/1) to ignore `property` declarations that contain a `default`. If you need to restore the pre-4.0 behavior, add `omitDefaultedProperties` and/or `omitTypedProperties` to your configuration, set to `false`.
* [391](https://github.com/framework-one/fw1/issues/391) - Adobe ColdFusion 9.0.2 is no longer a supported platform. FW/1 4.0 relies on closure support and therefore requires Adobe ColdFusion 10 or later. Support for Railo and Lucee has not changed.
* [390](https://github.com/framework-one/fw1/issues/390) - If you override `missingBean()` in DI/1, read the documentation carefully as this is now an official extension point with different behavior to previous releases.

Enhancements
---

* [442](https://github.com/framework-one/fw1/issues/442) - In Alpha 1 and Beta 1, the `decodeRequestBody` setting was called `enableJSONPOST`. As a migration aid, attempting to set `enableJSONPOST` will throw an exception saying you should use `decodeRequestBody` instead.
* [441](https://github.com/framework-one/fw1/pull/441) - `decodeRequestBody` now also handles URL-encoded form variables, which is typical for PUT, making it a poorly named setting but...
* [439](https://github.com/framework-one/fw1/issues/439) - Add `framework.facade` component to make FW/1 accessible out-of-band (for integration purposes).
* [434](https://github.com/framework-one/fw1/issues/434) - Add `getRoutePath()` convenience method.
* [419](https://github.com/framework-one/fw1/issues/419) - Add `getCGIRequestMethod()` convenience method.
* [418](https://github.com/framework-one/fw1/issues/418) - Allow `factoryBean()` to accept function/closure.
* [417](https://github.com/framework-one/fw1/issues/417) - Add builder syntax for bean declarations.
* [416](https://github.com/framework-one/fw1/issues/416) - Delay bean discovery (where possible) until after declarations are processed.
* [415](https://github.com/framework-one/fw1/issues/415) - Add support for CFML-only and Clojure-only search paths.
* [414](https://github.com/framework-one/fw1/issues/414) - Add support for Boot to cfmljure.
* [413](https://github.com/framework-one/fw1/issues/413) - `layout()` may now be called from controllers.
* [412](https://github.com/framework-one/fw1/issues/412) - Add `renderer()` to access `renderData()` builder and add `header()` to set HTTP response headers.
* [411](https://github.com/framework-one/fw1/issues/411) - Add `headers` argument to controllers.
* [410](https://github.com/framework-one/fw1/issues/410) - Clarified license (Apache Source License 2.0), added LICENSE file.
* [409](https://github.com/framework-one/fw1/issues/409) - Dependency injection uses additional caches to improve `getBean()` performance (by a factor of 9x-25x, depending on your usage and your CFML engine).
* [407](https://github.com/framework-one/fw1/pull/407) - DI/1 now has a public `hasParent()` predicate method.
* [400](https://github.com/framework-one/fw1/issues/400) - Dependency injection ignores typed/defaulted properties by default. This can be disabled via the `omitDefaultedProperties` and `omitTypedProperties` settings.
* [399](https://github.com/framework-one/fw1/issues/399) - `getBean()` now accepts an optional second argument that can override beans in the factory to provide constructor arguments to be used in the bean's `init()` call.
* [394](https://github.com/framework-one/fw1/issues/394) - Improved error messages when DI/1 attempts to use a CFC that has syntax errors to include filename/line number of the underlying error.
* [392](https://github.com/framework-one/fw1/issues/392) - A wildcard resource match is generated for `$RESOURCES` to provide per-resource error handling. This can be disabled via the `perResourceError` setting.
* [390](https://github.com/framework-one/fw1/issues/390) - DI/1 now considers `missingBean()` to be a fully supported extension point that allows users to handling `getBean()` calls for unknown beans by any means, including creating and returning their own beans.
* [389](https://github.com/framework-one/fw1/issues/389) - A new setting `decodeRequestBody` tells FW/1 to deserialize the JSON-encoded body of a POST.
* [388](https://github.com/framework-one/fw1/issues/388) - The `statusCode` and `jsonpCallback` arguments to `renderData()` have been deprecated and a new builder syntax has been added to support all possible parameters available when rendering data, e.g., `renderData( "json" ).data( result ).statusCode( 202 )`.
* [387](https://github.com/framework-one/fw1/issues/387) - A new setting `preflightOptions` tells FW/1 to provide built-in support for HTTP `OPTIONS`. An additional setting `optionsAccessControl` allows you to fine tune the `Access-Control-*` headers returned.
* [386](https://github.com/framework-one/fw1/issues/386) - Routes can now have `$*` as an explicit wildcard for the HTTP method.
* [385](https://github.com/framework-one/fw1/issues/385) - The new `renderData()` build syntax supports `statusText()` to set the HTTP response status text.
* [328](https://github.com/framework-one/fw1/issues/328) - The `renderData()` `type` may be a function/closure that returns `contentType`, rendered `content`, and an optional `writer` for delivering the data to the browser.

Bug Fixes
---

* [440](https://github.com/framework-one/fw1/issues/440) - Improved thread safety on application reloading. Even `reloadApplicationOnEveryRequest : true` should be safe now!
* [429](https://github.com/framework-one/fw1/issues/429) - Removed `expandPath()` in calls to `cachedFileExists()`.
* [427](https://github.com/framework-one/fw1/issues/427) - Fixed bug that prevented `before()` / `after()` working in Clojure controllers; fixed bug that caused Clojure controller shims to be created twice.
* [422](https://github.com/framework-one/fw1/pull/422) - Fix bug with case insensitive routes.
* [420](https://github.com/framework-one/fw1/issues/420) - Fix bug in transient autowiring (caused by caching metadata).
* [395](https://github.com/framework-one/fw1/pull/395) - Corrected calls to `buildURL()` in examples.
