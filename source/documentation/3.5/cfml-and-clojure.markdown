---
layout: page
title: "Using Clojure with CFML"
date: 2015-05-30 16:30
comments: false
sharing: false
footer: true
---
_This is the upcoming (3.5 - clojure) documentation - for the current stable release, read the [3.0 master documentation](/documentation/3.0/)._

# Clojure and CFML Sitting in a tree

Back in 2010, I started to learn Clojure. It's a Lisp that runs on the JVM. It's a mostly pure functional programming language.
It's very simple -- very little syntax -- but very powerful and expressive. It has a thriving community and ecosystem. Being a
functional programming language it favors immutable data structures and higher order functions (map, filter, reduce, etc) which
makes programs easy to reason about and it also makes concurrent programming pretty easy. No more thread safety issues!

And because I was still writing a lot of CFML back then, I figured out an easy way to allow you, as a CFML developer, to load
and run Clojure code within your CFML application. That functionality has been available via the standalone
[cfmljure library](https://github.com/framework-one/cfmljure) for several years (since September 2010), and it has become core
to how the Internet dating platform works at my company ([World Singles Networks](http://worldsinglesnetworks.com)). We've had
**cfmljure** in production since Spring 2011 and, in 2014, we declared Clojure to be our primary language and nearly all new
development happens in Clojure, hosted within three ColdBox applications and two FW/1 applications. Clojure powers all of our
standalone processes as well.

As we've come to depend on FW/1 more and more, I've wanted to streamline the integration between FW/1 and **cfmljure** so that
we can write services in Clojure and have them autowired into our controllers, just like our CFML services are, as well as have
the option to write controllers in Clojure (where all the code they call is Clojure, not CFML).

That's why, in FW/1 3.5, I've made **cfmljure** part of the standard distribution and provided an extenion to DI/1 to allow
Clojure code to be discovered and autowired into your CFML code, as well as a way to write FW/1 controllers in pure Clojure.

## Some Caveats

Before you get started, there are a couple of things you need to be aware of:

* The Clojure integration works best on Lucee or Railo. ColdFusion 11 is supported too but interop between CFML and Clojure
can get pretty ugly (see the [ColdFusion-specific examples](https://github.com/framework-one/cfmljure/blob/master/index-acf.cfm)
in the **cfmljure** project repo for more details). I doubt it will run on ColdFusion 10 or earlier. I developed exclusively
on Railo from 2009 to 2015 and now I develop on Lucee. Thank [Andrew Myers](https://github.com/am2605) for the ColdFusion 11
support!
* Clojure developers are pretty comfortable at the command line, so the build tool you'll need to install for this is a command
line tool: be prepared to work in Terminal / Console quite a bit!
* Clojure developers use editors that auto-close parentheses, brackets, and braces. Most of those editors also support "structural
editing" which means they understand expressions enclosed in matched pairs of parentheses (or brackets or braces) and let you
easily manipulate _expressions_ rather than just text. There are plugins for Eclipse, Sublime Text, IntelliJ and several others.
Most Clojure developers use Emacs or Vim. I use Emacs for all my editing, CFML included.
* In order to use Clojure with CFML, you **must not run your CFML server under the root account** (on Mac or Linux). It's something
you should never do in production anyway for security reasons, and there's really no reason to do it on your development machine
either. **cfmljure** checks this for you and refuses to initialize (and throws a helpful exception).
* For ease of debugging any problems that may occur, I _strongly_ recommend you always keep a Terminal / Console window open
containing a tail of the console output from your CFML server. If you're using Lucee or a Railo installation based on Tomcat,
you'll want to find the `catalina.out` log file and tail that. If you're using ColdFusion 11, even tho' it is notionally based
on Tomcat, it's completely non-standard and you'll want to tail `cfusion/logs/coldfusion-out.log`.
* Finally, **cfmljure** uses a lock file to ensure that a couple of things it does during initialization are single-threaded
across the whole server, no matter how your CFML server and application instances are configured. On Mac and Linux, it
creates (and deletes) `/tmp/cfmljure.lock`. On Windows, it creates (and deletes) `cfmljure.lock` in your `TEMP` folder which
is normally in the `AppData` folder off your local accounts home folder -- check your `TEMP` environment variable for the
exact location. **cfmljure** tries really hard to clean up after itself but if something goes badly wrong during initialization,
it's possible the lock file will be left behind and you will need to manually delete it. **cfmljure** will try its best to
let you know if it needs you to delete the lock file -- but if you're tailing the logs, you'll see messages as the
initialization proceeds.

## Getting Started with Clojure and CFML

Clojure has a standard build tool called [Leiningen](http://leiningen.org). This manages all of your project dependencies
(e.g., automatically downloading and installing any libraries you need) as well as providing a REPL (Read-Eval-Print-Loop) for
interactive development, running your tests, packaging applications into JAR files, deploying them to standard repositories and so on.

**cfmljure** leverages **Leiningen** to figure out the complete classpath that your Clojure code needs -- including any
libraries you use -- so that it can easily load all the JAR files (and Clojure source code) into your CFML server.

### Leiningen

That means the first step to using FW/1 and Clojure together is to install **Leiningen** and make sure it is working.

If you're on Mac or Linux, follow the [Leiningen Install steps](http://leiningen.org/#install) for the `lein` shell script.

If you're on Windows, I recommend using the [Installer for Windows users](http://leiningen-win-installer.djpowell.net/) mentioned
near the end of that **Install** section.

Once you have it installed, open up a new Terminal / Console window and navigate to directory where you can create a project to
test that it is working properly, then do this:

    lein new app myapp

It will probably download a bunch of libraries the first time you run it, but it should (eventually) respond:

    Generating a project called myapp based on the 'app' template.

Now go into that project and run it:

    cd myapp
    lein run

It should say:

    Hello, World!

Congratulations! **Leiningen** is installed and working!

### Testing FW/1 3.5 and Clojure

At this point you should be able to start your CFML server (remember: not under the **root** account!) and, with FW/1 3.5
installed (and possibly with a `/framework` mapping set up, or with the `framework` folder moved into your webroot), you
should be able to run all of the FW/1 examples. Make sure examples 1-5 work, then try `6helloclojure`.

If it doesn't seem to run correctly, and there's no obvious exception information in the browser or useful information in the
log file, you can try to re-run the `6helloclojure` example with `?cfmljure=abortOnFailure` added to the URL. This should
dump a whole bunch of information to the browser as well as any exception encountered.

If that doesn't help, add `diConfig : { debug : true }` to the `variables.framework` configuration in `Application.cfc` for
that example and try again. This will output more information to the CFML server's console log (which you're tailing, right?).

If you can't figure out the problem from all this extra debugging information, at least you'll have it all available
when you post to the FW/1 mailing list, asking for help!

At this point, however, I hope you got the example to run and you were able to try out the various links and see what it was
doing in the trace output in the browser, and perhaps by looking at the code, which we'll go over next.

## The 6helloclojure Example Explained

# Building Your Own CFML / Clojure Application

## Creating the Clojure project

## Adding the initial CFML files

## Writing a Clojure Service

## Writing a Clojure Controller

## Accessing a Database from Clojure

# A Clojure Primer

## Some Basic Clojure Functions

* Explain `def` and `defn`
* Some useful core functions
* Namespaces, Require, Import
* That `project.clj` file

## About Functional Programming

* Immutable Data Structures
* Functions as Building Blocks

## All You Know About OO Programming is Wrong

* State
* Encapsulation
* Inheritance
* Polymorphism

## More Stuff to Read

* the tutorial for http://www.tryclj.com
* the https://www.4clojure.com puzzles
* the Clojure Koans http://clojurekoans.com
* some great books to read:
  * Clojure Programming http://www.clojurebook.com followed by
  * The Joy Of Clojure http://www.joyofclojure.com :)
