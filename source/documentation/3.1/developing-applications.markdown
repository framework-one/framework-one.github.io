---
layout: page
title: "Developing Applications with FW/1"
date: 2015-07-12 21:40
comments: false
sharing: false
footer: true
---
FW/1 is intended to allow you to quickly build applications with the minimum of overhead and interference from the framework itself. The convention-based approach means that you can quickly put together an outline of your site or application merely by creating folders and files in the `views` folder. As you are ready to start adding business logic to the application, you can add controllers and/or services and domain objects as needed to implement the validation and data processing.

Basic Application Structure
---
FW/1 applications must have an `Application.cfc` that extends `framework.one` and an empty `index.cfm` as well as at least one view (under the `views` folder). Typical applications will also have folders for `controllers`, a `model`, containing subfolders for `services` and `beans`, and `layouts`. The folders may be in the same directory as `Application.cfc` / `index.cfm` or may be in a directory accessible via a mapping (or some other path under the webroot). If the folders are not in the same directory as `Application.cfc` / `index.cfm`, then `variables.framework.base` must be set in `Application.cfc` to identify the location of those folders.

Note: because `Application.cfc` must extend the FW/1 `/framework/one.cfc`, you need a mapping in the CFML administrator. An alternative approach is to simply copy the `framework` folder to your application's web root. This requires no mapping - but means that you have the framework CFCs as web-accessible resources.

The `views` folder contains a subfolder for each section of the site, each section's subfolder containing individual view files (pages or page fragments) used within that section. Note that if your operating system is case-sensitive, all view folders and filenames must be all lowercase.

The `layouts` folder may contain general layouts for each section and/or a default layout for the entire site. The `layouts` folder may also contain subfolders for sections within the site, which in turn contain layouts for specific views. Note that if your operating system is case-sensitive, all layout folders and filenames must be all lowercase.

The `controllers` folder contains a CFC for each section of the site (that needs a controller!). Each CFC contains a method for each requested item in that section (where control logic is needed). Again, controller CFC filenames must be all lowercase if your operating system is case-sensitive.

You would typically also have a `model` folder containing CFCs for your services and your domain objects - the business logic of your application. The convention is to have your domain objects in a `beans` subfolder and all your singleton service CFCs in a `services` subfolder. FW/1 and DI/1 use a convention where you typically reference model instances via a name that is the name of the CFC followed by the singular of the subfolder, e.g., `productService`, `userBean` but that behavior can be configured.

An application may have additional web-accessible assets such as CSS, images and so on.

Views and Layouts
---
Views and layouts are simple CFML pages. Both views and layouts are passed a variable called `rc` which is the request context (containing the URL and form variables merged together). Layouts are also passed a variable called `body` which is the current rendered view. Both views and layouts have direct access to the full FW/1 API (see below).

The general principle behind views and layouts in FW/1 is that each request will yield a unique page that contains a core view, optionally wrapped in a section-specific layout, wrapped in a general layout for the site. In fact, layouts are more flexible than that, allowing for item-specific layouts as well as section-specific layouts. See below for more detail about layouts.

Both views and layouts may invoke other views by name, using the `view()` method in the FW/1 API. For example, the home page of a site might be a portal style view that aggregates the company mission with the latest news. `views/home/default.cfm` might therefore look like this:

    <cfoutput>
      <div>#view('company/mission')#</div>
      <div>#view('news/list')#</div>
    </cfoutput>

This would render the `company.mission` view and the `news.list` view.

Note: The `view()` method behaves like a smart include, automatically handling subsystems and providing a `local` scope that is private to each view, as well as the `rc` request context variable (through which views can communicate, if necessary). No controllers are executed as part of a `view()` call. Additional data may be passed to the `view()` method in an optional second argument, as elements of a struct that is added to the `local` scope.

### Views and Layouts in more depth

As hinted at above, layouts may nest, with a view-specific layout, a section-specific layout and a site-wide layout. When FW/1 is asked for `section.item`, it looks for layouts in the following places:

* `layouts/section/item.cfm` - The view-specific layout
* `layouts/section.cfm` - The section-specific layout
* `layouts/default.cfm` - The site-wide layout

For a given _action_ (_section.item_) up to three layouts may be found and executed, so the view may be wrapped in a view-specific layout, which may be wrapped in a section-specific layout, which may be wrapped in a site-wide layout. To stop the cascade, set `request.layout = false;` in your view-specific (or section-specific) layout. This allows for full control in authoring section and/or page specific layouts which may be very different from your site-wide layout.

By default, FW/1 selects views (and layouts) based on the action initiated, _subsystem:section.item_ but that can be overridden in a controller by calling the `setView()` and `setLayout()` methods to specify a new action to use for the view and layout lookup respectively. This can be useful when several actions need to result in the same view, such as redisplaying a form when errors are present.

For example, in your `product.list` controller method, you could call `setLayout( 'general.list' )` and FW/1 would look for `layouts/general/list.cfm` and `layouts/general.cfm` instead of layouts related to `product.list`. In addition, you can pass `true` as the second argument to `setLayout()` and cascading is automatically disabled (so only the specified layout will be applied).

The view and layout CFML pages are actually executed by being included directly into the framework CFC. That's how the `view()` method is made available to them. In fact, all methods in the framework CFC are directly available inside views and layouts so you can access the bean factory (if present), execute layouts and so on. It also means you need to be a little bit careful about unscoped variables inside views and layouts: a struct called `local` is available for you to use inside views and layouts for temporary data, such as loop variables and so on.

It would be hard to give a comprehensive list of variables available inside a view or layout but here are the important ones:

