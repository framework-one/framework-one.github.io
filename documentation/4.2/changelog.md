
### Summary

The [4.2 release](/documentation/4.3) is very a minor maintenance release over 4.1.

### Enhancements

* Add `onMissingMethod()` pass-thru support to AOP/1
* [486](https://github.com/framework-one/fw1/issues/486) Experimental ColdBox Module support (work-in-progress)

### Bug Fixes

* [500](https://github.com/framework-one/fw1/pull/500) Fix unscoped variables `q` and `a` in `buildURL()`
* [496](https://github.com/framework-one/fw1/pull/496) Allow AOP/1 to accept array of folders (DI/1 already accepted this)
* [493](https://github.com/framework-one/fw1/pull/493) Use `encodeForURL()` instead of `urlEncodedFormat()`
* [491](https://github.com/framework-one/fw1/pull/491) Fix trailing `&` on query string