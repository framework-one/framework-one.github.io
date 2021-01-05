---
layout: post
title: "Open Source: RTW or Collaboration?"
date: 2010-08-28 13:21:49 -0700
comments: true
categories: [di1, fw1, oss]
author: Sean Corfield
---
RTW - Reinventing The Wheel - is normally a bad thing. I'm said several times that the main reason we have no truly great open source software in ColdFusion is because when we see an open source project that doesn't do what we need, we go off and build our version, which we may or may not open source, instead of collaborating on the existing project to make it better.<!-- more -->

[Henry Ho](http://twitter.com/henrylearn2rock) asked on Twitter for recommendations about dependency injection frameworks and quoted a list of several candidates. I replied that DI/1 would soon be added to that list and that led [Peter J. Farrell](http://twitter.com/maestrofjp) to ask "Aren't u reinventing the wheel? We must stop propagating if you don't like something, just create ur own. Contribute instead."

Well, yeah, that's something I've been saying for ages and I've contributed to just about every major CFML framework out there: Fusebox, Mach-II, Model-Glue, ColdBox, Reactor, Transfer, ColdSpring. I've been lead developer on two of those and I contributed enough to ColdSpring to be one of the five copyright holders (along with Dave Ross, Chris Scott, Kurt Wiersma and Peter J. Farrell). With the others I've contributed a range of patches. Because these are all frameworks I've used and it's made sense to contribute, to enhance the projects and to feed those patches back into the project.

So why am I RTW with DI/1? I replied to Peter that I felt DI/1 was addressing different needs to ColdSpring. Peter pushed for details and I said DI/1 was convention-based - no XML. Peter asked "couldn't that be accomplished in CS by just enhancing that project?". It's a reasonable question within the context of ColdSpring but my answer is no, DI/1's design goals are very focused and are intended to offer a very simple, single CFC that relies purely on conventions to provide a lightweight way to manage and autowire your CFCs using conventions instead of configuration. It's what led me to create FW/1 last year, initially as an experiment to see whether I could create my 'ideal' simple MVC framework. FW/1 has developed an active community since then, well over 4,000 downloads, around 300 members on the mailing list, lots of production sites powered by FW/1 and, most importantly for me, a team of people committing changes to both the code and the documentation. FW/1 had similar goals to DI/1 and I'd call FW/1 a success. I hope that DI/1 has even a fraction of that success.

So, yes, RTW is normally a bad thing. Your first choice should always be to contribute to existing projects. Help with testing, help with documentation - those are both key to successful open source projects. If you can fix bugs by providing high quality, well-tested patches, you might become a committer and maybe even a core member of a project. Perhaps you might even end up taking it over - as happened with Mach-II for me early on and later for Matt Woodward and Peter. Model-Glue has transitioned from Joe Rinehart to a team under Dan Wilson's guidance. ColdSpring has transitioned from Chris Scott and his team of contributors to Mark Mandel.

Sometimes tho', something that looks like RTW can be a good thing. If we never RTW'd, we'd all still be using Fusebox, or Mach-II, or Model-Glue, or ... In the ORM space, we started with ARF (Joe Rinehart) and objectBreeze (Nic Tunney) and then Reactor came along (originally Doug Hughes, now Mark Drew) and then Mark Mandel created Transfer. RTW has been a good thing there. Even in the DI space, we've had a number of object factories along the way and ColdSpring became the 800lb gorilla, so to speak, but Lightwire also got some traction. Mark Mandel is currently undertaking a ground-up rewrite of ColdSpring so he can incorporate a number of good things based on Java's Spring framework. In some ways, he is also RTW.

I think Peter will also be writing a blog post around this topic. His tweets covered some other issues, such as whether moving ColdSpring to git (on SourceForge) is good for the community, and I know he has some very strong opinions on open source software! I wish ColdSpring was on github instead of SourceForge but I view the move to git as a good thing. Github has really helped FW/1 get more collaboration from the community. I expect DI/1 will see similar collaboration. It's also why Railo has published its source code on github (although snapshots of builds will still be committed to the JBoss community SVN repo).

I look forward to Peter's blog post on this topic. As I've said in my Open Source Landscape talk, the CFML community is still pretty new to open source software development. We've come a long way in the last five years... but we've got a **long** way to go to be on a par with many other communities!

 
