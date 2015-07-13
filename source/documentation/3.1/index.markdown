---
layout: page
title: "Getting Started with FW/1"
date: 2015-07-12 21:40
comments: false
sharing: false
footer: true
---
FW/1 was created in July 2009 as a reaction against complexity and bloat in other frameworks in the CFML community. FW/1 itself is a single file, and provides a simple, convention-based approach to MVC (Model-View-Controller) applications. Whilst it has become more sophisticated over time, it has remained a single file, focused on getting out of your way and providing the intuitive plumbing you need. For historical background, you can read the [introductory blog post](http://framework-one.github.io/blog/2009/07/19/introducing-framework-one/) from July 2009.

As of release 3.1, FW/1 also includes DI/1 - a simple, convention-based Dependency Injection framework - and AOP/1 - a simple, convention-based Aspect-Oriented Programming framework. If those phrases don't mean anything to you, don't worry, you won't need to know anything about them to get started.

Requirements & Supported Platforms
---
FW/1 3.1 supports Adobe ColdFusion 9.0.2 or later (not 9.0.0 or 9.0.1), Lucee 4.5.0 or later, and Railo 4.1 or later. I recommend using [Lucee 4.5.0](http://lucee.org/downloads.html) or later since it's free, open source, and fast, with a small footprint. If you're using Adobe ColdFusion, I recommend upgrading to the latest version (ColdFusion 11 as of August 2014) to take advantage of the huge improvements in the core language since ColdFusion 9.0.2 (closures, member functions, full cfscript support, etc) -- although there are quite a few bugs in several areas of ColdFusion 11 (even as of June 2015).

If you're on ColdFusion 9.0.1, FW/1 3.1 should work but there may be some edge cases in `Application.cfc` lifecycle behavior that may trip you up.

If you're on ColdFusion 9.0.0 or earlier, or still using Open BlueDragon, you'll need to stick with FW/1 1.3. Sorry, but supporting those versions is just too painful!

Your First FW/1 Application
---
FW/1 itself consists of a single CFC: `framework.one`, i.e., `framework/one.cfc`. Your `Application.cfc` will extend that and your application's "pages" will live under a `views` folder inside a subfolder for each _section_ of your application.

When you download FW/1 (or check it out from Github), it's a complete web application. **The `framework` folder should either be copied to your webroot (the simplest way to get started) or else made accessible via a mapping for `/framework`.** Since `Application.cfc` extends `framework.one`, you have to add that mapping in your CFML admin - you cannot use a per-application mapping.

_Note: do not install FW/1 into a subfolder that contains . in the name as this may prevent CFC resolution from working!_

The simplest FW/1 application comprises:

* `Application.cfc` which extends `framework.one`
* Empty `index.cfm` (this file is required but the content is ignored)
* `views` folder containing a `main` subfolder containing `default.cfm` - your initial application view

See the `1helloworld` example in the `examples` folder which is this same minimal application.

Create `Application.cfc` containing:

    component extends=framework.one {
    }

Create an empty `index.cfm` file.

Create `views/main/default.cfm` containing:

    <p>Hello FW/1!</p>

When you access the application, it should say `Hello FW/1!`

### Linking Pages Together

FW/1 provides a convenient method for generating links between pages. We're going to use that to link two pages together.

Edit `views/main/default.cfm` and wrap the content in `cfoutput` since we're going to be executing CFML code now:

    <cfoutput>
      <p>Hello FW/1</p>
    </cfoutput>

Now we'll add a link to a new page - a new _section.item_ - using the `buildURL()` function FW/1 provides. Edit `default.cfm` so it looks like this:

    <cfoutput>
      <p>Hello FW/1</p>
      <p><a href="#buildURL('main.other')#">Go away</a>!</p>
    </cfoutput>

This will generate a link with an `action` of `main.other`. Now we'll create `views/main/other.cfm` with this content:

    <cfoutput>
      <p>Goodbye FW/1</p>
      <p><a href="#buildURL('main')#">Come back</a>!</p>
    </cfoutput>

This will generate a link with an `action` of `main`, which is equivalent to `main.default` as we'll see in the next section.

Reload your application in your browser: you should see a `Go away!` link after `Hello FW/1!`. Click it and you should go to a URL with `?action=main.other` which should display `Goodbye FW/1!` and a link back to your default page. Click that `Come back` link and you should see your original page. Note that FW/1 realized that the `main` action is the default and omitted it from the URL completely: this helps keep URLs unique for search engines.

See the `2hellolinked` example in the `examples` folder which is this simple two page application.

### An Aside on URL Structure

Pages are accessed using `?action=section.item` in the URL which will display the `views/section/item.cfm` file. The default action is `main.default`, as you might have guessed from the examples above! If you specify just the section name - `?action=section` then the item has the value `default`, in other words, `?action=section` is equivalent to `?action=section.default`.

If your application server supports it, so-called SES (Search Engine Safe) URLs can be used with FW/1:

* `index.cfm/section` - equivalent to `?action=section`
* `index.cfm/section/item` - equivalent to `?action=section.item`
* `index.cfm/section/item/name/value` - equivalent to `?action=section.item&name=value`

To use _name/value_ pairs in SES URLs, you must specify both the _section_ and _item_ parts of the action. A trailing _name_ with no _value_ is equivalent to _&name=_ in a normal URL.

You can tell FW/1 to generate URLs like this from `buildURL()` through a configuration setting we'll look at later. You can also tell FW/1 to omit `index.cfm/` from the generated URL (although you'll need web server URL rewrites in place to add `index.cfm/` back in so the requests are processed correctly by your CFML application server).

### Adding a Controller to Your Application

When you ask for `action=section.item` FW/1 also looks for `section.cfc` in a `controllers` folder and, if present, invokes the `item()` method on it (and then displays the matching view). We're going to change our view to display a variable and add a controller to set that variable.

Change `views/main/default.cfm` to contain:

    <cfoutput>Hello #rc.name#!</cfoutput>

Add `controllers/main.cfc` with a method, `default()`, that takes a single struct argument called `rc` (for request context) like this:

    component {
        function default( struct rc ) {
            param name="rc.name" default="anonymous";
        }
    }

Note that controller CFC names must be all lowercase.

When you access the application now, it should say `Hello anonymous!` but if you put `?name=Sean` on the URL, it should say `Hello Sean!` The request context passed to the controller method contains all the URL and form variables from the browser and is also made available to the view directly, as `rc`.

See the `3hellocontroller` example in the `examples` folder which is this simple controller example.

Controllers are cached. Add `?reload=true` to the URL to reload your controllers if you make changes.

### Adding a Layout to Your Application

When you ask for `action=section.item` FW/1 looks for `layouts/section/item.cfm` to find a specific layout (it also knows how to look for default layouts for sections and for an application-wide layout - we'll cover that next). The basic view is passed in as a variable called `body`. Let's try this for our default action, `main.default`:

Create `layouts/main/default.cfm` containing:

    <h1>Welcome to FW/1!</h1>
    <cfoutput>#body#</cfoutput>

Layout filenames, like view filenames, must be all lowercase.

When you access the application now, it should have `Welcome to FW/1!` as a heading above the previous output.

FW/1 also looks for a section-specific layout in `layouts/section.cfm` and an application-wide layout in `layouts/default.cfm`. Let's create those just to see them in action!

Create `layouts/main.cfm` containing:

    <cfoutput><div style="border: solid blue 1px;">#body#</div></cfoutput>

When you access the application, you should see a blue box around the output you had before. Note that if you ask for `action=main.other` you'll get the blue box but you won't get `Welcome to FW/1!` because that came from a layout specific to the `main.default` action.

Now create `layouts/default.cfm` containing:

    <cfoutput>
      <div style="border: solid green 2px; padding: 20px;">
        #body#
      </div>
    </cfoutput>

When you access the application now, you should see a green box added around the blue box that was there before.

<div style="border: solid green 2px; padding: 20px;">
  <div style="border: solid blue 1px;">
    <h1>Welcome to FW/1!</h1>
    <p>Hello anonymous!</p>
  </div>
</div>

You can see how the layouts nest (and hopefully your artistic skills with CSS and HTML can produce something that looks a lot nicer than this example!).

The `4hellolayout` example in the `examples` folder corrsponds to what you should have at this point.

### Adding a Service to Your Application

Whilst you can keep adding functionality to your controllers, a well-structured MVC application tries to keep the controllers lightweight and delegate all the business logic to the "Model" of your application. The Model is generally exposed to your controllers through a service layer, including smart objects that represent the problem domain, known as domain objects.

You can manage service CFCs yourself if you want but FW/1 ships with an easy to use "bean factory" that makes it easy to create service CFCs, have them automatically wired into your controllers, and then call their methods directly.

Let's create a greeting service that we can call from our controller. Create a `model` folder, with a `services` subfolder, and inside that we'll put our `greeting.cfc`:

    component {
        function greet( string name ) {
            return "so-called " & name;
        }
    }

FW/1 automatically looks in the `model` folder for CFCs to create and it expects domain objects to be in a `beans` subfolder and treats everything else as singletons, creating only a single unique instance.

Now we need to tell our controller about this service so we will add `accessors=true` to the `component` tag (so that CFML will automatically provide set and get methods for our properties), and we will add a `property` for our greeting service like this:

    component accessors=true {
        property greetingService;
        function default( struct rc ) {
            param name="rc.name" default="anonymous";
        }
    }

Then `Service` suffix is just a convention that FW/1 and DI/1 use so that you don't have to worry about CFCs with the same name in different parts of your application model - the subfolder is used as the suffix.

Now we'll add a call to the `greet()` method of that service:

    component accessors=true {
        property greetingService;
        function default( struct rc ) {
            param name="rc.name" default="anonymous";
            rc.name = variables.greetingService.greet( rc.name );
        }
    }

When you access the application, you'll see it says `Hello so-called anonymous!` so FW/1 has automatically found our service, _injected_ it into our controller, and made it available for us to call in our `default()` method! This is called _Dependency Injection_ and as we mentioned right at the start of this page, you mostly don't need to worry about what it means or how it works: you just declare a `property` for each service you _depend on_ and FW/1 and DI/1 work together to _inject_ it for you.

If you access the application with `?name=Sean` in the URL, it should say `Hello so-called Sean!` so you can see how URL variables are passed into the controller and can be passed as arguments to service calls.

Your application at this point corresponds to the `5helloservice` example in the `examples` folder.

What's Next?
---
Once you've read this Getting Started guide, you'll want to move on to the [Developing Applications Manual](/documentation/3.1/developing-applications.html) and when you need to look things up, use the [Reference Manual](/documentation/3.1/reference-manual.html). You may also want to learn about [Using Subsystems](/documentation/3.1/using-subsystems.html) which allows FW/1 applications to be combined as modules of a larger FW/1 application, or [DI/1](/documentation/3.1/using-di-one.html), a simple, convention-based dependency injection framework.

You probably also want to [join the FW/1 mailing list](http://groups.google.com/group/framework-one/) on Google Groups,
[join the CFML Slack team](http://cfml-slack.herokuapp.com) for the [#fw1 channel on Slack](https://cfml.slack.com/messages/fw1/),
 or [follow the #FW1 twitter stream](http://twitter.com/search/%23FW1); you may also find help and inspiration in the [FW/1 Site Showcase](https://github.com/framework-one/fw1/wiki/FW-1-Site-Showcase), a directory of sites created with FW/1. To read about what's coming in the future, take a look at the [Roadmap](/documentation/3.1/roadmap.html).
