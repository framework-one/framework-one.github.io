
### Summary

The [4.1 release](/documentation/4.1) is intended to be a minor maintenance release over 4.0. 
The following changes are part of FW/1 4.1 and DI/1 1.3.0.

### Breaking Changes

* [466](https://github.com/framework-one/fw1/issues/466) - Clojure integration is no longer provided out of the box, so that Lucee 5.x can be officially supported.

### Enhancements

* [481](https://github.com/framework-one/fw1/issues/481) - `customTemplateEngine()` extension point and `makeMethodProxies()` utility function have been added to make it easier to support additional templating engines. An example is provided for `mustache` in which the views and layouts are HTML templates rendered by a Java implementation of the [Mustache template language](http://mustache.github.io).
* [472](https://github.com/framework-one/fw1/issues/472) - Since Adobe ColdFusion 9 is no longer supported, dynamic method invocations are done via `invoke()` rather than `evaluate()` (and so they might be slightly faster).
* [467](https://github.com/framework-one/fw1/pull/467) - Session scope handling is now pluggable (but still uses `session` scope by default).
* [465](https://github.com/framework-one/fw1/issues/465) - The tests have been switched from MXUnit to TestBox and the CI system has been switched from Ant (and explicitly download CFML engines) to CommandBox. The test matrix now includes: Adobe ColdFusion 10, 11, 2016; Lucee 4.5, 5 (current 5.1).
* [460](https://github.com/framework-one/fw1/issues/460) - New framework option `missingview` can specify an action to take on a `FW1.viewNotFound` exception, rather than the default `error` action.
* [452](https://github.com/framework-one/fw1/issues/452) - `baseURL` with trailing `/` no longer causes `//` to appear in URLs (when calling `buildURL()`).
* [424](https://github.com/framework-one/fw1/issues/424) - To partially support the desire to unload bean factory data when FW/1 is reloaded, there is a new extension point `onReload()`.

### Bug Fixes

* [463](https://github.com/framework-one/fw1/issues/463) - A potential XSS vulnerability in the default exception reporting function has been addressed.
* [462](https://github.com/framework-one/fw1/pull/462) - Addresses a race condition around resolving transients under heavy load. Thanks to John Whish and jcberquist for chasing this down!
* [458](https://github.com/framework-one/fw1/issues/458) - If an exception occurs during bean discovery, and an application's error handling causes any DI/1 method to be invoked, it's likely that `discoverBeans()` will be run a second time. Previously, that caused beans to be loaded twice and, if you had `omitDirectoryAliases : true`, you would get a new exception (that bean names were not unique) which masked the original exception. Now, if an exception occurs during initialization, bean discovery is considered complete, which should allow the original exception to propagate (even if your exception handling then crashes and burns!).
* [456](https://github.com/framework-one/fw1/issues/456) - `onError()` now resets `setLayout()` so that if an error occurs in a subsystem, the error handling is rendered correctly, using only layouts from the error action.
* [383](https://github.com/framework-one/fw1/issues/383) - `redirect()` now accepts a `struct` for the `queryString` argument: this was introduced in version 3.5 but never actually worked until now!
