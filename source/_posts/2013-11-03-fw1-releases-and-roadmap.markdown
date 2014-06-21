---
layout: post
title: "FW/1 Releases and Roadmap"
date: 2013-11-03 12:38:30 -0700
comments: true
categories: [releases, roadmap]
---
Today I released the first [FW/1 Version 2.2 Release Candidate](https://github.com/framework-one/fw1/releases/tag/v2.2-rc1) for testing. See below for the list of changes in this release, compared to Version 2.1. I also released a maintenance update for the old compatibility branch: [FW/1 Version 1.3](https://github.com/framework-one/fw1/releases/tag/v1.3) (the "Latest Release" label is due to Github's view of the world, but it is the latest 1.x release). This is the version to use if you're on CFMX 7, CF 8, CF 9.0.0, Railo versions prior to 3.2, or OpenBD.<!-- more -->

In addition to these two releases, I also added milestones (and dates!) for the next two FW/1 releases: 2.5 and 3.0. There are some fairly big changes coming, including some important breaking changes. I'll post a follow-up with more details on **why** these changes are coming, but you should read the [FW/1 Roadmap](https://github.com/framework-one/fw1/issues/milestones) and start thinking about how these changes will affect you (as well as reading the next blog post, of course!).

As always, FW/1 wouldn't be possible without all the contributions from the community. This version includes contributions from John Berquist (regex caching in routes, resource packs in routes), Marco Betschart, Chris Blackwell, Peter Boughton, Will Coleda, Billy Cravens, Mark Drew, and all those who have provided feedback via the mailing list and Twitter and IM and... Thank you!

Here's a fairly complete change list for Version 2.2:

* \#198 `renderData()` added for easier REST APIs
* \#197 resource pack support added to routes for easier REST APIs
* \#195 override `processRoutes()` to customize them (`addRoute()` is deprecated)
* \#192 regex in routes are now cached for improved performance
* \#181 improve null value support (Railo compatibility)
* \#179 can use `onMissingMethod()` for `injectFramework()`
* \#176 `buildURL()` now correctly supports `?` at end of action
* \#175 subsystems automatically enabled if you specify subsystem configuration
* \#170 layout suppression is now correctly reset on an exception
* \#169 ensure `framework.home` is consistent with subsystem settings
* \#168 don't strip additional characters from start of path
* \#165 `onMissingView()` correctly called when view is missing
* \#163 trace code no longer fails if session scope is disabled
* \#160 `buildURL()` sanity checks `queryString` as struct

Additional changes:

* fixed error message for service() missing action
* framework initialization logic is more robust
* improve example that uses DI/1 to avoid confusion over what to manage
* improve consistency of framework injection (of itself)
* layout trace shows correct path