* `body` - The generated output passed into a layout that the layout should wrap up. Strictly speaking, this is `arguments.body`, the value passed into the `layout()` method.
* `rc` - A shorthand for the request context (the `request.context` struct). If you write to the `rc` struct, layouts will be able to read those values so this can be a useful way to set a page title, for example (set in the view, rendered in the layout where `<title>` appears).
* `local` - An empty struct, created as a local scope for the view or layout.
* `framework` - The FW/1 configuration structure (`variables.framework` in the framework CFC) which includes a number of useful values including `framework.action`, the name of the URL parameter that holds the action (if you're building links, you should use the `buildURL()` API method which knows how to handle subsystems as well as regular section and item values in the action value). You can also write SES URLs without this variable, e.g., _/index.cfm/section/item_ as long as your application server supports such URLs (better to use `buildCustomURL()` for this!).

In addition, FW/1 uses a number of `request` scope variables to pass data between its various methods so it is advisable not to write to the `request` scope inside a view or layout. See the [Reference Manual](/documentation/3.1/reference-manual.html) for complete details of `request` scope variables used by FW/1.

It is strongly recommended to use the `local` struct for any variables you need to create yourself in a view or layout!

If you have data that is needed by all of your views, it may be convenient to set that up in your `setupView()` method in `Application.cfc` - see **Taking Actions on Every Request** below.

### Rendering Data to the Caller

If you want to return plain text, XML, or JSON from a request instead of rendering an HTML view, you can use the `renderData()` API to bypass views and layouts completely and automatically return data rendered as JSON, XML, or plain text to your caller, with the correct content type automatically set. See below for more details.

Designing Controllers
---
Controllers are the pounding heart of an MVC application and FW/1 provides quite a bit of flexibility in this area. The most basic convention is that when FW/1 is asked for `section.item` it will look for `controllers/section.cfc` and attempt to call the `item()` method on it, passing in the request context as a single argument called `rc` and controllers may call into the application model as needed, then render a view.

Controllers are cached in FW/1's application cache so controller methods need to be written with thread safety in mind (i.e., use `var` to declare variables properly!). Any `setXxx()` methods on a controller CFC may be used by FW/1 to autowire beans from the bean factory into the controller when it is created. In addition, if you add `accessors=true` to your controller's `component` tag, you can declare dependencies with the `property` keyword are those will be autowired by FW/1. `property`-based injection is the preferred approach.

In addition, if you need certain actions to take place before all items in a particular section, you can define a `before()` method in your controller and FW/1 will automatically call it for you, before calling the `item()` method. This might be a good place to put a security check, to ensure a user is logged in before they can execute other actions in that section. The variable `request.item` contains the name of the controller method that will be called, in case you need to have exceptions on the security check (such as for a `main.doLogin` action that attempts to log a user in).

Similarly, if you need certain actions to take place after all items in a particular section, you can define an `after()` method in your controller and FW/1 will automatically call it for you, after calling the `item()` method.

Note that your `Application.cfc` is also considered to be a controller and if it defines `before()` and/or `after()` methods, those are called as part of the lifecycle, around any regular controller methods. Unlike other controllers, it does not need an `init()` method and instead of referring to the FW/1 API methods via `variables.fw...` you can just use the API methods directly - unqualified - since `Application.cfc` extends the framework and all those methods are available implicitly.

Here is the full list of methods called automatically when FW/1 is asked for `section.item` :

* `Application.cfc` : `before()`
* `controllers/section.cfc` : `before()`
* `controllers/section.cfc` : `item()`
* `controllers/section.cfc` : `after()`
* `Application.cfc` : `after()`

Methods that do not exist are not called.

### Using onMissingMethod() to Implement Items

FW/1 supports `onMissingMethod()`, i.e., if a desired method is not present but `onMissingMethod()` is defined, FW/1 will call the method anyway. That applies to all three potential controller methods: `before`, `item`, and `after`. That means you must be a little careful if you implement `onMissingMethod()` since it will be called whenever FW/1 needs a method that isn't already defined. If you are going to use `onMissingMethod()`, I would recommend always defining `before()` and `after()` methods, even if they are empty. Calls to `onMissingMethod()` are passed two arguments in `missingMethodArguments`: `rc` and `method` which is the type of the method being invoked (`"before"`, `"item"` - literally, regardless of the actual requested _item_, `"after"`).

### Using onMissingView() to Handle Missing Views

FW/1 provides a default `onMissingView()` method for `Application.cfc` that throws an exception (view not found). This allows you to provide your own handler for when a view is not present for a specific request. Whatever `onMissingView()` returns is used as the core view and, unless layouts are disabled, it will be wrapped in layouts and then displayed to the user. Make sure you return a string value!

Be aware that `onMissingView()` will be called if your application throws an exception and you have not provided a view for the default error handler (`main.error` - if your `defaultSection` is `main`). This can lead to exceptions being masked and instead appearing as if you have a missing view!

### Taking Actions on Every Request

FW/1 provides direct support for handling a specific request's lifecycle based on an action (either supplied explicitly or implicitly) but relies on your `Application.cfc` for general lifecycle events. That's why FW/1 expects you to write per-request logic in `setupRequest()` (or `before()` if you need to interact with `rc`), per-session logic in `setupSession()` and application initialization logic in `setupApplication()`. In addition there is  `setupView()` which is called just before view rendering begins to allow you to set up data for your views that needs to be globally available, but may depend on the results of running controllers or services.

If you have some logic that is meant to be run on every request, the best way is generally to implement `setupRequest()` in your `Application.cfc` and have it queue up the desired controller method by name, like this:

    function setupRequest() {
        controller( 'security.checkAuthorization' );
    }

This queues up a call to that controller at the start of the request processing, calling `before()`, `checkAuthorization()`, and `after()` as appropriate, if those methods are present.

Note that the _request context_ itself is _not available at this point_! `setupRequest()` is to set things up _prior to the request being processed_. If you need access to `rc`, you will want to implement `before()` in your `Application.cfc` which is a regular controller method that is called before any others (including `before()` in other controllers which get queued up):

    function before( struct rc ) {
        // set up your RC values
    }

If you need to perform some actions after controllers and services have completed but before any views are rendered, you can implement `setupView()` in your `Application.cfc` and FW/1 will call it after setting up the view and layout queue but before any rendering takes place:

    function setupView( struct rc ) {
        // pre-rendering logic
    }

You cannot call controllers here - this lifecycle method is intended for common data setup that is needed by most (or all) of your views and layouts. If services from your model have been autowired into `Application.cfc`, you can call those.

Finally, there is a lifecycle method that FW/1 calls at the end of every request - including redirects - where you can implement `setupResponse()` in your `Application.cfc`:

    function setupResponse( struct rc ) {
        // end of request processing
    }

This is called after all views and layouts have been rendered in a regular request or immediately before the redirect actually occurs when `redirect()` has been called. You cannot call controllers here. If services from your model have been autowired into `Application.cfc`, you can call those.

### Short-Circuiting the Controller / Services Lifecycle

If you need to immediately halt execution of a controller and prevent any further controllers or services from being called, use the `abortController()` method. See the [Reference Manual](/documentation/3.1/reference-manual.html) for more details of `abortController()`, in particular how it interacts with exception-handling code in your controllers.

### Controllers for REST APIs

You can return data directly to the caller, bypassing views and layouts, using the `renderData()` function.

    variables.fw.renderData( contentType, resultData );

Calling this function does not exit from your controller, but tells FW/1 that instead of looking for a view to render, the `resultData` value should be converted to the specified `contentType` and that should be the result of the complete HTTP request.

`contentType` may be `"html"`, `"json"`, `"jsonp"`, `"rawjson"`, `"xml"`, or `"text"`. The `Content-Type` HTTP header is automatically set to:

* `text/html; charset=utf-8`
* `application/json; charset=utf-8`
* `application/javascript; charset=utf-8`
* `application/json; charset=utf-8`
* `text/xml; charset=utf-8`
* `text/plain; charset=utf-8`

respectively. For JSON and JSONP, the `resultData` value is converted to
a string by calling `serializeJSON()` (so use RAWJSON if your
`resultData` value is already a valid JSON string); for XML, the
`resultData` value is expected to be either a valid XML string or an XML
object (constructed via CFML's various `xml...()` functions); for plain
text and HTML, the `resultData` value must be a string. _`"html"`, `"jsonp"` and `"rawjson"` are new in 3.1._

For JSONP, you must also specify the `jsonpCallback` argument:

    variables.fw.renderData( contentType, resultData, statusCode, callback );
    // or:
    variables.fw.renderData(
        type = contentType, data = resultData, jsonpCallback = callback
    );

You can also specify an HTTP status code as a third argument. The default is 200.

When you use `renderData()`, no matching view is required for the action being executed.

Designing Services and Domain Objects
---
Services - and domain objects - should encapsulate all of the business logic in your application. Where possible, most of the application logic should be in the domain objects, making them smart objects, and services can take care of orchestrating work that reaches across multiple domain objects.

Controllers should call methods on domain objects and services to do all the heavylifting in your application, passing specific elements of the request context as arguments. FW/1's `populate()` API is designed to allow you to store arbitrary elements of the request context in domain objects.

It is expected that you'll be using a bean factory, your services will be autowired into your controllers, making it easier to call them directly, without having to worry about how and where to construct those CFCs. Using a bean factory means that your domain objects can also be managed, with services autowired into them as necessary, so your controllers can simply ask the bean factory for a new domain object (or ask a service for it), `populate()` it from the request context, call methods on the domain object, or pass them to services as necessary.

Services should not know anything about the framework. Service methods should not "reach out" into the request scope to interact with FW/1 - or any other scopes! - they should simply have some declared arguments, perform some operation and return some data.

### Services, Domain Objects and Persistence

There are many ways to organize how you save and load data. You could use the ORM that comes with ColdFusion, Lucee, or Railo, you could write your own data mapping service, you could write custom SQL for every domain object. Regardless of how you choose to handle your persistence, encapsulating it in a service CFC is probably a good idea. For convenience it is often worth injecting your persistence service into your domain object so you can have a convenient `domainObject.save()` call to use from your controller, even if it just delegates to the persistence service internally:

    component accessors=true{
        property dataService;
        //...
        function save() {
            variables.dataService.save( this );
        }
    }

Using Bean Factories
---
By default, FW/1 will use DI/1 to manage your controllers and your model. You don't have to do anything for that to happen automatically.

_Note: in previous releases of FW/1 (prior to 3.0), you had to create the bean factory yourself and tell FW/1 about it, in `setupApplication()`. If you are migrating a 2.x application that uses a bean factory to 3.0, review the **Bean Factory Configuration** section below to see how to set up your bean factory the same way._

FW/1 & DI/1 expect to find a `controllers` folder and a `model` folder in the base of the application tree. Under DI/1's conventions, `controllers/section.cfc` becomes `sectionController`, `model/beans/foo.cfc` becomes `fooBean`, and `model/services/bar.cfc` becomes `barService`. Other pluralized subfolders under `model` are treated in a similar fashion. By default, everything is treated as a singleton - a single, unique instance, essentially cached in `application` scope - except for CFCs in the `beans` subfolder.

In general, managing dependencies is as simple as adding `accessors=true` to your `component` tag, and declaring dependencies with the `property` keyword, like this:

    component accessors=true {
        property userService;
        property securityService;
        //...
    }

This will make `variables.userService` and `variables.securityService` available, based on `model/services/user.cfc` and `model/services/security.cfc`. You could also use long form CFC names like `userservice.cfc` and `securityservice.cfc` if you wanted. See the next section for more details on configuring DI/1.

If you let FW/1 use DI/1 to automatically manage your beans, and you are using subsystems, FW/1 will also use it to manage your subsystems' beans. See [Using Subsystems](/documentation/3.1/using-subsystems.html) for more details.

### Transients, Bean Factory, Framework

All of the above talks about singletons being injected into your controllers and services. There are three important cases that are not standard singletons:

* transients - your domain objects (in the `beans` subfolder of the `model`), each time you request one of these from the bean factory, you get a fresh instance, fully populated.
* bean factory - sometimes you need access to the bean factory directly (such as for obtaining a transient) and whilst you can get at it inside your controllers via `variables.fw.getBeanFactory()` it's better to have the bean factory injected by declaring `property beanFactory;` (which can be used in both controllers and services), then you can call `variables.beanFactory.getBean()` whenevr you need a transient.
* framework - if you let FW/1 use DI/1 (or AOP/1), it will create an alias of `fw` for the FW/1 instance and if your controllers have a constructor argument that matches, the framework will be passed into `init( fw )`; if you are using a different bean factory, or you don't want to bother with a constructor, you can declare `property framework;` and FW/1 will inject itself into your controller.

### Bean Factory Configuration

Although FW/1 uses DI/1 by default, it also has out-of-the-box support for AOP/1 and WireBox. It can also support other bean factories that follow certain conventions - see **Custom Bean Factory Support** below for more details.

You tell FW/1 which bean factory to use through the `variables.framework.diEngine` configuration variable:

* `"di1"` - the default, use DI/1.
* `"aop1"` - use AOP/1; additional configuration is the same as for DI/1.
* `"wirebox"` - use WireBox, via the supplied adapter.
* `"none"` - do not use a bean factory automatically; you may still create your own bean factory in `setupApplication()` and use `setBeanFactory()` to tell FW/1 about it (this is the easiest way to migrate a 2.x application to 3.0 - but also see below).
* `"custom"` - use a non-standard bean factory - see **Custom Bean Factory Support** below for the requirements on such a bean factory.

Unless you choose `"none"`, FW/1 will use `variables.framework.diComponent` as the dotted-path of a CFC to construct and use as the bean factory. This configuration variable has a default based on `diEngine` as follows:

* `"di1"` - `"framework.ioc"`
* `"aop1"` - `"framework.aop"`
* `"wirebox"` - `"framework.WireBoxAdapter"`

You can override these if you have installed FW/1's `framework` folder contents elsewhere. If you specify a `diEngine` of `"custom"`, you must supply your own value for `diComponent`.

_Note: in order to use AOP/1 you need to manually download it and place it in the `framework` folder yourself._

The following configuration variables are used in the construction of the bean factory component:

* `diLocations` - for `diEngine` `"di1"` or `"aop1"`, this is the first argument to the constructor and represents a list of folders to be scanned; for `diEngine` `"wirebox"`, this is used to set the `scanLocations()` on WireBox's binder; for `diEngine` `"custom"`, this is the first argument to the constructor.
* `diConfig` - for `diEngine` `"di1"` or `"aop1"`, this is the second argument to the constructor and represents configuration settings for DI/1 (or AOP/1); for `diEngine` `"wirebox"`, this is the `properties` argument to the constructor; for `diEngine` `"custom"`, this is the second argument to the constructor.

Here's how those values are used in code to construct the bean factory:

    // di1 / aop1:
    var bf = new "#variables.framework.diComponent#"(
        variables.framework.diLocations,
        variables.framework.diConfig
    );
    // wirebox:
    var bf = new "#variables.framework.diComponent#"(
        properties = variables.framework.diConfig
    );
    bf.getBinder().scanLocations( variables.framework.diLocations );
    // custom:
    var bf = new "#variables.framework.diComponent#"(
        variables.framework.diLocations,
        variables.framework.diConfig
    );

If you are using subsystems and also using DI/1 as your default bean factory component, `diConfig` will be passed to subsystem bean factories when they are constructed. You can override this on a per-subsystem basis by setting `diConfig` in the specific `framework.subsystems` configuration structure. _Per-subsystem `diConfig` is new in 3.1._

#### Migrating 2.x Applications to 3.0

If you migrated through FW/1 2.5 (recommended), you'll have already dealt with the features that were deprecated in 2.5 and are removed in 3.0. The other changes are that `org.corfield.framework` has moved to `framework.one` and both `getRC()` and `getRCValue()` have been removed (these changes were deprecated throughout the 3.0 prerelease cycle and removed in the RC cycle).

The final change that you will run into if you were using a bean factory is how FW/1 3.0 manages this automatically now. The simplest migration is to set `variables.framework.diEngine` to `"none"` and carry on doing what you were doing (manually creating the bean factory).

The recommended migration is to set the `di*` configuration variables appropriately to allow FW/1 to take over and manage your bean factory for you. If you are using DI/1, this is likely to be easier than, say, ColdSpring (see **Custom Bean Factory Support** below). If your DI/1 setup is fairly simple, you won't need to do much. Here are some examples:

    // 2.x setupApplication():
    var bf = new framework.ioc("model,controllers");
    setBeanFactory(bf);
    // 3.0 - no configuration necessary
    // just remove those lines from setupApplication()

    // 2.x setupApplication():
    var bf = new framework.ioc("/model,/app/controllers");
    setBeanFactory(bf);
    // 3.0 - remove those lines and add this config:
    variables.framework = {
        ...
        diLocations = "/model,/app/controllers",
        ...
    };

    // 2.x setupApplication():
    var bf = new path.to.ioc("/model,/app/controllers");
    setBeanFactory(bf);
    // 3.0 - remove those lines and add this config:
    variables.framework = {
        ...
        diComponent = "path.to.ioc",
        diLocations = "/model,/app/controllers",
        ...
    };

    // 2.x setupApplication():
    var bf = new path.to.ioc(
        "/model,/app/controllers",
        { ... config ... }
    );
    setBeanFactory(bf);
    // 3.0 - remove those lines and add this config:
    variables.framework = {
        ...
        diComponent = "path.to.ioc",
        diLocations = "/model,/app/controllers",
        diConfig = { ... config ... },
        ...
    };

If you use a load listener and call `bf.onLoad( myListener )`, you use `diConfig` and add `loadListener = myListener` to it instead.

If you perform more complex configuration of DI/1 (adding bean declarations etc), add a new function to your `Application.cfc` that accepts the bean factory as an argument, and then specify that as the `loadListener`:

    // 2.x setupApplication():
    var bf = new path.to.ioc(
        "/model,/app/controllers",
        { ... config ... }
    );
    bf...( ... );
    ... other bf stuff ...
    setBeanFactory(bf);
    // 3.0 - move the bf configuration to a load listener function:
    function factoryConfig( bf ) {
        bf...( ... );
        ... other bf stuff ...
    }
    // 3.0 - then remove the 2.x code and add this config:
    variables.framework = {
        ...
        diComponent = "path.to.ioc",
        diLocations = "/model,/app/controllers",
        diConfig = { ... config ..., loadListener = factoryConfig },
        ...
    };

If you're still using ColdSpring - which hasn't been updated in many years now! - you'll have to use the `"custom"` `diEngine` and probably create a wrapper CFC if you want FW/1 to manage it for you (but it's probably simpler to set `diEngine` to `"none"` and continue doing what you're doing!).

### Custom Bean Factory Support

FW/1 can support any bean factory that conforms to the following API:

* `boolean containsBean(string name)` - returns true if the factory knows of the named bean
* `any getBean(string name)` - returns a fully initialized bean identified by name

You can either set `diEngine` to `"none"` and do all your configuration manually and then call `setBeanFactory()` in `setupApplication()` or you can configure FW/1 to use your bean factory and manage it for you. You may need to write a wrapper CFC.

Here's an example of writing a wrapper CFC for ColdSpring so you could have FW/1 manage your beans with that. To manually setup ColdSpring as a bean factory, you would need the following in your `setupApplication()`:

    var bf = new coldspring.beans.DefaultXmlBeanFactory();
    bf.loadBeans( expandPath('config/coldspring.xml') );
    setBeanFactory(bf);

The constructor may take two optional arguments (`defaultAttributes` and `defaultProperties`) so we need to cater for those, and the actual bean "location" is specified via a file path. This maps well to `diLocations` for the latter and `diConfig` for the former pair.

    // ioc.ColdSpringAdapter
    component extends=coldspring.beans.DefaultXmlBeanFactory {
        function init( filePath, config ) {
            super.init( argumentCollection = config );
            this.loadBeans( filePath );
            return this;
        }
    }

Now we can configure FW/1 to use this as follows:

    variables.framework = {
        ...
        diEngine = "custom",
        diComponent = "ioc.ColdSpringAdapter",
        diLocations = expandPath('config/coldspring.xml'),
        diConfig = {},
        ...
    };

And we could pass in default properties like this:

    variables.framework = {
        ...
        diEngine = "custom",
        diComponent = "ioc.ColdSpringAdapter",
        diLocations = expandPath('config/coldspring.xml'),
        diConfig = { defaultProperties = { ... } },
        ...
    };

Error Handling
---
By default, if an exception occurs, FW/1 will attempt to run the `main.error` action (as if you had asked for `?action=main.error`), assuming your `defaultSection` is `main`. If you change the `defaultSection`, that implicitly changes the default error handler to be the `error` item in that section. The exception thrown is stored directly in the `request` scope as `request.exception`. If FW/1 was processing an action when the exception occurred, the name of that action is available as `request.failedAction`. The default error handling action can be overridden in your `Application.cfc` by specifying `variables.framework.error` to be the name of the action to invoke when an exception occurs.

If the specified error handler does not exist or another exception occurs during execution of the error handler, FW/1 provides a very basic fallback error handler that simply displays the exception. If you want to change this behavior, you can either override the `fail()` method or the `onError()` method but I don't intend to "support" that so the only documentation will be in the code!

Note: If you override `onMissingView()` and forget to define a view for the error handler, FW/1 will call `onMissingView()` and that will hide the original exception.

Configuring FW/1 Applications
---
All of the configuration for FW/1 is done through a simple structure in `Application.cfc`. The default behavior for the application is as if you specified this structure:

    variables.framework = {
        action = 'action',
        usingSubsystems = false,
        defaultSubsystem = 'home',
        defaultSection = 'main',
        defaultItem = 'default',
        subsystemDelimiter = ':',
        siteWideLayoutSubsystem = 'common',
        subsystems = { },
        home = 'main.default', // defaultSection & '.' & defaultItem
        // or: defaultSubsystem & subsystemDelimiter & defaultSection & '.' & defaultItem
        error = 'main.error', // defaultSection & '.error'
        // or: defaultSubsystem & subsystemDelimiter & defaultSection & '.error'
        reload = 'reload',
        password = 'true',
        reloadApplicationOnEveryRequest = false,
        generateSES = false,
        SESOmitIndex = false,
        // base = omitted so that the framework calculates a sensible default
        baseURL = 'useCgiScriptName',
        // cfcbase = omitted so that the framework calculates a sensible default
        unhandledExtensions = 'cfc',
        unhandledPaths = '/flex2gateway',
        unhandledErrorCaught = false,
        preserveKeyURLKey = 'fw1pk',
        maxNumContextsPreserved = 10,
        cacheFileExists = false,
        applicationKey = 'framework.one',
        trace = false,
        routes = [ ],
        // resourceRouteTemplates - see routes documentation
        routesCaseSensitive = true,
        noLowerCase = false,
        diEngine = "di1",
        diComponent = "framework.ioc",
        diLocations = "model,controllers",
        diConfig = { }
    };

The keys in the structure have the following meanings:

* `action` - The URL or form variable used to specify the desired action (`section.item`).
* `usingSubsystems` - Whether or not to use subsystems - see `Using Subsystems` below. This is automatically set `true` if you set any of the of the subsystem-related configuration.
* `defaultSubsystem` - If subsystems are enabled, this is the default subsystem when no action is specified in the URL or form post.
* `defaultSection` - If subsystems are enabled, this is the default section within a subsystem when either no action is specified at all or just the subsystem is specified in the action. If subsystems are not enabled, this is the default section when no action is specified in the URL or form post.
* `defaultItem` - The default item within a section when either no action is specified at all or just the section is specified in the action.
* `subsystemDelimiter` - When subsystems are enabled, this specifies the delimiter between the subsystem name and the action in a URL or form post.
* `siteWideLayoutSubsystem` - When subsystems are enabled, this specifies the subsystem that is used for the (optional) site-wide default layout.
* `subsystems` - An optional struct of structs containing per-subsystem configuration data. Each key in the top-level struct is named for a subsystem. The contents of the nested structs can be anything you want for your subsystems. Retrieved by calling `getSubsystemConfig()`. Currently the only keys used by FW/1 are `baseURL` and `diConfig` which can be used to configure per-subsystem values.
* `home` - The default action when it is not specified in the URL or form post. By default, this is `defaultSection`.`defaultItem`. If you specify `home`, you are overriding (and hiding) `defaultSection` but not `defaultItem`. If `usingSubsystem` is `true`, the default for `home` is `"home:main.default"`, i.e., `defaultSubsystem & subsystemDelimiter & defaultSection & '.' & defaultItem`.
* `error` - The action to use if an exception occurs. By default this is `defaultSection.error`.
* `reload` - The URL variable used to force FW/1 to reload its application cache and re-execute `setupApplication()`.
* `password` - The value of the reload URL variable that must be specified, e.g., `?reload=true` is the default but you could specify `reload = 'refresh', password = 'fw1'` and then specifying `?refresh=fw1` would cause a reload.
* `reloadApplicationOnEveryRequest` - If this is set to `true` then FW/1 behaves as if you specified the `reload` URL variable on every request, i.e., at the start of each request, the controller/service cache is cleared and `setupApplication()` is executed.
* `generateSES` - If true, causes `redirect()` and `buildURL()` to generate SES-style URLs with items separated by `/` (and the path info in the URL will begin `/section/item` rather than `?action=section.item` - see the [Reference Manual](/documentation/3.1/reference-manual.html) for more details).
* `SESOmitIndex` - If SES URLs are enabled and this is `true`, will attempt to omit the base filename in the path when constructing URLs in `buildURL()` and `redirect()` which will generally omit `/index.cfm` from the start of the URL. Again, see the [Reference Manual](/documentation/3.1/reference-manual.html) for more details.
* `base` - Provide this if the application itself is not in the same directory as `Application.cfc` and `index.cfm`. It should be the **relative** path to the application from the `Application.cfc` file.
* `baseURL` - Normally, `redirect()` and `buildURL()` default to using `CGI.SCRIPT_NAME` as the basis for the URL they construct. This is the right choice for most applications but there are times when the base URL used for your application could be different. You can also specify `baseURL = "useRequestURI"` and instead of `CGI.SCRIPT_NAME`, the result of `getPageContext().getRequest().getRequestURI()` will be used to construct URLs. This is the right choice for FW/1 applications embedded inside Mura.
* `cfcbase` - Provide this if the `controllers` and `model` folders are not in the same folder as the application. It is used as the dotted-path prefix for controller and service CFCs, e.g., if `cfcbase = 'com.myapp'` then a controller would be `com.myapp.controllers.MyController`.
* `unhandledExtensions` - A list of file extensions that FW/1 should not handle. By default, just requests for CFCs, e.g., `some.cfc`, are not handled by FW/1.
* `unhandledPaths` - A list of file paths that FW/1 should not handle. By default, just requests for `/flex2gateway` are not handled by FW/1 (hey, some people are still using Flex - don't judge!). If you specify a directory path, requests for any files in that directory are then not handled by FW/1. For example, `unhandledPaths = '/flex2gateway,/404.cfm,/api'` will cause FW/1 to not handle requests from Flex, requests for the `/404.cfm` page and any requests for files in the `/api` folder.
* `unhandledErrorCaught` - By default the framework does not attempt to catch errors raised by unhandled requests but sometimes when you are migrating from a legacy application it is useful to route error handling of legacy (unhandled) requests through FW/1. The default for this option is `false`. Set it `true` to have FW/1's error handling apply to unhandled requests.
* `preserveKeyURLKey` - In order to support multiple, concurrent flash scope uses - across redirects - for a single user, such as when they have multiple browser windows open, this value is used as a URL key that identifies which flash context should be restored for that browser window. If that doesn't make sense, don't worry about it - it's magic! This value just needs to be something unique that won't clash with any of your own URL variables. This will be ignored if you set `maxNumContextsPreserved` to `1` because with only one context, FW/1 will not use a URL variable to track flash scope across redirects.
* `maxNumContextsPreserved` - If you expect users to have more than 10 browser windows open at the same time, you'll want to set this value higher. I know, Ryan was very thorough when he implemented multiple flash contexts! Setting `maxNumContextsPreserved` to `1` will prevent the URL key from being used for redirects (since FW/1 will not need to track multiple flash contexts).
* `cacheFileExists` - If you are running on a system where disk access is slow - or you simply want to avoid several calls to `fileExists()` during requests for performance - you can set this to true and FW/1 will cache all its calls to `fileExists()`. Be aware that if the result of `fileExists()` is cached and you add a new layout or a new view, it won't be noticed until you reload the framework.
* `applicationKey` - A unique value for each FW/1 application that shares a common ColdFusion application name.
* `noLowerCase` - If `true`, FW/1 will not force actions to lowercase so subsystem, section and item names will be case sensitive (in particular, filenames for controllers, views and layouts may therefore be mixed case on a case-sensitive operating system). The default is `false`. Use of this option is _not_ recommended and is not considered good practice.
* `trace` - If `true`, FW/1 will print out debugging / tracing information at the bottom of each page. This can be very useful for debugging your application! If you want to track framework behavior across redirects, you need to enable session management in your application if you use this feature. (Note that FW/1 will not print out debugging / tracing information when the `renderData()` function is used, unless the content type is `"html"`. You can still access and output debugging / tracing information in such cases by overriding the `setupTraceRender()` function. See the [Reference Manual](/documentation/3.1/reference-manual.html) for more details.).
* `routes` - An array of URL path mappings. This allows you to override the conventional mapping of `/section/item` to controllers.
* `resourceRouteTemplates` - see **URL Routes** below.
* `routesCaseSensitive` - Default `true`. Controls whether route matches are case-sensitive or not. _New in 3.1._
* `diEngine` - the Dependency Injection framework that FW/1 should use.
* `diComponent` - the dotted-path to the CFC used for the bean factory (which has sensible defaults based on `diEngine`).
* `diLocations` - the list of folders to check for CFCs to manage; defaults to `"model,controllers"`.
* `diConfig` - any additional configuration needed for the Dependency Injection engine; defaults to `{ }`.

At runtime, this structure also contains the following key (from release 0.4 onward):

* `version` - The release number (version) of the framework.

This is set automatically by the framework and cannot be overridden (well, it shouldn't be overridden!).

In addition you can override the base directory for the application, which is necessary when the `controllers`, `model`, `views` and `layouts` are not in the same directory as the application's `index.cfm` file. `variables.framework.base` should specify the relative path to the directory containing the `layouts` and `views` folders, either as a mapped path or a webroot-relative path (i.e., it must start with / and expand to a full file system path). If the `controllers` and `model` folders are in that same directory, FW/1 will find them automatically. If you decide to put your `controllers` and `model` folders somewhere else, you can also specify `variables.framework.cfcbase` as a dotted-path to those components, e.g., `com.myapp.cfcs` assuming that `com.myapp.cfcs.controllers.Controller` maps to your `Controller.cfc` and `com.myapp.cfcs.model.services.Service` maps to your `Service.cfc`.

URL Routes
---
In addition to the standard /section/item URLs that FW/1 supports, you can also specify `routes` that are URL patterns, optionally containing variables, that map to standard /section/item URLs.

To use routes, specify `variables.framework.routes` as an array of structures, where each structure specifies mappings from routes to standard URLs. The array is searched in order and the first matching route is the one selected (and any subsequent match is ignored). This allows you to control which route should be used when several possibilities match.

Placeholder variables in the route are identified by a leading colon and can appear in the URL as well, for example `{ "/product/:id" = "/product/view/id/:id" }` specifies a match for `/product/something` which will be treated as if the URL was `/product/view/id/something` - section: `product`, item: `view`, query string `id=something`.

Routes can also be restricted to specific HTTP methods by prefixing them with `$` and the _method_, for example `{ "$POST/search" = "/main/search" }` specifies a match for a `POST` on `/search` which will be treated as if the URL was `/main/search` - section: `main`, item: `search`. A `GET` operation will not match this route.

Routes can also specify a redirect instead of a substitute URL by prefixing the URL with an HTTP status code and a colon, for example `{ "/thankyou" = "302:/main/thankyou" }` specifies a match for `/thankyou` which will cause a redirect to `/main/thankyou`.

A route of `"*"` is a wildcard that will match any request and therefore must be the last route in the array. A wildcard route may be restricted to a specific method, e.g., `"$POST*"` will match a `POST` to any URL.

Route matches are case-sensitive unless you set `routesCaseSensitive` to `false` in the FW/1 configuration.

The keyword `"$RESOURCES"` can be used as a shorthand way of specifying resource routes: `{ "$RESOURCES" = "dogs,cats,hamsters,gerbils" }`. FW/1 will interpret this as if you had specified a standard set of routes for each of the listed resources. For example, for the resource "dogs", FW/1 will parse the following routes:

    { "$GET/dogs/$" = "/dogs/default" },
    { "$GET/dogs/new/$" = "/dogs/new" },
    { "$POST/dogs/$" = "/dogs/create" },
    { "$GET/dogs/:id/$" = "/dogs/show/id/:id" },
    { "$PATCH/dogs/:id/$" = "/dogs/update/id/:id", "$PUT/dogs/:id/$" = "/dogs/update/id/:id" },
    { "$DELETE/dogs/:id/$" = "/dogs/destroy/id/:id" }

There are also some additional resource route settings that can be specified. First you should note that the following three lines are equivalent:

    { "$RESOURCES" = "dogs,cats,hamsters,gerbils" },
    { "$RESOURCES" = [ "dogs","cats","hamsters","gerbils" ] },
    { "$RESOURCES" = { resources = "dogs,cats,hamsters,gerbils" } }

The first two lines are shorthand ways of specifying the full configuration struct given in the third line. An example of a full configuration struct would be the following:

    {
        resources = "dogs",
        methods = "default,create,show",
        pathRoot = "/animals",
        nested = "..."/[...]/{...}
    }

The key `"methods"`, if specified, limits the generated routes to the method names listed.

The key `"pathRoot"`, if specified, is prepended to the generated route paths, so, given the above configuration struct, you get routes such as `{ "$GET/animals/dogs/:id" = "/dogs/show/id/:id" }`.

Alternatively (or in addition), you can specify a subsystem: `subsystem = "animals"`, which generates routes such as `{ "$GET/animals/dogs/:id" = "/animals:dogs/show/id/:id" }`.

The key `"nested"` is used to indicate resources which should be nested under another resource, and again can be specified as a string list, an array, or a struct. For example: `{ "$RESOURCES" = { resources = "posts", nested = "comments" } }` results in all of the standard routes for `"posts"`, and in addition generates nested routes for `"comments"` such as `{ "$GET/posts/:posts_id/comments" = "/comments/default/posts_id/:posts_id" }`. Here it should be noted that the convention is to map the parent resource key to the variable name `"#resource#_id"`. Also, you cannot specify a path root or subsystem for a nested resource as it inherits these from its parent resource.

The specific routes that FW/1 generates are determined by the `variables.framework.resourceRouteTemplates` array. By default it looks like the following:

     variables.framework.resourceRouteTemplates = [
         { method = 'default', httpMethods = [ '$GET' ] },
         { method = 'new', httpMethods = [ '$GET' ], routeSuffix = '/new' },
         { method = 'create', httpMethods = [ '$POST' ] },
         { method = 'show', httpMethods = [ '$GET' ], includeId = true },
         { method = 'update', httpMethods = [ '$PUT','$PATCH' ], includeId = true },
         { method = 'destroy', httpMethods = [ '$DELETE' ], includeId = true }
    ];

If you wish to change the controller methods the routes are mapped to, for instance, you can specify this array in your `Application.cfc` and then change the default method names. For example, if you want `"$GET/dogs/$"` to map to `"/dogs/index"`, you would change `method = 'default'` to `method = 'index'` in the first template struct.

A route structure may also have documentation by specifying a hint: `{ "/product/:id" = "/product/view/id/:id", hint = "Display a product" }`.

Here's an example showing all the features together:

    variables.framework.routes = [
        { "/product/:id" = "/product/view/id/:id", "/user/:id" = "/user/view/id/:id",
          hint = "Display a specific product or user" },
        { "/products" = "/product/list", "/users" = "/user/list" },
        { "/old/url" = "302:/new/url" },
        { "$GET/login" = "/not/authorized", "$POST/login" = "/auth/login" },
        { "$RESOURCES" = { resources = "posts", subsystem = "blog", nested = "comments,tags" } },
        { "*" = "/not/found" }
    ];

Environment Control
---
FW/1 supports _environment control - the ability to automatically detect your application environment (development, production, etc) and adjust the framework configuration accordingly. There are three components to environment control:
* `variables.framework.environments` - An optional structure containing groups of framework options for each environment.
* `getEnvironment()` - A function that you override in `Application.cfc` that returns a string indicating the application environment.
* `setupEnvironment( string env )` - A function that may optionally override in `Application.cfc` to provide more programmatic configuration for your application environment.

Environment control is based on the concept of _tier_ - development, staging, production etc - and optionally a _server_ specifier. This two-part string determines how elements of `variables.framework.environments` are selected and merged into the base framework configuration. A string of the format _`"tier"`_ or _`"tier-server"`_ should be returned from `getEnvironment()`. FW/1 first looks for a group of options matching just _tier_ and, if found, appends those to the base configuration. FW/1 then looks for a group of options matching _tier-server_ and, if found, appends those to the configuration. After merging configuration options, FW/1 calls `setupEnvironment()` passing the tier/server string so your application may perform additional customization. This process is executed on every request (so be aware of performance considerations) which allows a single application to serve multiple domains and behave accordingly for each domain, for example.

Note that for the most part, the _tier_ or _tier-server_ configuration will override any default configuration in the `variables.framework` structure. There are two exceptions to that basic rule:

* `diConfig` is recursively merged: the `constants` and `singulars` structures are key-merged, the `exclude` and `transients` arrays are appended. _tier-server_ takes precedence in the key-merging, then _tier_, then the default configuration.
* `subsystems` is key-merged across the _tier_ and _tier-server_ data (with _tier-server_ taking precedence, then _tier_, then then default).

This gives the most intuitive behavior for DI/1 bean factory configuration. It also provides a convenient behavior for environment-based subsystem configuration. Be cautious when specifying subsystem-specific DI/1 configuration since that does not merge across environments. _Note that environment merging is new in 3.1._

Your `getEnvironment()` function can use any means to determine which environment is active. Common methods are examining `CGI.SERVER_NAME` or using the server's actual hostname (accessible thru the new `getHostname()` API method). Here's an example setup:

    public function getEnvironment() {
        if ( findNoCase( "www", CGI.SERVER_NAME ) ) return "prod";
        if ( findNoCase( "dev", CGI.SERVER_NAME ) ) return "dev";
        else return "dev-qa";
    }

    variables.framework.environments = {
        dev = { reloadApplicationOnEveryRequest = true, error = "main.detailederror" },
        dev-qa = { reloadApplicationOnEveryRequest = false },
        prod = { password = "supersecret" }
    };

With this setup, if the URL contains `www`, e.g., `www.company.com`, the tier will be production (`"prod"`) and the reload password will be changed to `"supersecret"`. If the URL contains `dev`, e.g., `dev.company.com`, the tier will be development (`"dev"`) and the application will reload on every request. In addition, a detailed error page will be used instead of the default. Otherwise, the tier will still be development (`"dev"`) but the environment will be treated as a QA server: the development error setting will still be in effect but the framework will no longer reload on every request.

Here is another example:

    public function getEnvironment() {
        var hostname = getHostname();
        if ( findNoCase( "local", hostname ) ) return "dev-" & listFirst( hostname, "-" );
        var svrname = listFirst( hostname, "." ); // drop domain name etc
        switch ( svrname ) {
            case "proserver14a": return "prod-1";
            case "proserver15c": return "prod-2";
            case "proserver22x": return "prod-3";
            case "proserver03b": return "qa";
            default: return "dev-unknown";
        }
    }

This maps local hostnames to specific developers machines (e.g., my development machine is called `Sean-Corfields-iMac.local` and my team's machines follow similar naming). It specifically identifies the three servers in the production cluster and the QA server. All other environments are treated as development with an unknown server specifier.

If you want to ensure that all environments are known configurations, your `setupEnvironment()` function can halt the application if a default environment is detected.

Setting up application, session and request variables
---
The easiest way to setup `application` variables is to define a `setupApplication()` method in your `Application.cfc` and put the initialization in there. This method is automatically called by FW/1 when the application starts up and when the FW/1 application is reloaded.

The easiest way to setup `session` variables is to define a `setupSession()` method in your `Application.cfc` and put the initialization in there. This method is automatically called by FW/1 as part of `onSessionStart()`.

The easiest way to setup `request` variables (or even global variables scope) is to define a `setupRequest()` method in your `Application.cfc` and put the initialization in there. Note that if you set `variables` scope data, it will be accessible inside your views and layouts but not inside your controllers or services.

Using Subsystems
===
The subsystems feature allows you to combine existing FW/1 applications as modules of a larger FW/1 application. The subsystems feature was contributed by Ryan Cogswell and the documentation was written by Dutch Rapley. Read about [Using Subsystems](/documentation/3.1/using-subsystems.html) to combine FW/1 applications.

Accessing the FW/1 API
===
FW/1 uses the `request` scope for some of its temporary data so that it can communicate between `Application.cfc` lifecycle methods without relying on `variables` scope (and potentially interfering with user data in variables scope). The [Reference Manual](/documentation/3.1/reference-manual.html) specifies which request scope variables are used and what you may and may not do with them.

In addition, the API of FW/1 is exposed to controllers, views and layouts in a particular way as documented below.

Controllers and the FW/1 API
---
Each controller method is passed the request context as a single argument called `rc`, of type `struct`. If access to the FW/1 API is required inside a controller, you can define an `init()` method (constructor) which takes a single argument, of type `any`, and when FW/1 creates the controller CFC, it passes itself in as the argument to `init()`. Your `init()` method should save that argument in the variables scope for use within the controller methods:

    function init(fw) {
        variables.framework = fw;
        return this;
    }

    function setBeanFactory( beanFactory ) {
        variables.beanFactory = beanFactory;
    }

Alternatively, you can declare a dependency on the `framework` like this:

    property beanFactory;
    property framework;

Then you could call any framework method:

     var user = variables.beanFactory.getBean("user");
     variables.framework.populate(user);

This will call `setXxx()` methods on the user bean, passing in matching elements from the request context. An optional second argument may be provided that specifies the keys to populate (the default is to attempt to match against every `setXxx()` method on the bean):

    variables.framework.populate(user,'firstName, lastName, email');

This will call `setFirstName()`, `setLastName()` and `setEmail()` on the user bean, passing in matching elements from the request context.

The other framework methods that are useful for controllers are:

    variables.framework.redirect( action, preserve, append, path, queryString );

where `action` is the action to redirect to, `preserve` is a list of request context keys that should be preserved across the redirect (using `session` scope) and `append` is a list of request context keys that should be appended to the redirect URL. `preserve` and `append` can both be omitted and default to none, i.e., no values preserved or appended. The optional `path` argument allows you to force a new base URL to be used (instead of the default `variables.framework.baseURL` which is normally `CGI.SCRIPT_NAME`). `queryString` allows you to specify additional URL parameters and/or anchors to be added to the generated URL. See the [Reference Manual](/documentation/3.1/reference-manual.html) for more details.

I cannot imagine other FW/1 API methods being called from controllers at this point but the option is there if you need it.

Views/Layouts and the FW/1 API
---
As indicated above under the "in depth" paragraph about views and layouts, the entire FW/1 API is available to views and layouts directly (effectively in the `variables` scope) because of the way views and layouts are executed. This allows views and layouts to access utility beans from the bean factory, such as formatting services, as well as render views and, if necessary, other layouts. Views and layouts also have access to the `framework` structure which contains the `action` key - which could be used for building links:

    <a href="?#framework.action#=section.item">Go to section.item</a>

But you're better off using the *buildURL()* API method:

    <a href="#buildURL( 'section.item' )#">Go to section.item</a>

You can provide additional query string values to `buildURL()`:

    <a href="#buildURL( 'section.item?arg=val' )#">Go to section.item with arg=val in URL</a>
    <a href="#buildURL( action = 'section.item', queryString = 'arg=val' )#">Go to section.item with arg=val in URL</a>

I cannot imagine a view or layout needing full access to the FW/1 API methods beyond `view()`, `layout()` and `getBeanFactory()` but the option is there if you need it.

Convenience Methods in the FW/1 API
---
FW/1 provides a number of convenience methods for manipulating the action value to extract parts of the action (the `action` argument is optional in all these methods and defaults to the currently requested action):

* `getSubsystem( action )` - If subsystems are enabled, this returns either the _subsystem_ portion of the action or the default subsystem. If subsystems are not enabled, returns an empty string.
* `getSection( action )` - Returns the _section_ portion of the action. If subsystems are enabled but no section is specified, returns the default section.
* `getItem( action )` - Returns the _item_ portion of the action. If no item is specified, returns the default item.
* `getSectionAndItem( action )` - Returns the _section.item_ portion of the action, including default values if either part is not specified.
* `getFullyQualifiedAction( action )` - If subsystems are enabled, returns the fully qualified _subsystem:section.item_ version of the action, including defaults where appropriate. If subsystems are not enabled, returns `getSectionAndItem( action )`.
