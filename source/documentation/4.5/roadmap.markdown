---
layout: page
title: "FW/1 Roadmap"
date: 2016-01-05 23:00
comments: false
sharing: false
footer: true
---
_This is documentation for the upcoming 4.0 release. For the current release, see [this documentation](/documentation/)._

Whilst you can read the [FW/1 issues list](https://github.com/framework-one/fw1/issues) to see what's on the cards for future releases, several people have asked that I document things at a higher level so they can get a better sense of what's coming.

5.x Lifecycle Restructuring
---
FW/1 5.0 will move the request lifecycle methods out of `Application.cfc` into a separate CFC. In addition, subsystems will also get a CFC that controls their lifecycle methods (`subsystem.cfc`). Since this will be a breaking change, there will be a 4.5 release to pave the way with deprecation warnings and exceptions, in the same way that the 2.5 release paved the way between 2.2 and 3.0.

4.x REST and Modernization
---
FW/1 4.0 is planned as improving REST support and Dependency Injection, and will be a maintenance release for 3.5. The major version number change indicates that support for ACF9.0.2 will be dropped to allow for the use of closures in the code as this will provide some nicer syntax for some framework features. The core code will be modernized to take advantage of features in ACF10+ and Lucee 4.5+ in this release.

3.x Integrations
---
FW/1 3.5 is the current stable release, coming soon after the 3.1 release. It adds integraton of Clojure controllers and autowired Clojure services. It does this by bundling cfmljure and providing an extension to DI/1, as well as a controller adapter. It also provides direct support for the Lucee language (a new dialect of CFML, introduced in Lucee 5.0), as well as a new, improved way to add subsystems to an existing application. In addition, WireBox is better support as a DI option, and the folder naming conventions can be configured for the first time. It is a major release but also remains compatible with earlier 3.x releases.

FW/1 3.1 is the previous stable release, and it was primarily a maintenance release for 3.0, but it also bundled AOP/1 for the first time. It builds on core 3.0 features:

* Using DI/1 to manage services and beans
* Explicitly calling services in the `item()` controller methods

FW/1 3.0 bundles DI/1 and automatically creates bean factories following the convention of a `model` folder containing `services` and `beans` subfolders. Subsystems also follow this convention and use the main application's bean factory as a parent to enable sharing of common services across subsystems.

FW/1 3.0 also slims down the framework by removing all of the service queue machinery and the start/end variants of controller methods, as well as coalescing all the framework components in a single folder `framework` (with `org.corfield.framework` replaced by `framework.one`).

2.5 Deprecations
---
FW/1 2.5 represented a large shift from 2.2.1 and deprecated a number of long-standing features that were removed in 3.0:

* Service queue - implicit service calls were disabled in 2.x but still supported for backward compatibility with 1.x, but in 2.5 the whole service queue machinery is deprecated, including the `service()` call, and will be removed in 3.0.
* `startItem()` / `endItem()` controller methods - these were originally introduced to encourage a clean separation of controller logic from (queued) service logic. With the service queue going away, they make no sense, so they are deprecated in 2.5 and will be removed in 3.0.
* Global access to the `rc` variable - this was never really intended to be supported but some users found it convenient and started to rely on it. Due to edge case bugs, and the fact that there are better ways to manage the `rc` in the lifecycle, global access is deprecated in 2.5 and will be removed in 3.0. This will mostly affect users who update the `rc` in `setupRequest()` - logic which should generally be moved to `Application.cfc` in the global `before()` controller method.

2.1/2.2 Release Stream
---
This is the previous stable stream. FW/1 2.5 is the last release in that stream. The 2.x versions support Adobe ColdFusion 9.0.1 and later, Lucee 4.5.0 and later, and Railo 3.2.2 and later.

Changes between 1.x and 2.x include:

* No more implicit service calls
* Custom route support
* Property-based injection in controllers
* Better lifecycle control: layout and view control, abort controllers
* Better REST support: render data instead of views, resource packs and regular expressions in routes
* Environment control
* Application tracing
* Automated test suite

1.x Release Stream
---
This is the legacy stream. FW/1 1.3 is the most recent legacy version, provided to support older CFML engines (Adobe ColdFusion 9.0.0 and earlier, Railo 3.1.x, Open BlueDragon). FW/1 2.x and 3.x will not run on these older CFML engines. Only critical fixes will be made to the 1.x release stream. Currently no new 1.x releases are planned.
