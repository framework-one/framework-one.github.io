
### Summary

The [4.3 release](/documentation/4.3) is very a minor maintenance release over 4.2.

### Enhancements

* [514](https://github.com/framework-one/fw1/issues/514) Add optional third flag `ignoreMissing` (defaulted to false) to `injectProperties()` method to allow ignoring of undefined properties in the target bean

### Bug Fixes

* [518](https://github.com/framework-one/fw1/pull/518) aop1 cannot intercept if base name in dottedPath is same as another bean
* [513](https://github.com/framework-one/fw1/pull/513) Remove locking from framework one global controller retrieval
* [522](https://github.com/framework-one/fw1/issues/522) PreserveKey is not passed when using fw.redirect with Lucee Modern local scope mode

### Other Changes

* [516](https://github.com/framework-one/fw1/pull/516) Add adobe 2018 and specify distribution to test runners