---
layout: post
title: "FW/1 - The napkin spec"
date: 2010-02-06 19:09:11 -0700
comments: true
categories: [roadmap, fw1]
---
A few folks have asked me to post the "napkin" on which I wrote the spec for FW/1. My "napkin" is actually Evernote because I have it on every computer and my iPhone so it's always with me and it's easy to develop notes with. I started the spec on July 17th and "finished" it on July 20th. The spec was titled "New Lightweight Framework". Here's what it said:<!-- more -->

> Goal: Create an extremely lightweight convention over configuration framework. Considerations:
> 
> * Leverage `Application.cfc` and lifecycle
> * Automatically call controller, model, view if appropriate
> * Autowire from bean factory?
> * `Application.cfc` extends `org.corfield.X`
> * Programmatically set everything, no XML
> * `variables.framework` struct to specify everything
> * `variables.framework.action` is URL / form variable for the, er, action, defaults to `'action'`
> * `variables.framework.home` is home action, defaults to `main.default`
fold URL / form into request.context
> * `?action=section.item` maps to `controllers/section.cfc:item()` then `models/section.cfc:item()` then `views/section/item.cfm`
> * implicit layouts based on actions
> 
> Caveats:
> 
> * Should controller / model be instantiated every request or cached?
> * How should cache be refreshed?

That's it. I wrote the first version of FW/1 on July 19th. You can see the [original 381 line framework.cfc](https://github.com/framework-one/fw1/blob/a686fd441ccd86e147f770f41b10f79a07be11f2/org/corfield/framework.cfc) on Github. So there you go: an insight into my design process!
