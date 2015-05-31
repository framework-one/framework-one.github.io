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

* The Clojure integration works best on **Lucee** (4.5 or later) or **Railo** (4.2). ColdFusion 11 is supported too but interop between CFML and Clojure
can get pretty ugly (see the [ColdFusion-specific examples](https://github.com/framework-one/cfmljure/blob/master/index-acf.cfm)
in the **cfmljure** project repo for more details). I doubt it will run on ColdFusion 10 or earlier. I developed exclusively
on Railo from 2009 to 2015 and now I develop on Lucee. Thank [Andrew Myers](https://github.com/am2605) for the ColdFusion 11
support!
* You should be running **Java 8** at this point. The Clojure integration _will not work on Java 6_. It has not been tested on Java 7.
The Java 6 EOL notice was February 2011 and public updates stopped in February 2011. The Java 7 EOL notice was March 2014 and public
updates stopped in April 2015.
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

BTW, **Leiningen** supports unit testing out of the box, so in addition to creating an application skeleton for `myapp`,
it also generates a test skeleton which you can run like this:

    lein test

Because the test skeleton doesn't have any valid tests -- just one deliberate failure -- you should see:

    lein test myapp.core-test
    
    lein test :only myapp.core-test/a-test
    
    FAIL in (a-test) (core_test.clj:7)
    FIXME, I fail.
    expected: (= 0 1)
      actual: (not (= 0 1))
    
    Ran 1 tests containing 1 assertions.
    1 failures, 0 errors.
    Tests failed.

When you start developing services and controllers in Clojure, you'll find it handy to write unit tests as you go and run
them with `lein test`!

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

If you look in the `6helloclojure` folder, you'll see a combination of things you expect to see in a FW/1 application and
files and folders that would see in your `myapp` Clojure test project above:

    Application.cfc index.cfm views
    .gitignore LICENSE README.md doc resources target
    project.clj src test

The files shown in the first line are for FW/1. You might later add a `controllers` folder and a `model` folder if you 
write any of those pieces in CFML.

The files shown in the second line are generated by **Leiningen** and you can pretty much ignore them. You'll see that by
default a Leiningen-generated Clojure project is assumed to live under version control with `git`, have an open source
software license (Eclipse Public License 1.0 by default), have a readme file that explains what the project is for and
how to use it, a documentation folder, possibly some non-source code resource files (configuration files or perhaps
assets for a web application), and you may see a `target` folder which is used by **Leiningen** for compiling code and
generating JAR files etc.

The files shown in the third line are the important parts of the Clojure code:
* `project.clj` defines the dependencies of
your project (the libraries it needs), as well as any development or test tooling (as plugins) and several other
important aspects of how to run, test, and package your code. You'll notice that it also has a description, a URL for
where to find the project (e.g., on GitHub), and details of the license.
* `src` is where your Clojure source code lives. Clojure itself places very few restrictions on the structure of this
(beyond assuming that the file structure matches the namespace structure within the Clojure source files, much the
same way Java's file structure matches its package structure). If you have a Clojure namespace with `-` in its name,
the corresponding folder will have `_` in its name.
* `test` is where your Clojure test code lives. It's a common convention that if your source file is `foo/bar/baz.clj`
which will have a namespace of `foo.bar.baz`, then the tests for that code will live in `foo/bar/baz_test.clj` with
a namespace of `foo.bar.baz-test`. We have `hello/controllers/main_test.clj` which contains tests for the source code
in `hello/controllers/main.clj`.

If you run `lein test` in the `6helloclojure` folder, you'll see:

    lein test hello.controllers.main-test
    
    lein test hello.services.greeter-test
    
    Ran 2 tests containing 3 assertions.
    0 failures, 0 errors.

We'll take a look at that in a minute.

Let's start with the CFML files and look at `Application.cfc` first, then the `views` folder. As usual `index.cfm` is an empty file.

### Application.cfc

There are a few items of note here, and the first is that we specify `diComponent` as `"framework.ioclj"` to tell FW/1 that we want to use `ioclj.cfc`
rather than the default `ioc.cfc` for the Dependency Injection component (the bean factory). `ioclj.cfc` extends DI/1 and
provides the Clojure-specific magic.

Next we specify `diLocations` as the full filesystem path of the current folder. In a CFML / Clojure application, you need to tell the bean factory
about two things: where to find your `project.clj` file and where to look for your CFML beans (if any). Locations are specified as
a comma-separated list of file paths and one of them must specify the exact file path to where `project.clj` lives. That will also be
searched (recursively) for CFCs so you can store both your Clojure code and your CFML beans in the same tree structure if you wish, or
you can store them separately and provide both file paths in `diLocations`.

The other code of note is that `setupRequest()` provides a way to trigger reloading the Clojure code from disk (and recompiling it). Normally, in
a FW/1 app, you can specify an application reload and your bean factory is recreated. Because of the way Clojure code is compiled and loaded
into the JVM, reloading your bean factory is not sufficient to force a reload of those parts of the JVM, so you need to do this programmatically
somehow. You will generally pass the string `"all"` into the `ioclj` bean factory's `reload()` function, although this won't reload any
depedent namespaces, just the ones that `ioclj` treats like beans. More on this below.

## views

This is a regular FW/1 views folder, containing a subfolder for each section of the app (just `main` in this case) and a file for each item (there are
three views here). As expected we have a `main.default` view and a `main.error` view which are basic defaults for FW/1 applications. We also have
a `main.stopped` view. We'll see how each of these is used when we look at the `main` controller.

The `default` view references `rc.greeting` which we'll see being set up in the `main.clj` controller below, and it also gets a `greeterService` from
the bean factory and calls a function in that. We'll see where `greeterService` comes from below as well.

## src/hello/controllers

In this folder we have a single Clojure file, `main.clj`. As you might guess from the file path, this is our application controller. Inside you'll
see a namespace declaration (the `ns` expression) and four functions which represent our handlers.

As with CFML controller functions, each function is passed an argument called `rc` which is the request context. Unlike CFML controllers
which might modify elements of the `rc` struct directly,
Clojure controllers return an updated version of the `rc` data structure to the framework.

In the `default` handler, we get the `:name` element of the `rc` and we return `rc` with an additional element called `:greeting`. In Clojure,
`(:foo bar)` is roughly equivalent to CFML's `bar.foo`, and `(:foo bar "baz")` is similar to:

    structKeyExists( bar, "foo" ) ? bar.foo : "baz"

So `default` passes `rc.name` (or `"anonymous"` if `name` isn't present in `rc`) to the `greet/hello` function and then stores the result in
the `greeting` element of `rc`. Note that the `assoc` function (pronounced _assosh_ like the word _associate_) return a new struct with the
key added -- it does not modify the original struct. This seems very strange at first but you'll get used to it and it's very powerful (and
very safe) since `rc` is immutable. We'll look at where `greet/hello` comes from in a minute.

Next we have the `do-redirect` handler. Yes, Clojure functions can have `-` in their names. The standard naming convention in Clojure is _words-like-this_
rather than _wordsLikeThis_ or _WordsLikeThis_. It's a long-standing Lisp tradition. Yes, it's strange but you'll get used to it and you'll
soon find it more readable than CamelCase. Anyway, this handler tells FW/1 to redirect to `main.default` with the specified query string.
Adding a key called `:redirect` to `rc` is the equivalent of calling `variables.fw.redirect()` in a pure CFML FW/1 app. All four arguments
to the `redirect()` function can be specified as keys in the `:redirect` struct (`:append`, `:preserve`, `:queryString`, `:statusCode`).

Then we have the `stop-it` handler. By the way, the URL actions that correspond to `do-redirect` and `stop-it` are `do_redirect` and
`stop_it` respectively, following the filesystem convention of swapping `_` for `-` in Clojure. This handler tells FW/1 it wants to
abort the controller cycle and also set the view to the `main.stopped` action. Adding a key called `:abort` with a value of `:controller`
is equivalent to calling `variables.fw.abortController()`. Adding a key called `:view` is equivalent to calling
`variables.fw.setView()`. You'll note that `assoc` can take any number of key/value pairs and add them all into the given struct.

Finally we have the `json` handler. By this point, it won't surprise you to learn that this tells FW/1 to render the specified data
as JSON. The Clojure struct `{:a 1 :b "two" :c [3 4 5]}` is equivalent to the CFML struct `{a : 1, b : "two", c : [3, 4, 5]}`.

## src/hello/services

In this folder we have a single Clojure file, `greeter.clj`. As with the controller convention, the file path tells FW/1 that this is
a service (and it would be autowired into any CFML code that declared `property greeterService;` as a dependency). This can also be
pulled from the bean factory as `"greeterService"`, as seen in the `main.default` view file.

There's a single function `hello` in here that takes a string and returns it wrapped with `"Hello "` and `"!"`.

## test/hello

Finally, we'll look at the tests, first for the `main` controller, then the `greeter` service.

In `controllers/main_test.clj`, we have one test function `default-item-test` which contains two related
tests. The arrow syntax means "take this thing and pass it through these functions" so:

    (-> {} default :greeting)

takes an empty struct and passes it as the first argument to the `default` function (our handler method being tested) and then 
pass the result to the `:greeting` function -- remember that `(:greeting my-struct)` is like `my_struct.greeting` in CFML. So
this tests that if the `rc` is empty and you run `main.default` you get a new key called `:greeting` whose value is `"Hello anonymous!"`.

The second test checks that if `rc` contains a `:name` element, you get back the appropriate greeting based on the value of that.

The `(testing "label" ...)` expression can have multiple `is` tests in it, even tho' ours do not.

In `services/greeter_test.clj`, we again have one test function `hello-test` that verifies the `hello` function does what we expect.

## FW/1 Conventions with Clojure

The naming and file path conventions for how you structure your Clojure code for use with FW/1 should be familiar to you if you've
already used DI/1 in the past: plural folder names, containing "components", become beans named for the component, with a suffix that
is the singular of the folder name:

* `src/hello/controllers/main.clj` -> `mainController`
* `src/hello/services/greeter.clj` -> `greeterService`

FW/1 requires that there are at least three segments in a name for this convention (so just `src/controllers/main.clj` would not
match the convention, but `src/hello/admin/controllers/user.clj` would match and become `userController`). An additional restriction
is that the filename + suffix must be unique across your whole application (within the Clojure code (so also having
`src/hello/public/controllers/user.clj` would conflict with `src/hello/admin/controllers/user.clj`).

You can have additional Clojure code that doesn't follow this convention, but the bean factory `reload()` function only attempts to
reload Clojure files that it "knows" about via this convention. You can explicitly reload others -- by passing their namespace to
`reloadClojure=` in the URL, instead of just `all` -- but there are some subtleties there which are beyond the scope of this
documentation (if you want to learn more, read the [clojure.core/require docstring](http://clojure.github.io/clojure/clojure.core-api.html#clojure.core/require)
and know that `reloadClojure=all` does `(require ... :reload)` on each namespace covered by the convention but
`(require ... :reload-all)` for an explicitly provided namespace).

## What is ns all about?

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

# Digging Into Reloading

