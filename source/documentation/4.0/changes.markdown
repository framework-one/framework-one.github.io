---
layout: page
title: "Change Log for FW/1 and Friends"
date: 2015-12-22 21:20
comments: false
sharing: false
footer: true
---
_This is documentation for the upcoming 4.0 release. For the current release, see [this documentation](/documentation/)._

The following changes are part of FW/1 4.0, DI/1 1.2.0, and cfmljure 1.1.0.

Summary
---
The focus of the 4.0 release is on improving REST support. Improvements include:

* JSON-encoded POST body support.
* Controllers have easy access to HTTP headers.
* Builder syntax for `renderData()` result elements.
* Support for user-supplied rendering functions.
* Integrated support for HTTP `OPTIONS` verb.
* Per-resource error handling.
* Setting status text (in addition to status code) in HTTP responses.
* Wildcard HTTP method support.

In addition, DI/1 has had a number of enhancements.

Breaking Changes
---

* [400](https://github.com/framework-one/fw1/issues/400) - By default, `property` declarations that contain a `type` or `default` are now ignored for autorwiring. In earlier versions of FW/1 (DI/1), such `property` declarations would have been treated as dependencies and autowired: you could override that behavior for _typed_ properties by specifying `omitTypedProperties : true` in your configuration. That is now the default behavior. In addition a new `omitDefaultedProperties` setting has been added, also defaulted to `true`, which is what tells FW/1 (DI/1) to ignore `property` declarations that contain a `default`. If you need to restore the pre-4.0 behavior, add `omitDefaultedProperties` and/or `omitTypedProperties` to your configuration, set to `false`.
* [391](https://github.com/framework-one/fw1/issues/391) - Adobe ColdFusion 9.0.2 is no longer a supported platform. FW/1 4.0 relies on closure support and therefore requires Adobe ColdFusion 10 or later. Support for Railo and Lucee has not changed.
* [390](https://github.com/framework-one/fw1/issues/390) - If you override `missingBean()` in DI/1, read the documentation carefully as this is now an official extension point with different behavior to previous releases.

Enhancements
---

* [414](https://github.com/framework-one/fw1/issues/414) - Add support for Boot to cfmljure.
* [411](https://github.com/framework-one/fw1/issues/411) - Add `headers` argument to controllers.
* [413](https://github.com/framework-one/fw1/issues/413) - `layout()` may now be called from controllers.
* [409](https://github.com/framework-one/fw1/issues/409) - Dependency injection uses additional caches to improve `getBean()` performance (by a factor of 9x-25x, depending on your usage and your CFML engine).
* [407](https://github.com/framework-one/fw1/pull/407) - DI/1 now has a public `hasParent()` predicate method.
* [400](https://github.com/framework-one/fw1/issues/400) - Dependency injection ignores typed/defaulted properties by default. This can be disabled via the `omitDefaultedProperties` and `omitTypedProperties` settings.
* [399](https://github.com/framework-one/fw1/issues/399) - `getBean()` now accepts an optional second argument that can override beans in the factory to provide constructor arguments to be used in the bean's `init()` call.
* [394](https://github.com/framework-one/fw1/issues/394) - Improved error messages when DI/1 attempts to use a CFC that has syntax errors to include filename/line number of the underlying error.
* [392](https://github.com/framework-one/fw1/issues/392) - A wildcard resource match is generated for `$RESOURCES` to provide per-resource error handling. This can be disabled via the `perResourceError` setting.
* [390](https://github.com/framework-one/fw1/issues/390) - DI/1 now considers `missingBean()` to be a fully supported extension point that allows users to handling `getBean()` calls for unknown beans by any means, including creating and returning their own beans.
* [389](https://github.com/framework-one/fw1/issues/389) - A new setting `enableJSONPOST` tells FW/1 to deserialize the JSON-encoded body of a POST.
* [388](https://github.com/framework-one/fw1/issues/388) - The `statusCode` and `jsonpCallback` arguments to `renderData()` have been deprecated and a new builder syntax has been added to support all possible parameters available when rendering data, e.g., `renderData( "json" ).data( result ).statusCode( 202 )`.
* [387](https://github.com/framework-one/fw1/issues/387) - A new setting `preflightOptions` tells FW/1 to provide built-in support for HTTP `OPTIONS`. An additional setting `optionsAccessControl` allows you to fine tune the `Access-Control-*` headers returned.
* [386](https://github.com/framework-one/fw1/issues/386) - Routes can now have `$*` as an explicit wildcard for the HTTP method.
* [385](https://github.com/framework-one/fw1/issues/385) - The new `renderData()` build syntax supports `statusText()` to set the HTTP response status text.
* [328](https://github.com/framework-one/fw1/issues/328) - The `renderData()` `type` may be a function/closure that returns `contentType`, rendered `content`, and an optional `writer` for delivering the data to the browser.

Bug Fixes
---

* [395](https://github.com/framework-one/fw1/pull/395) - Corrected calls to `buildURL()` in examples.
