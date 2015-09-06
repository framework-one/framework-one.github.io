---
layout: page
title: "Using Clojure with CFML"
date: 2015-09-05 14:30
comments: false
sharing: false
footer: true
---
_This is the upcoming (3.5 - develop) documentation - for the current stable release, read the [3.1 master stable documentation](/documentation/3.1/)._

# Clojure and CFML Sitting in a tree
{:.no_toc}

Back in 2010, I started to learn Clojure. It's a version of the Lisp programming language that runs on the JVM. It's a mostly pure functional programming language.
It's very simple -- very little syntax -- but very powerful and expressive. It has a thriving community and ecosystem. Being a
functional programming language it favors immutable data structures and higher order functions (map, filter, reduce, etc) which
makes programs easy to reason about and it also makes concurrent programming pretty easy. No more thread safety issues!

Although Clojure can be compiled to JVM bytecode and deployed as a JAR or WAR file, it can also be treated like
a dynamic scripting language and compiled on-the-fly to JVM bytecode, just like CFML, giving you that very fast edit-reload-test
cycle that you're used to with CFML. In fact, if you use the REPL (Read-Eval-Print-Loop) you can pretty much reduce that to
just edit-test since Clojure code is compiled as you type it in and can be evaluated immediately.

Because I was still writing a lot of CFML back in 2010, I figured out an easy way to allow you, as a CFML developer, to load
and run Clojure code within your CFML application. That functionality has been available via the standalone
[cfmljure library](https://github.com/framework-one/cfmljure) for several years, and it has become core
to how the Internet dating platform works at my company ([World Singles Networks](http://worldsinglesnetworks.com)). We've had
**cfmljure** in production since Spring 2011 and, in 2014, we declared Clojure to be our primary language and nearly all new
development happens in Clojure, hosted within three ColdBox applications and two FW/1 applications. Clojure powers all of our
standalone processes as well.

As we've come to depend on FW/1 more and more, I've wanted to streamline the integration between FW/1 and **cfmljure** so that
we can write services in Clojure and have them autowired into our controllers, just like our CFML services are, as well as have
the option to write controllers in Clojure (if all the code they need to call is Clojure, not some mix of CFML and Clojure).

That's why, in FW/1 3.5, I've made **cfmljure** part of the standard distribution and provided an extension to DI/1 to allow
Clojure code to be discovered and autowired into your CFML code, as well as a way to write FW/1 controllers in pure Clojure.

* Table of Contents
{:toc}

## Some Important Caveats (and System Requirements)

Before you get started, there are a handful of things you need to be aware of:

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
(e.g., automatically downloading and installing any libraries you need) as well as providing a REPL for
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

If you look in the `6helloclojure` folder (in `examples/subsystems/`), you'll see a combination of things you expect to see in a FW/1 application and
files and folders that would see in your `myapp` Clojure test project above:

    Application.cfc MyApplication.cfc index.cfm views
    .gitignore LICENSE README.md doc resources target
    project.clj src test

The files shown in the first line are for FW/1. You might later add a `controllers` folder and a `model` folder if you 
write any of those pieces in CFML.

The files shown in the second line are generated by **Leiningen** and you can pretty much ignore them. You'll see that by
default a Leiningen-generated Clojure project is assumed to live under version control with `git`, have an open source
software license (Eclipse Public License 1.0 by default), have a README (markdown) file that explains what the project is for and
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

This is based on `/framework/Application.cfc` and allows us to use FW/1 without a global mapping. It creates a per-application mapping for
`/framework` via a relative path using `expandPath( '../../framework' )`, and then it creates an instance of `MyApplication.cfc`, passing in the
FW/1 configuration structure.

There are a few items of note here, and the first is that we specify `diComponent` as `"framework.ioclj"` to tell FW/1 that we want to use `ioclj.cfc`
rather than the default `ioc.cfc` for the Dependency Injection component (the bean factory). `ioclj.cfc` extends DI/1 and
provides the Clojure-specific magic. Note that we leave `diEngine` as the default (which is `"di1"`) because `ioclj.cfc` is just an extension to DI/1.

Next we specify `diLocations` as the full filesystem path of the current folder. In a CFML / Clojure application, you need to tell the bean factory
about two things: where to find your `project.clj` file and where to look for your CFML beans (if any). Locations are specified as
either a comma-separated list of file paths, or an array of paths, and one of them must specify the exact directory path to where `project.clj` lives. That will also be
searched (recursively) for CFCs so you can store both your Clojure code and your CFML beans in the same tree structure if you wish, or
you can store them separately and provide both file paths in `diLocations`.

### MyApplication.cfc

This is based on `/framework/MyApplication.cfc` and allows us to override parts of FW/1. It extends `framework.one` (using the per-application
mapping created in `Application.cfc`), and then overrides any FW/1 we want to customize.

The code of note is that `setupRequest()` reloads the Clojure code whenever the framework itself is reloaded.
Normally, in
a FW/1 app, you can specify an application reload and your bean factory is recreated. Because of the way Clojure code is compiled and loaded
into the JVM, reloading your bean factory is not sufficient to force a reload of those parts of the JVM, so you need to do this programmatically
somehow. Note that this will only reload the namespaces that follow the FW/1 conventions to be discovered. See below for more on this.

### views

This is a regular FW/1 views folder, containing a subfolder for each section of the app (just `main` in this case) and a file for each item (there are
three views here). As expected we have a `main.default` view and a `main.error` view which are basic defaults for FW/1 applications. We also have
a `main.stopped` view. We'll see how each of these is used when we look at the `main` controller.

The `default` view references `rc.greeting` which we'll see being set up in the `main.clj` controller below, and it also gets a `greeterService` from
the bean factory and calls a function in that. We'll see where `greeterService` comes from below as well.

### src/hello/controllers

In this folder we have a single Clojure file, `main.clj`. As you might guess from the file path, this is our application controller. Inside you'll
see a namespace declaration (the `ns` expression) and four functions which represent our handlers.

As with CFML controller functions, each function is passed an argument called `rc` which is the request context. Unlike CFML controllers
which might modify elements of the `rc` struct directly,
Clojure controllers return an updated version of the `rc` data structure to the framework.

In the `default` handler, we get the `:name` element of the `rc` and we return `rc` with an additional element called `:greeting`. In Clojure,
`(:foo bar)` is roughly equivalent to CFML's `bar.foo`, and `(:foo bar "baz")` is similar to:

    structKeyExists( bar, "foo" ) ? bar.foo : "baz"

So `default` passes `rc.name` (or `"anonymous"` if `name` isn't present in `rc`) to the `greet/hello` function and then stores the result in
the `greeting` element of `rc`. Note that the `assoc` function (pronounced _assosh_ like the word _associate_) returns a new struct with the
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
The **[Clojure Primer](#a-clojure-primer)** below explains basic symbols and data structures, when you're ready to read more about this.

### src/hello/services

In this folder we have a single Clojure file, `greeter.clj`. As with the controller convention, the file path tells FW/1 that this is
a service (and it would be autowired into any CFML code that declared `property greeterService;` as a dependency). This can also be
pulled from the bean factory as `"greeterService"`, as seen in the `main.default` view file.

There's a single function `hello` in here that takes a string and returns it wrapped with `"Hello "` and `"!"`.

### test/hello

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

In a typical Clojure application, you have `src/` and `test/` folders containing your source code and your test code. Then you have
the top-level name of your application -- `hello` in the paths above -- and then below that you have all your Clojure "components".

Therefore, FW/1 requires that there are at least three "segments" in a file path and name for this convention:

* `src/myapp/controllers/main.clj` has `myapp.controllers.main` as the namespace -- three segments
* `src/hello/admin/services/user.clj` has `hello.admin.services.user` as the namespace -- four segments

That means that just `src/controllers/main.clj` would not match the convention as its namespace would be `controllers.main` (only two
segments -- no top-level application name). An additional restriction for FW/1 to automatically manage injection of Clojure namespaces
into your CFCs is that the filename + suffix must be unique across your whole application within the Clojure code (so also having
`src/hello/public/services/user.clj` would conflict with `src/hello/admin/services/user.clj`).

Aside: You can have additional Clojure code that doesn't follow this convention, but the bean factory `reload()` function only attempts to
reload Clojure files that it "knows" about via this convention. You can explicitly reload others -- if you allow for a URL variable that can
be passed to the `reload()` method of the bean factory, identifying a single namespace --
but there are some subtleties there which are beyond the scope of this
documentation (if you want to learn more, read the [clojure.core/require docstring](http://clojure.github.io/clojure/clojure.core-api.html#clojure.core/require)
and know that `reload("all")` does `(require ... :reload)` on each namespace covered by the convention but `reload("some.namespace")` does
`(require 'some.namespace :reload-all)` for an explicitly provided namespace).

## What is ns all about?

Probably the most complex and confusing aspect when you are first learning Clojure is the `ns` expression. `ns` serves two main purposes:

* It declares the namespace in which the code in a given file lives.
* It declares what additional namespaces you depend on and how you're going to access functions in them.

Think of a namespace as a "package" or "module". The default convention in Clojure is that the namespace matches the filesystem path below your `src`
or `test` folder, except that `-` in a namespace identifier matches `_` in the file path. Technically, a namespace can be implemented across multiple
files but it isn't very common to see that, so don't worry about it.

In the `6helloclojure` example, you'll see `hello.controllers.main` for `src/hello/controllers/main.clj` and `hello.controllers.main-test` for
`test/hello/controllers/main_test.clj`. If you follow this basic convention, you won't go wrong. If your `ns` declaration doesn't match the 
filesystem path, you can get strange errors when you attempt to access it. Think of it much like the dotted-path used to access CFCs in CFML.

The second important part of `ns` is the list of namespaces your code `:require`s. There are two basic forms here:

* `[some.namespace :as alias]` which lets you use `alias/name` as a way to reference any functions defined in `some.namespace`.
* `[some.namespace :refer [fn1 fn2]]` which lets you use `fn1` and `fn2` directly from `some.namespace` without a prefix. The
shorthand `:refer :all` brings in every function from that namespace, and is generally only used in test code.

Prefer the first form -- which you see in `hello.controllers.main` where `hello.services.greeter` is given an alias of `greet` so that
the controller can call `greet/hello`.

You'll use the `:require` expression to bring in Clojure standard libraries, 3rd party Clojure libraries, and parts of your own code. For most
3rd party libraries, you'll also need to add an entry to `:dependencies` in your `project.clj` file in order to tell **Leiningen** that you
need that library downloaded. We'll see this when we learn about database access below.

I recommend writing your required namespaces in alphabetical order -- most production Clojure code relies on quite a long list of other namespaces
so organization is important.

You might think this sounds a bit like `import` statements in Java and you'd be right, except that `:require` is purely for Clojure code.
There is another form for importing Java classes, called `:import`, but Java interop is a whole other subject and you should consult a
Clojure book or the online documentation for more details about that.

# Building Your Own CFML / Clojure Application

Since we want to focus on the Clojure aspect of a combined CFML / Clojure project, we'll
start out by creating a Clojure project, and adding the necessary FW/1 CFML files to it
in order to create a web application.

We will build a very simple task manager, backed by a database. We will initially create
a CFML controller and write a Clojure service to interact with the database, then we'll
replace the CFML controller with a Clojure controller to wrap things up.

## Creating the Clojure project

Our Clojure code doesn't need to be in our CFML webroot so just pick a folder and we'll create a new Clojure application, in which we'll write our task manager code:

    lein new app taskmanager

This will create a folder called `taskmanager` containing a bare bones Clojure project. It'll have a `-main` function so we can use it to run specific operations from the command line, and it'll have tests we can run to validate our code.

## Accessing a Database from Clojure

In order to access a database (via JDBC) from Clojure, you will first need to update your `project.clj` file to include
dependencies on Clojure's JDBC library and what your choice of JDBC driver is. For now we'll use Apache Derby but if you
look at the [java.jdbc](https://github.com/clojure/java.jdbc) page on GitHub, you'll see links to several other common
database drivers (if you use SQL Server, you'll find it easiest to use the jTDS driver to get started!).

Edit `project.clj` and update the `:dependencies` section to include:

    [cfml-interop "0.1.1"]
    [org.apache.derby/derby "10.11.1.1"]
    [org.clojure/java.jdbc "0.4.1"]

You'll now have a vector with four vectors inside it like this:

    :dependencies [[cfml-interop "0.1.2"]
                   [org.apache.derby/derby "10.11.1.1"]
                   [org.clojure/java.jdbc "0.4.1"]
                   [org.clojure/clojure "1.7.0"]]

_Note: if the last vector has `org.clojure/clojure "1.6.0"`, you must update it to use `"1.7.0"` instead!_

The first entry is the open source version of the CFML / Clojure interop library we use at World Singles.
The second entry is the JDBC driver for Apache Derby.
The third entry is Clojure's core JDBC library.
The fourth entry is the Clojure language and core functions.

Now run `lein repl` and we can try this out. As the REPL starts up, it will download the new
libraries and then you'll get the prompt. Let's create test database and write and read some 
data with it:

    taskmanager.core> (require '[clojure.java.jdbc :as sql])
    nil ;; now we have the JDBC library loaded with an alias
    taskmanager.core> (def db {:dbtype "derby" :dbname "/path/to/clojure/taskmanager/cfmltest" :create true})
    #'user/db ;; this is our database spec -- adjust the path accordingly!
    user=> (sql/execute! db [(str "CREATE TABLE task ("
      "id INT GENERATED ALWAYS AS IDENTITY,"
      "task VARCHAR(32),"
      "done BOOLEAN DEFAULT false"
      ")")])
    [0] ;; success! we created the task table 
    taskmanager.core> (sql/insert! db :task {:task "Test database"})
    ({:1 1M}) ;; the sequence of inserted keys:
    ;; there is just one key, labeled :1, with the value 1
    ;; the M indicates a BigDecimal value
    taskmanager.core> (sql/insert! db :task {:task "Read some data"})
    ({:1 2M}) ;; generated key is 2 this time
    taskmanager.core> (sql/query db ["SELECT * FROM task WHERE NOT done"])
    ({:done false, :task "Test database", :id 1} {:done false, :task "Read some data", :id 2})
    ;; our two records came back, let's update one
    taskmanager.core> (sql/update! db :task {:done true} ["id = ?" 1])
    (1) ;; one row was updated
    taskmanager.core> (sql/query db ["SELECT * FROM task WHERE NOT done"])
    ({:done false, :task "Read some data", :id 2})
    ;; yup, that's our only task not done now
    taskmanager.core> (sql/update! db :task {:done true} ["id = ?" 2])
    (1) ;; one row was updated
    ;; let's delete our table to clean up
    taskmanager.core> (sql/execute! db [ "DROP TABLE task"])
    [0] ;; success! we dropped the table

Press control-d to exit the REPL.

Some notes on the syntax:

* Functions that modify the database end in `!` to indicate they have side effects
* SQL and any parameters are specified together in a vector:
  * `["a SQL string" param1 param2 param3]`
  * Use `?` in the SQL string to indicate a parameter placeholder
  * A `PreparedStatement` is used for all operations (which, together with using `?`, protects you from SQL injection attacks)
* `execute!` can be used for arbitrary SQL or DDL
* `query` is intended for `SELECT` only and returns a result set
* `insert!` inserts a new row from a hash map (struct)
* `update!` sets the column values shown in the hash map for all rows that match the `WHERE` clause provided in the vector, so it is shorthand for `["UPDATE table SET col1 = ?, col2 = ?, col3 = ? WHERE clause" param1 param2 param3]`

## Adding the initial CFML files

We're going to put our CFML files in a folder somewhere in our webroot, e.g., a `taskmanager` folder. We'll need `Application.cfc` and an empty `index.cfm`.
Here's what needs to be in our `Application.cfc` file:

    component extends=framework.one {
        variables.framework = {
            diComponent : "framework.ioclj",
            diLocations : "/path/to/clojure/taskmanager"
        };
        function setupRequest() {
            // reload Clojure when FW/1 is reloaded:
            if ( isFrameworkReloadRequest() ) {
                getBeanFactory().reload( "all" );
            }
        }
    }

We need to tell FW/1 to use the `ioclj` extension to DI/1, and we need to tell it the full path to the Clojure code (the folder where our `project.clj` lives).
We also add the code to programmatically reload our Clojure files, in `setupRequest()`.

Then we need a default `main.default` view so we can test the app:

    <!--- views/main/default.cfm --->
    Hello, World!

If this loads without error, FW/1 has successfully located your Clojure project and loaded it.

## Writing a Clojure Service

We'll start by writing a simple service containing just a single function so we can check that FW/1 finds and loads it correctly and that we can access it from our view.
In your `taskmanager` Clojure folder, create a `services` subfolder under `src/taskmanager` and then create this `greeting.clj` file:

    ;; src/taskmanager/services/greeting.clj
    (ns taskmanager.services.greeting)
    
    (defn hello [name] (str "Hello, " name "!"))

Now we'll update our `main.default` view to look like this:

    <cfoutput>
      #getBeanFactory().getBean("greetingService").hello( "Clojure" )#
    </cfoutput>

If you just try to hit the taskmanager app in your browser, you'll get an error that `greetingService` does not exist. You need to reload FW/1 (via adding `?reload=true` to the URL)
so that it will reload the bean factory and find our newly created Clojure service. If you do that, you should see `Hello, Clojure!` in your browser.

Now we can get to work writing our simple task manager service!

We already wrote most of the code we need, as part of our REPL session above, and if you look in that `taskmanager` folder, you'll find a (hidden) file called `.lein-repl-history` which contains the code you typed in.
We're going to use that as the basis of our service. Let's create `task.clj` in our `services` folder like this:

    ;; src/taskmanager/services/task.clj
    (ns taskmanager.services.task
      (:require [clojure.java.jdbc :as sql]))
    
    ;; for now we'll just hard-code one database spec but
    ;; we could pass it in from our controller as needed
    (def db {:dbtype "derby" :dbname "cfmltest" :create true})
    
    (defn create-task-table []
      (sql/execute! db [(str "CREATE TABLE task ("
                             "id INT GENERATED ALWAYS AS IDENTITY,"
                             "task VARCHAR(32),"
                             "done BOOLEAN DEFAULT false"
                             ")")]))
    
    (defn add-task
      "Given a task name, add it to our database and return the new row's ID."
      [task]
      (-> (sql/insert! db :task {:task task}) first :1))
    
    (defn complete-task
      "Given a task ID, mark it as done and return the number of rows updated."
      [id]
      (-> (sql/update! db :task {:done true} ["id = ?" id]) first))
    
    (defn task-list
      "Return the tasks.
      If all is true, return all tasks, else just the incomplete ones."
      [all]
      (sql/query db [(str "SELECT * FROM task"
                          (when-not all " WHERE NOT done"))]))

You can try out each function in the REPL or you can write unit tests, but they're all fairly simple and we "tested" them when we were exploring how
to work with a database in the REPL earlier.

## Fleshing out the Views

Next we'll create the views we need for our simple task manager. The default view will show a list of tasks with options to:

* List all / outstanding tasks
* Add a new task
* Next to each task, the status, and the ability to mark a task as complete

We'll reuse `main.default` as our list view and add `main.newtask` for adding new tasks. We'll additionally have actions for `main.addtask` and `main.completetask` which will both
redirect to `main.default`. We'll have an optional boolean argument to `main.default` which will specify showing all tasks or just outstanding tasks.

Here's our `main.default` view:

    <!--- views/main/default.cfm --->
    <cfparam name="rc.all" default="false"/>
    <cfoutput>
        <p>
            <cfif rc.all>
                <a href="#buildURL( 'main.default?all=false' )#">List To-Do</a>
            <cfelse>
                <a href="#buildURL( 'main.default?all=true' )#">List All</a>
            </cfif>
            |
            <a href="#buildURL( 'main.newtask' )#">Add New Task</a>
        </p>
        <cfif arrayLen( rc.tasks )>
            <ul>
                <cfloop index="task" array="#rc.tasks#">
                    <li>
                        <form action="#buildURL( 'main.completetask' )#" method="post">
                            #task.task#
                            <cfif !task.done>
                                <input type="hidden" name="id" value="#task.id#"/>
                                <input type="submit" value="Done!"/>
                            </cfif>
                        </form>
                    </li>
                </cfloop>
            </ul>
        <cfelse>
            <p>You have nothing to do!</p>
        </cfif>
    </cfoutput>

Here's our `main.newtask` view:

    <!--- views/main/newtask.cfm --->
    <cfoutput>
        <form action="#buildURL( 'main.addtask' )#" method="post">
            <p>
                Task:
                <input name="task" type="text"/>
                <input type="submit" value="Add!"/>
            </p>
        </form>
        <p><a href="#buildURL( 'main.default' )#">Cancel</a></p>
    </cfoutput>

## Testing with a CFML Controller

We'll add a dummy controller so we can test this:

    // controllers/main.cfc
    component accessors=true {
        property framework;
        function default( rc ) {
            rc.tasks = [ ];
        }
        function addtask( rc ) {
            framework.redirect( 'main.default' );
        }
        function completetask( rc ) {
            framework.redirect( 'main.default' );
        }
    }

If we hit the app in our browser with `?reload=true` in the URL, you should see:

    List All | Add New Task
    
    You have nothing to do!

If you click `Add New Task` and fill in the form and click `Add!`, you'll end up back on this page with no tasks listed.

### Using the Clojure Service

Now we'll wire in our Clojure service that we wrote above and make something useful happen!

We'll change our controller to look like this:

    // controllers/main.cfc
    component accessors=true {
        property framework;
        property taskService; // add this
        function default( rc ) {
            // default rc.all and pass boolean to Clojure:
            param name="rc.all" default="false";
            rc.tasks = taskService.task_list( rc.all ? true : false );
        }
        function addtask( rc ) {
            // call Clojure
            taskService.add_task( rc.task );
            framework.redirect( 'main.default' );
        }
        function completetask( rc ) {
            // call Clojure
            taskService.complete_task( rc.id );
            framework.redirect( 'main.default' );
        }
    }

_Note: because URL and form values come into CFML as strings that it will automatically coerce to numeric, boolean, etc, we need to explicitly pass either `true`
or `false` to `taskService.task_list()` which is why we write `rc.all ? true : false`. Most languages do not have CFML's flexibility when interpreting data, but
they also provide a lot more type safety! You might wonder why we don't convert `rc.id` to numeric in the call to `taskService.complete_task()`? We can get away
with it there because it's passed directly to the JDBC driver and that knows that the `id` field is numeric and it will parse the string for you. In general,
you probably shouldn't rely on that and should explicitly convert your inbound string data from `rc` to the type your Clojure code is expecting._

If we run this (with `?reload=true` in the URL), we'll probably get a SQL exception because our table doesn't exist. We dropped the table at the end
of our REPL session (unless you recreated it again while testing the `task.clj` service?).

Let's make our controller automatically create the table if the task list fails:

        function default( rc ) {
            // default rc.all and pass boolean to Clojure:
            param name="rc.all" default="false";
            try {
                rc.tasks = taskService.task_list( rc.all ? true : false );
            } catch ( any e ) {
                taskService.create_task_table();
                rc.tasks = taskService.task_list( rc.all ? true : false );
            }
        }

Run it again (with `?reload=true`) and you should see an empty list of tasks. Add a new task and... oh dear! We got an exception:

    Key [TASK] doesn't exist in Map (clojure.lang.PersistentArrayMap)

The reason for this is that raw Clojure structs (hashmaps) are not quite compatible with CFML structs (because they use keywords as keys,
whereas CFML uses case-insensitive strings). The Clojure integration provides a way to convert back and forth, so we'll need our
controller to depend on `cfmljure`, and then we'll use `toCFML()` to convert the Clojure data structure
to a CFML-compatible data structure so we can display it:

    // controllers/main.cfc
    component accessors=true {
        property framework;
        property taskService;
        property cfmljure; // add this
        function default( rc ) {
            param name="rc.all" default="false";
            try {
                rc.tasks = taskService.task_list( rc.all ? true : false );
            } catch ( any e ) {
                taskService.create_task_table();
                rc.tasks = taskService.task_list( rc.all ? true : false );
            }
            rc.tasks = cfmljure.toCFML( rc.tasks ); // make it CFML-compatible
        }
        ...
    }

_Note: if you don't add the `cfml-interop` dependency to a project, the `toCFML()` and `toClojure()` functions available in
**cfmljure** will still "work" but the struct keys will be case-sensitive and **lowercase** so you would need to use `myStruct['key']`
notation to access them instead of just `myStruct.key`._

## Writing a Clojure Controller

In this final section of **Building Your Own CFML / Clojure Application**, we're going to replace our CFML Controller with an equivalent Clojure
Controller. In your Clojure `taskmanager` project, create a `src/taskmanager/controllers/` folder and inside it create `main.clj` containing:

    ;; src/taskmanager/controllers/main.clj
    (ns taskmanager.controllers.main
      (:require [cfml.interop :refer [->boolean]
                [taskmanager.services.task :as task]))

    (defn default [rc]
      (let [all? (->boolean (:all rc))
            tasks (try
                    (task/task-list all?)
                    (catch Exception _
                      (task/create-task-table)
                      (task/task-list all?)))]
        (assoc rc :tasks tasks)))

    (defn addtask [rc]
      (task/add-task (:task rc))
      (assoc rc :redirect {:action "main.default"}))

    (defn completetask [rc]
      (task/complete-task (:id rc))
      (assoc rc :redirect {:action "main.default"}))

This is the equivalent of our CFML controller above so let's walk through each piece of it:

1. `(ns ...)` specifies `taskmanager.controllers.main` as our namespace and makes our service available with the `task` alias.
1. `default` does the following:
    1. binds `all?` to the boolean value of `rc.all` (which comes in as the string `"true"` or `"false"`!).
    1. binds `tasks` to the result of calling `task-list` in our service (including catching any exception, attempting to create the task table, and calling `task-list` again -- `try`/`catch` is an expression in Clojure that returns a value, just like any other expression).
    1. returns `rc` with the task list added (as `rc.tasks`).
1. `addtask` calls `add-task` in our service and returns `rc` with a redirect added to it.
1. `completetask` calls `complete-task` in our service and returns `rc` with a redirect added to it.

Now run this with `?reload=true` in the URL and it should work just as it did before, except now it's using the Clojure controller instead of the CFML controller!
How do you know it's using the Clojure version? Because `framework.ioclj` adds the Clojure namespaces it finds after any CFCs it finds, overwriting any beans
with the same alias. If you want to convince yourself, remove `controllers/main.cfc` from your `taskmanager` folder in the webroot, and reload the application again.

So why would we write our controllers in Clojure instead of CFML? 

* As simple functions (that take `rc` as input and produce an updated `rc` as output), they're easy to write unit tests for. 
* You can work in the REPL building and testing your entire application's functionality (and then work on the views with a browser).
* We get the full power of Clojure's data abstractions, concurrency, and immutability working for us.
* With your services and controllers in Clojure, you're one step away from building all-Clojure web applications using [FW/1 for Clojure](https://github.com/framework-one/fw1-clj). 

### RC Value Conversion

Since all values come into the `rc` (from URL and form scope) as strings, you'll usually need to convert them to the appropriate numeric, boolean,
or other data type in order to use them in your Clojure code. We saw above that you can get away with some things, but in the pure Clojure world,
you need to be much more specific about types.

The [`cfml-interop` library](https://github.com/seancorfield/cfml-interop), containing the `cfml.interop` namespace, contains some useful functions for converting from string to various types - the `->type` naming is just a convention in Clojure for conversion
functions:

* `->long` -- Accepts any value, string or numeric, and tries to convert it to a `Long` value. You can provide a second argument which specifies the default to return if conversion fails (otherwise it will return `0`).
* `->double` -- Accepts any value, string or numeric, and tries to convert it to a `Double` value. You can provide a second argument which specifies the default to return if conversion fails (otherwise it will return `0.0`).
* `->boolean` -- Accepts any value, boolean, string or numeric, and tries to convert it to a `Boolean` value. You can provide a second argument which specifies the default to return if conversion fails (otherwise it will return `false`). This treats the strings `"true"` and `"yes"` as `true` (not case sensitive), as well as treating non-zero numeric values as `true` (and zero as `false`), just like CFML.

# A Clojure Primer

To learn about Clojure in any depth, I'd recommend you go through the **[More Stuff to Read](#more-stuff-to-read)** section at the end of this
page, but I'm going to give you a quick run through of some useful basics that should get you up and running more quickly.

As I claimed earlier, Clojure is a very simple language with only a few pieces of syntax:

* A semicolon introduces a comment. Typically a single semicolon `;` is used for an end of line comment and a double semicolon `;;` is used for
a whole line comment.
* `(func arg1 arg2 arg3)` represents a function call (with three arguments). You can use commas if you want but they are
just whitespace: `(func arg1, arg2, arg3)`. Most Clojure developers omit commas in function calls. Almost everything in Clojure is a function call.
A few things that look like function calls are actually "[special forms](http://clojure.org/special_forms)" but you can pretend they're really function calls
until you get a bit more proficient (a function call evaluates all its arguments before the call, a special form may not
evaluate its arguments until it needs them).
* `a` is a symbol -- a variable or function name -- that evaluates to whatever it has been bound to. See `def`, `defn` and `let` below.
* `:a` is a keyword -- it evaluates to itself -- that is a bit like a string except it is cached (so two `:a`s in different parts of
your code are the exact same thing and can be compared for identity based on their address, not their value). Keywords are commonly
used as the keys in a hash map (see below).
* `[1 2 3 4]` represents a vector (array) with the specified elements. You can use commas if you want, but they are just whitespace:
`[1, 2, 3, 4]`. Most Clojure developers omit commas in vectors.
* `{:a 1, :b 2, :c 3}` represents a hash map (struct) with keys `:a`, `:b`, `:c` and values `1`, `2`, `3` respectively. The commas
are just whitespace but some Clojure developers use them anyway to clearly delineate pairs.
* `#{1 2 3}` represents a set of values. If you add `4` to that set you get `#{1 2 3 4}` but if you add `1`, `2`, or `3` to it, it remains
unchanged because those values are already in the set. CFML doesn't have a set data type, but you might simulate it with a struct whose
values are `true` (or some other arbitrary value) and the presence or absence of a key says whether it's in the set or not.

That's about it. When you learn about macros (in one of the Clojure books), you'll encounter a few new pieces of syntax but you
don't need that to get going.

## Some Basic Clojure Functions

Functions all the way down. Some functions / special forms are very important so I'm going to go through some of those here.

### `def`, `fn`, and `defn`

The `def` special form creates a global (top-level) binding of a name to a value within the current namespace:

    (def a 42)
    ;; binds a to the value 42 -- technically a is a Var
    a
    ;; produces 42

The `fn` special form creates an anonymous function:

    (fn [a b c] (* a b c))
    ;; creates a function with three arguments a, b, and c
    ;; that multiplies those three values together

Anonymous functions are often used as arguments to other functions:

    (map (fn [x] (* x 2)) [1 2 3 4 5])
    ;; produces (2 4 6 8 10)

Strictly speaking, a function can contain more than one expression: they are all evaluated in order, but only the value of the last
expression is returned. The others are thrown away. So why have multiple expressions? You might have some operations that cause side
effects, such as logging to file or writing to a database, and you want to evaluate those for their effects but not necessarily
their result.

You can bind a name to an anonymous function, to create a named function:

    (def twice (fn [x] (* x 2)))

This is so common that `defn` exists as a shorthand for it:

    (defn twice [x] (* x 2))

Documentation is built into Clojure and you can (and should) provide a "docstring" for all your functions that say what they do
and possibly provide example usage:

    (defn twice "Doubles its argument." [x] (* x 2))

You'll often see this written on multiple lines like this:

    (defn sum-of-squares
      "Given two values, return the sum of their squares."
      [a b]
      (+ (* a a) (* b b)))

It's common for functions to have short argument names because each function should be very short and simple, and the function names should
be descriptive.

The docstring -- and the source code -- of functions is available in the REPL (and in any Clojure editor that supports live evaluation) through
the `doc` -- and `source` -- functions (in the `clojure.repl` namespace). This makes it very easy to experiment with new libraries since all
of the documentation and source code is right there at your fingertips!

### `if`, `when`, and `do`

In CFML we have an `if` statement. If the condition is true, the first group of statements is executed, else the second group of statements
is executed. In functional languages like Clojure, `if` is an expression (a special form) and it takes a condition and two expressions.
If the condition is true, the first expression is evaluated, else the second expression is evaluated. It's more like the ternary operator
`? :` in CFML.

Also, it's important to note that Clojure has strong views on what is true and false. You'll hear of "truthy" and "falsey" in the Clojure
world because Clojure treats everything as true except for `false` and `nil`. In particular, that means that `0` is true, unlike in CFML.
The reason for this is an idiom called "nil punning": it's common for Clojure functions to return `nil` for "no such value" instead of
throwing an exception, and allowing `nil` to mean false makes it easy to test for such things:

    (if (:name rc) (str "Hello " (:name rc) "!") "Who?")

In CFML, if you did `rc.name` and it wasn't present, you'd get an exception. In Clojure, `(:name rc)` simple returns `nil` if there's no
such key and you just test for that. You can prevent repetition of `(:name rc)` with `if-let` -- see below.

If you don't have an "else" expression (and want to return `nil` instead), use `when` instead of `if`:

    (when (:name rc) (str "Hello " (:name rc) "!"))

If the condition is "falsey", you get `nil` back. Be aware that `when` allows multiple expressions, just like `fn` above and `let` below, and it evaluates
all of them but only returns the value of the last expression.

If you want to evaluate multiple expressions in an `if`, you need `do`:

    (if some-condition
      (do
        (log-something)
        (update-the-database)
        "Result!")
      "Failure!")

`do` can be used where any single expression is accepted and behaves like the body of `fn`.

It might not surprise you to learn that `when` is really defined in terms of `if` and `do` something like this:

    (when condition expr1 expr2 expr3)
    ;; is treated like
    (if condition (do expr1 expr2 expr3) nil)

### `let` and its cousins

Global bindings are fine for functions that you want exposed to the world, but you often want local bindings inside a function. That's what the
`let` special form is for:

    (defn sum-of-squares
      "Given two values, return the sum of their squares."
      [a b]
      (let [a-squared (* a a)
            b-squared (* b b)]
        (+ a-squared b-squared)))

`let` is followed by a vector of bindings -- pairs of symbol and expression -- and then one or more expressions. It creates a local binding
for each pair by evaluating the expression and binding it to the name (symbol), and then it evaluates the expressions in its body and returns
the last expression's value (in the same way function bodies are evaluated).

There are several variants of `let` that are useful. The most common is probably `if-let`:

    (if-let [name (:name rc)]
      (str "Hello " name "!")
      "Who?")

Unlike `let`, `if-let` only allows one binding pair followed by two expressions. If the binding is "truthy", the first expression will be evaluated,
else the second expression will be evaluated. Note that the bound symbol is only available in the _first_ expression!

### `loop` and `recur`

Since I often tell people that functional programming means no mutable variables and therefore no loops, it might surprise you to learn that
Clojure has a `loop` construct. It isn't quite what it seems!

First, let's look at a recursive function:

    (defn fact [n] (if (zero? n) 1 (* n (fact (dec n)))))
    ;; (fact 1) => 1
    ;; (fact 3) => 6
    ;; (fact 5) => 120

Because it is recursive, it creates a new stack frame for every call to itself. You can still call it with some big numbers but eventually you will get
a stack overflow:

    (fact 1000N) => 402387260077093773543702433923003985719374...
    (fact 10000N) => StackOverflowError   java.math.BigInteger.valueOf (BigInteger.java:1098)

There's a technique called tail recursion that allows some languages to evaluate recursive functions without needing stack frames. You
have to rewrite the function to ensure the recursive call is in the tail position (the `fact` call above is nested inside multiplcation).
Often, you need to create a helper function:

    (defn fact-helper [n prod] (if (zero? n) prod (fact-helper (dec n) (* n prod))))
    (defn fact [n] (fact-helper n 1))

This produces the same results as `fact` above, but now the recursive call to `fact-helper` is in the tail position. Clojure doesn't optimize this
directly but provides `recur` for you to tell the compiler you want it optimized -- and to verify you really are using tail recursion:

    (defn fact [n] (if (zero? n) 1 (* n (recur (dec n)))))
    ;; CompilerException: Can only recur from tail position

But:

    (defn fact-helper [n prod] (if (zero? n) prod (recur (dec n) (* n prod))))
    (defn fact [n] (fact-helper n 1))
    (fact 10000N) => 28462596809170545189064132121198688901480514...

You can see that `recur` replaces the recursive call, but what it's really doing behind the scenes is rebinding the function arguments
and "jumping" back to the beginning of the function, to avoid a function call altogether. Since this is really a loop-with-rebinding,
Clojure provides a `loop` construct that lets you do this directly:

    (defn fact [n] (loop [n n, prod 1] (if (zero? n) prod (recur (dec n) (* n prod)))))

This single function is identical in behavior to the `fact` / `fact-helper` pair shown immediately above. You can see how the `loop`
binding behaves just like the initial call to `fact-helper`, binding `fact`s argument `n` to the local symbol `n` and `1` to the local
symbol `prod`.

### Useful core Functions

Since Clojure is all about data structures, there is a rich selection of functions that operate on them. Some of the valuable ones to know are:

* `(first [1 2 3 4])` returns the first element of a sequence: `1`
* `(rest [1 2 3 4])` returns the rest of a sequence: `(2 3 4)`

The `rest` of a single element sequence is an empty sequence -- `()` -- and the `rest` of an empty sequence is _also_ an empty sequence, as is `rest` of `nil`!

* `(seq some-collection)` returns a sequence of `some-collection`s elements if it is non-empty, else returns `nil`

It is common to test for empty sequences with `if (seq some-collection)`.

* `(cons 1 [2 3 4])` returns a new sequence with that element added: `(1 2 3 4)`
* `(assoc {:a 1} :b 2)` returns a new hash map with the key associated to the value: `{:a 1, :b 2}`
* `(dissoc {:a 1, :b 2} :a)` returns a new hash map with the key removed (dissociated): `{:b 2}`
* `(map inc [1 2 3 4])` returns a new sequence with the function applied to each element: `(2 3 4 5]`
* `(filter even? [1 2 3 4])` returns a new sequence containing just the elements for which the predicate is "truthy": `(2 4)`
* `(reduce + 0 [1 2 3 4])` returns the value computed by repeatedly applying the function to the initial value and each element of the sequence: `10`

If the initial value is omitted, the first element of the sequence is used so:

    (reduce func some-collection)
    ;; is the same as
    (reduce func (first some-collection) (rest some-collection))

## The project.clj File

The piece of `project.clj` you'll touch most often is the `:dependencies` entry. This is a list of all the libraries your
program needs and the versions of each you want to use:

    :dependencies [[clj-time "0.9.0"]
                   [org.clojure/clojure "1.7.0"]]

Libraries come from two locations by default: [Maven Central](http://search.maven.org) and [Clojars](https://clojars.org).

Most Clojure libraries tell you what to put in `project.clj` to pull them in so you mostly won't need to care which location
they actually come from but it's instructive to at least know how to read this stuff and how to search for libraries yourself.

Each entry is called a "coordinate" and contains a "group ID" and an "artifact ID". A single name just means that the group ID
and artifact ID are the same thing (so `clj-time` is shorthand for `clj-time/clj-time`).

To search Maven Central for `org.clojure/clojure` you would use the query `g:"org.clojure" AND a:"clojure"` which asks for
group ID `org.clojure` and artifact ID `clojure`. Right now there are 80 versions of that library on Maven Central and the 
latest is `1.8.0-alpha4` but if you click the `All (80)` link, you'll see the most recent non-prerelease version is `1.7.0`
which is what **Leiningen** puts in `project.clj` by default (in Leiningen 2.5.2 and later).

On the other hand, `clj-time` comes from Clojars because it is a community project. If you search for `clj-time` you'll 
get a lot of results but most of them are not canonical versions. The most recent canonical version is https://clojars.org/clj-time but
there are other, earlier canonical versions, such as https://clojars.org/backtype/clj-time so you need to be a bit careful. If in doubt,
get on IRC, Slack, or the mailing list and ask!

For an overview of all the possible settings in `project.clj`, take a look at the [Sample project.clj File on GitHub](https://github.com/technomancy/leiningen/blob/master/sample.project.clj).

# About Functional Programming

Functional programming isn't new. It's origins lie in Lisp which was created in the 1950's and is the second-oldest computer language
(second only to FORTRAN). Throughout the 70's and 80's a lot of functional languages were created, mostly in academia, to study the 
benefits of the functional style, as well look at levels of expressiveness in programming languages. Classic functional languages
include Standard ML, Miranda, and Haskell. Haskell was a result of the proliferation of similar functional languages being created
by each university in England (and elsewhere). It was decided that a single, committee-designed functional language should exist 
that included the best ideas of all of the diverse variants out there. Haskell is probably the most widely used language today from
that era. It has an extremely powerful type system and a very strong view of purity -- lack of side effects -- but it has been
used extensively over the last 25 years in industry as well as academia.

The recent resurgence of functional programming has shown itself in languages like F# from Microsoft, Scala, and Clojure, even Rust, as well
as some compile-to-JS languages like Elm and PureScript. The reason behind this resurgence is that immutable data structures and
pure functions offer the ability to write concurrent code a lot more easily and lot more safely than the mainstream OOP approach.
And we need concurrency in order to take advantage of multi-core machines, now that we're no longer seeing continued speed increases
in individual cores like we had for the previous several decades.

While it may seem obvious that functional programming leans heavily on functions as building blocks, the real core values of
functional programming are avoiding mutable data and avoiding side effects in functions. The more that you can push side effects
to the edges of your program, the more of your code becomes pure functions that can be easily reasoned about, easily tested, and
often easily reused. Functional programming focuses on small, pure functions that can be composed to create larger pieces of
functionality. If you have a function `inc` that adds one to its argument and a function `twice` that doubles its argument, then
`(comp twice inc)` is a function that adds one to its argument and then doubles it: functions are like Legos that you can easily
assemble to build products.

## Immutable or Persistent Data Structures

In the context of functional programming, you'll hear a lot of talk of immutable data structures and persistent data structures.
In OOP languages, you typically perform operations on a data structure to modify it in place. That means you can't safely share
it with other pieces of code, especially across multiple threads. By contrast, in a functional language, when you perform an
operation on a data structure, you get back a new data structure that shares as much structure as possible with the original
data structure -- and yet leaves the original data structure unchanged.

While they are designed for efficiency, it is usually at scale, rather than for small examples. In Clojure, many data structures
are "chunked" internally into groups of 32 elements. A vector of 100 elements is going to be four chunks and is optimized for
adding elements to the end of the vector. A list is optimized for adding elements at the start of it. A hash map is also chunked
and optimized for adding elements in random locations. They are also optimized for different patterns of access: a list is optimized
for purely sequential access, a vector for indexed (random) access, and a hash map for keyed (random) access.

Clojure and Scala share a lot of heavily optimized implementation details in their persistent data structures.

Despite all this efficiency, you still need to think about how to use data structures, and there are going to be some algorithms
where bashing a data structure in place is just going to be faster. It won't be as safe, just faster. Clojure is fine with the 
idea of localized mutation and has versions of vectors and hash maps that are optimized for that purpose (known as transients).

The most important aspect of these data structures in Clojure is the set of abstractions over them, including "sequence"
and "associative". These are uniform ways to think
about data structures as just sequences of data (accessed in order, or randomly by keys),
no matter what their actual implementation is, so that you can apply all of
Clojure's core functions to arbitrary data structures in standardized ways. The lack of unique types for each data structure
you use means that you don't need unique functions to operate on them all. A function that operates on a sequence can accept
any data structure that supports the sequence abstraction. A function that operates on an associative collection can accept
any data structure that supports the associative abstraction.

For example, a result set from a database query is simply a sequence of associative collections. It can be mapped, filtered,
reduced using standard functions and its rows transformed using any of the standard associative functions. This allows
abstraction over different data stores as well since, from Clojure's point of view, MySQL and MongoDB look very similar.

## Functions as Building Blocks

I've already emphasized that functional programming favors small, simple, pure functions but you can do that sort of
functional decomposition in most languages. Many modern languages allow you to write anonymous functions and pass them
around as arguments, as well as return them from functions. Even with those features available, it takes a while to
shift from an imperative style with some functions being passed around to a functional style where functions are your
primary abstraction.

You'll hear the term "higher order functions" a lot but all this means is a function that accepts a function as one (or
more) of its arguments or a function that returns a function as a result. You'll see the former in CFML with the new
`map`, `filter`, and `reduce` member functions -- or perhaps you've seen it in JavaScript.

Consider these two functions (in CFML):

    function saveStuff( data ) {
        if ( validStuff( data ) ) {
            dbSave( "stuffTable", data );
        } else {
            writeLog( "Invalid stuff, not saved" );
        }
    }
    
    function saveThing( data ) {
        if ( thingIsValid( "all", data ) ) {
            dbSave( "thingTable", data );
        } else {
            writeLog( "Invalid thing, not saved" );
        }
    }

Now, consider this function:

    function saveValidData( validator, type, data ) {
        if ( validator( data ) ) {
            dbSave( type & "Table", data );
        } else {
            writeLog( "Invalid #type#, not saved" );
        }
    }

This is probably not a transform you would do in CFML but it's a natural one in a functional language, because
you would see the commonality and want to remove the duplication:

    var saveStuff = function( data ) { return saveValidData( validStuff, "stuff", data ); };
    var saveThing = function( data ) {
        return saveValidData(
            function( data ) { return thingIsValid( "all", data ); },
            "thing",
            data
        );
    }

Now imagine you had a higher order function called
`partial` that accepted a function and one or more of its arguments and returned a new function that accepted
the rest of its arguments and then called it, it would be very natural to make this transform and then write:

    var saveStuff = partial( saveValidData, validStuff, "stuff" );
    var saveThing = partial( saveValidData, partial( thingIsValid, "all" ), "thing" );

This is much cleaner, especially if you had quite a few of these "save data if valid" variants.

Another way to write this without `partial` would be:

    function thingIsAllValid( data ) { return thingIsValid( "all", data ); }
    function saveValidData( validator, type ) {
        // accept validator and type, return a function that accepts data
        return function( data ) {
            if ( validator( data ) ) {
                dbSave( type & "Table", data );
            } else {
                writeLog( "Invalid #type#, not saved" );
            }
        }
    }
    
    var saveStuff = saveValidData( validStuff, "stuff" ); // returns a function
    var saveThing = saveValidData( thingIsAllValid, "thing" );

You could also write "thing" validation like this:

    function thingValidator( scope ) {
        return function( data ) {
            return thingIsValid( scope, data );
        }
    }
    
    var saveThing = saveValidData( thingValidator( "all" ), "thing" );

By making our functions more flexible in how they accept arguments, we make it easier to reuse them. This is functional thinking!

In Clojure we can define a function with multiple argument lists so this becomes even easier:

    (defn thing-is-valid
      ([scope] (fn [data] (thing-is-valid scope data)))
      ([scope data] ... return true or false ...))
    
    (defn save-valid-data
      ([validator type] (fn [data] (save-valid-data validator type data)))
      ([validator type data]
       (if (validator data)
         (db-save (str type "Table") data)
         (write-log (str "Invalid " type ", not saved")))))
    
    (def save-stuff (save-valid-data valid-stuff "stuff"))
    (def save-thing (save-valid-data (thing-is-valid "all") "thing")

No need for `partial` (although Clojure has that built-in), no need for helper functions.

## All You Know About OO Programming is Wrong

I learned old-fashioned imperative procedural programming first. I learned BASIC, assembly language, Pascal and later
COBOL and FORTRAN. Although OOP has its roots back in the 50's and 60's (ironically, with Lisp, just like FP has its
roots in Lisp), it didn't really go mainstream until the mid-to-late 80's with the arrival of Eiffel and C++. I
learned a lot of FP during the 80's but since it wasn't going mainstream and OOP was, I switched horses to stay
employable. By the time Java appeared, OOP had become the default "standard" way to build software, even though
C++ and Java were not at all what Alan Kay had in mind when he coined the phrase "object-oriented".

What most developers know as modern OOP focuses on polymorphism, inheritance, encapsulation and objects that have both
state and behavior bundled together. You construct an object with its initial state. You run a bunch of methods on it,
interacting with other objects, to modify its state, and then you query the object to get that state back out.

A common idiom in the OOP world for collections is an iterator. In CFML this shows up in query objects. As you loop
over the iterator, it changes its state to refer to successive elements of the underlying collection. A CFML query
refers to successive rows of the result set as you loop over it and the `currentrow` element is updated at each
iteration. Instead of a result set looking like a sequence of rows, we're used to looking at a snapshot of the "current
row" -- and that's what iterators do to us for collections as well.

In addition, instead of processing collections holistically to produce either new collections or specific results,
we're using to iterating through the elements and either modifying them in place or performing side effecting operations
along the way.

What makes this problematic is that you can't then easily run this code concurrently to take advantage of multiple cores:
code that mutates collections in places or generates side effects is rarely thread safe.

In other words, mutable state is bad.

What about OOP's other basic tenets?

Why do you encapsulate data? The primary reason is so that you can control mutation of that data. If your data is
immutable, encapsulation is no longer needed for that. The other argument for encapsulation is so that you can
change the representation of the state without affecting your clients. In a functional world, if you change your
representation, you can always provide a function that transforms it to the original structure and then simply
compose that transformation with any client function that needs to access it. That composed client + transform
can be refactored away over time as the client transitions to the new API. In other words, your data is your API
and there's no need to encapsulate it (at least, not for the traditional OOP reasons).

Inheritance? Inheritance exists in OOP because the notion of data types is inherently tied to classes and objects.
Inheritance represents a strong coupling between an implementation and an interface or between one implementation
and another related implementation. It exists because there's no way to separate out the notion of data types and
their relationships from the class implementation relationships. Needless to say, in a functional world, you can
choose to have relationships between data types as you need them, without being forced to create relationships
between data representations. In Clojure, in particular, you can create hierarchies of types independent of
any data and use those to guide function call dispatching.

Which brings us nicely to polymorphism! Polymorphism is great. It's very useful. Unfortunately, in OOP it is
tied to inheritance which, as we've just seen, is all about coupling when you're dealing with classes. If you
only ever use interfaces and pure implementations, you can free yourself from some of the problems of coupling
forced on you by OOP, but you still get stuck if you have a class (implementation) that isn't declared to
implement an interface that you need to use, even though it has the right methods. Nor can you easily take
an arbitrary existing class and make it implement your interface (you can extend the class and implement your
interface, only if the class is not final in Java, for example). In addition to all that, polymorphism in OOP
is only effective on the first argument -- the object type itself -- which means that when faced with more
complex problems, you have to resort to design patterns like Visitor and implement double dispatch. But then
you are forced to modify the "visited" class every time you want to visit it with a new class. What you really
need there is polymorphism based on multiple arguments. Fortunately, you can have that in functional programming.

So what have we learned?

Polymorphism is very limited in OOP but can be very powerful in FP once we remove the restriction of single
dispatch and the coupling to a static inheritance hierarchy: this gives you ad hoc or a la carte polymorphism.

Inheritance as seen in OOP is essentially an implementation detail. You can have ad hoc inheritance in FP,
which provides expressiveness where you want it and avoids boilerplate and coupling where you don't.

Encapsulation is required in OOP when you have mutatable state and is also needed in order to allow changes in
implementation. In other words, encapsulation is also essentially an implementation detail. FP solves this
by making data immutable and making functions easily composable.

Mutable state is just bad. It prevents refactoring to leverage concurrency, it leads to hard to find bugs
in complex programs, and it also erases any notion of time in your program (because you have only the
current version of the state, rather than the series of values that were transformed to get to that point).
FP solves that by removing mutability.

If FP is so great, why aren't we all using it already?

Good question. The OOP industry is vast. Design Patterns, training and consulting, higher education based on
teaching OOP (ironically after supplanting a lot of courses that taught FP!), testing, tooling, IDEs. The
momentum behind OOP is huge and the inertia of industry to keep doing things the way they know is almost 
overwhelming. Yet we see functional features in nearly every new language being designed, we see functional
features being added to nearly every existing language over time, we see the functional style of programming
being advocated even in traditional languages -- with less reliance on mutable state. Most of that pressure
is coming from the need to do more concurrency and to avoid the bugs that arise from side-effecting code.
In other words, a lot of people already know that FP is a better way to solve a lot of problems.

Remember that modern OOP -- as enshrined in Java and C# particularly -- is not what the originators of OOP had
in mind. They imagined objects as proxies for real world elements such as displays and control devices, that
objects would be coarse-grained and communicate by sending messages between themselves. _In other words, they
would be more like "actors"... which you have in both Clojure and Scala!_

# More Stuff to Read

Once you've got a taste for Clojure, there are lots of online resources and a host of great books you can read.
Here's a small sample, roughly in order of approachability:

* The online tutorial for learning Clojure, no installation required: http://www.tryclj.com
* The "4Clojure" puzzles online: https://www.4clojure.com 
  * These quickly get hard enough that you'll want a REPL open locally to play with!
* The Clojure Koans: http://clojurekoans.com
  * You need at least Java, Leiningen, and Git installed for these.
* Some great books to read:
  * Clojure Programming http://www.clojurebook.com followed by
  * The Joy Of Clojure http://www.joyofclojure.com :)

# Digging Into Reloading

_More to come on this, eventually!_
