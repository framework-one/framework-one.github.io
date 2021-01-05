---
layout: post
title: "cfmljure updated"
date: 2011-08-11 13:21:49 -0700
comments: true
categories: [releases, clojure]
author: Sean Corfield
---
Even tho' it's far from a 1.0 release, I've updated the [cfmljure](https://github.com/framework-one/cfmljure) project on github to contain the latest version that we're using at World Singles and update the examples so they work again, along with updated instructions - mainly that the Clojure code needs to be in `WEB-INF/classes/` so that it can be picked up dynamically. This is a fundamental piece of our infrastructure at World Singles, since we rely heavily on Clojure for the Model of our application, with our View-Controller in CFML. It's not intended to be general purpose code but if you want to play around with calling Clojure from CFML, it should get you started.

Don't forget the [cfmljure](http://groups.google.com/group/cfmljure) mailing list if you need assistance!
