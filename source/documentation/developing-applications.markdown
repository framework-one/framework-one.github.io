---
layout: page
title: "Developing Applications with FW/1"
date: 2016-09-16 20:00
comments: false
sharing: false
footer: true
---
FW/1 is intended to allow you to quickly build applications with the minimum of overhead and interference from the framework itself. The convention-based approach means that you can quickly put together an outline of your site or application merely by creating folders and files in the `views` folder. As you are ready to start adding business logic to the application, you can add controllers and/or services and domain objects as needed to implement the validation and data processing.

* Table of Contents
{:toc}

Basic Application Structure
---
FW/1 applications generally have an `Application.cfc` that extends `framework.one` and an empty `index.cfm` as well as at least one view (under the `views` folder). Typical applications will also have folders for `controllers`, `layouts` and a `model` -- itself containing subfolders for `services` and `beans`. Some applications may also have a `subsystems` folder (see below). The folders may be in the same directory as `Application.cfc` / `index.cfm` or may be in a directory accessible via a mapping (or some other path under the webroot). If the folders are not in the same directory as `Application.cfc` / `index.cfm`, then `variables.framework.base` must be set in `Application.cfc` to identify the location of those folders (and you will need to use mapped paths for `diLocations`).

As of release 3.5, these folder name conventions can be modified. See **[Configuring FW/1 Applications](#configuring-fw1-applications)** below.

Note: because `Application.cfc` generally extends the FW/1 `/framework/one.cfc`, you need a mapping in the CFML administrator. An alternative approach is to simply copy the `framework` folder to your application's web root. This requires no mapping - but means that you have the framework CFCs as web-accessible resources. See **[Alternative Application Structure](#alternative-application-structure)** below for another option.

The `views` folder contains a subfolder for each section of the site, each section's subfolder containing individual view files (pages or page fragments) used within that section. Note that if your operating system is case-sensitive, all view folders and filenames must be all lowercase.

The `layouts` folder may contain general layouts for each section and/or a default layout for the entire site. The `layouts` folder may also contain subfolders for sections within the site, which in turn contain layouts for specific views. Note that if your operating system is case-sensitive, all layout folders and filenames must be all lowercase.

The `controllers` folder contains a CFC for each section of the site (that needs a controller!). Each CFC contains a method for each requested item in that section (where control logic is needed). Again, controller CFC filenames must be all lowercase if your operating system is case-sensitive.

You would typically also have a `model` folder containing CFCs for your services and your domain objects - the business logic of your application. The convention is to have your domain objects in a `beans` subfolder and all your singleton service CFCs in a `services` subfolder. FW/1 and DI/1 use a convention where you typically reference model instances via a name that is the name of the CFC followed by the singular of the subfolder, e.g., `productService`, `userBean` but that behavior can be configured.

Larger applications may also have a `subsystems` folder containing modules that are themselves "mini FW/1 applications". Each module is a subfolder of `subsystems` and may contain its own `controllers`, `layouts`, `views`, and even a `model` containing module-specific `services` and `beans`. _Note: in earlier versions of FW/1, you needed to explicitly indicate you wanted to use subsystems and the "main application" also had to be a subsystem -- see [Using Subsystems](using-subsystems.html) for more details._

An application may have additional web-accessible assets such as CSS, images and so on. Those can be organized however you prefer as they are outside FW/1's purview.

### Alternative Application Structure

Instead of having your `Application.cfc` extend `framework.one`, you can use `/framework/Application.cfc` as a template for your `Application.cfc`. It creates the FW/1 instance explicitly on each request and delegates the various lifecycle methods to FW/1. The framework configuration structure must be passed to the FW/1 constructor, instead of being set in `variables` scope. This allows you to use a per-application mapping for FW/1:

    // Application.cfc
    component { // does not extend anything
        this.name = 'MyWonderfulApp';
        this.mappings[ '/framework' ] = expandPath( '/libs/fw1/framework' );
        function _get_framework_one() {
            if ( !structKeyExists( request, '_framework_one' ) ) {
                // create your FW/1 application:
                request._framework_one = new framework.one( {
                    trace : true
                } );
            }
            return request._framework_one;
        }

        // lifecycle methods, per /framework/Application.cfc
        ...
    }

This works well when you cannot set a mapping in the CFML admin and you don't need to override any of FW/1's behavior. If you do need to override any of those extension points, you can create an intermediate CFC that extends `framework.one` and put the methods in there, and then create an instance of that in your `Application.cfc`. Use `/framework/MyApplication.cfc` as a template for this:

    // MyApplication.cfc (in webroot, next to Application.cfc)
    component extends=framework.one {
        function setupRequest() {
            controller( 'security.checkAuthorization' );
        }
    }

    // Application.cfc
    component { // does not extend anything
        this.name = 'MyWonderfulApp';
        this.mappings[ '/framework' ] = expandPath( '/libs/fw1/framework' );
        function _get_framework_one() {
            if ( !structKeyExists( request, '_framework_one' ) ) {
                // create my extended version of FW/1:
                request._framework_one = new MyApplication( {
                    trace : true
                } );
            }
            return request._framework_one;
        }

        // lifecycle methods, per /framework/Application.cfc
        ...
    }

Note that you can put your version of `MyApplication.cfc` anywhere and call it anything you want, as long as CFML can create an instance of it. A good strategy is to create a per-application mapping for your application's base folder and use that:

    // /path/to/app/CustomApp.cfc
    component extends=framework.one {
        function setupRequest() {
            controller( 'security.checkAuthorization' );
        }
    }

    // Application.cfc
    component { // does not extend anything
        this.name = 'MyWonderfulApp';
        this.mappings[ '/app' ] = expandPath( '/path/to/app' );
        this.mappings[ '/framework' ] = expandPath( '/libs/fw1/framework' );
        function _get_framework_one() {
            if ( !structKeyExists( request, '_framework_one' ) ) {
                // create my extended version of FW/1:
                request._framework_one = new app.CustomApp( {
                    base : '/app',
                    diLocations = '/app/model, /app/controllers',
                    trace : true
                } );
            }
            return request._framework_one;
        }

        // lifecycle methods, per /framework/Application.cfc
        ...
    }

This documentation assumes the **Basic Application Structure** but some of the examples provided in the FW/1 download use the **Alternative Application Structure** to show you how.

Views and Layouts
---
Views and layouts are simple CFML pages. Both views and layouts are passed a variable called `rc` which is the request context (containing the URL and form variables merged together). Layouts are also passed a variable called `body` which is the current rendered view. Both views and layouts have direct access to the full FW/1 API (see below).

The general principle behind views and layouts in FW/1 is that each request will yield a unique page that contains a core view, optionally wrapped in a section-specific layout, wrapped in a general layout for the site. In fact, layouts are more flexible than that, allowing for item-specific layouts as well as section-specific layouts. See below for more detail about layouts.

Both views and layouts may invoke other views by name, using the `view()` method in the FW/1 API. For example, the home page of a site might be a portal style view that aggregates the company mission with the latest news. `views/home/default.cfm` might therefore look like this:

    <cfoutput>
      <div>#view('company/section/mission')#</div>
      <div>#view('news/list')#</div>
    </cfoutput>

This would render the `views/company/section/mission.cfm` view and the `views/news/list.cfm` view. The latter is as if you had invoked just the view portion of the `news.list` action, but `view()` is generic and intended for use with both full views for actions as well as view fragments that are often kept in subfolders (so they cannot accidentally be run by using just an action in a URL).

Note: The `view()` method behaves like a smart include, automatically handling subsystems and providing a `local` scope that is private to each view, as well as the `rc` request context variable (through which views can communicate, if necessary). No controllers are executed as part of a `view()` call. Additional data may be passed to the `view()` method in an optional second argument, as elements of a struct that is added to the `local` scope.

### Views and Layouts in more depth

As hinted at above, layouts may nest, with a view-specific layout, a section-specific layout and a site-wide layout. When FW/1 is asked for `section.item`, it looks for layouts in the following places:

* `layouts/section/item.cfm` - The view-specific layout
* `layouts/section.cfm` - The section-specific layout
* `layouts/default.cfm` - The site-wide layout

For a given _action_ (_section.item_) up to three layouts may be found and executed, so the view may be wrapped in a view-specific layout, which may be wrapped in a section-specific layout, which may be wrapped in a site-wide layout. To stop the cascade, call `disableLayout()` in your view-specific (or section-specific) layout. This allows for full control in authoring section and/or page specific layouts which may be very different from your site-wide layout. You may also call `disableLayout()` in a controller to turn off all layouts for a request.

When FW/1 is asked for `module:section.item`, it looks for layouts in the following places:

* `subsystems/module/layouts/section/item.cfm` - The view-specific layout
* `subsystems/module/layouts/section.cfm` - The section-specific layout
* `subsystems/module/layouts/default.cfm` - The subsystem-wide layout
* `layouts/default.cfm` - The site-wide layout

For a given _action_ (_module:section.item_) up to four layouts may be found and executed, so the view may be wrapped in a view-specific layout, which may be wrapped in a section-specific layout, which may be wrapped in a subsystem-wide layout, which may be wrapped in a site-wide layout.

By default, FW/1 selects views (and layouts) based on the action initiated, _module:section.item_ but that can be overridden in a controller by calling the `setView()` and `setLayout()` methods to specify a new action to use for the view and layout lookup respectively. This can be useful when several actions need to result in the same view, such as redisplaying a form when errors are present.

For example, in your `product.list` controller method, you could call `setLayout( 'general.list' )` and FW/1 would look for `layouts/general/list.cfm` and `layouts/general.cfm` instead of layouts related to `product.list`. In addition, you can pass `true` as the second argument to `setLayout()` and cascading is automatically disabled (so only the specified layout will be applied).

The view and layout CFML pages are actually executed by being included directly into the framework CFC. That's how the `view()` method is made available to them. In fact, all methods in the framework CFC are directly available inside views and layouts so you can access the bean factory (if present), execute layouts and so on. It also means you need to be a little bit careful about unscoped variables inside views and layouts: a struct called `local` is available for you to use inside views and layouts for temporary data, such as loop variables and so on.

It would be hard to give a comprehensive list of variables available inside a view or layout but here are the important ones:

* `body` - The generated output passed into a layout that the layout should wrap up. Strictly speaking, this is `arguments.body`, the value passed into the `layout()` method.
* `rc` - A shorthand for the request context (the `request.context` struct). If you write to the `rc` struct, layouts will be able to read those values so this can be a useful way to set a page title, for example (set in the view, rendered in the layout where `<title>` appears).
* `local` - An empty struct, created as a local scope for the view or layout.
* `framework` - The FW/1 configuration structure (`variables.framework` in the framework CFC) which includes a number of useful values including `framework.action`, the name of the URL parameter that holds the action (if you're building links, you should use the `buildURL()` API method which knows how to handle subsystems as well as regular section and item values in the action value). You can also write SES URLs without this variable, e.g., _/index.cfm/section/item_ as long as your application server supports such URLs (better to use `buildCustomURL()` for this!).

In addition, FW/1 uses a number of `request` scope variables to pass data between its various methods so it is advisable not to write to the `request` scope inside a view or layout. See the [Reference Manual](reference-manual.html#request-scope) for complete details of `request` scope variables used by FW/1.

It is strongly recommended to use the `local` struct for any variables you need to create yourself in a view or layout!

If you have data that is needed by all of your views, it may be convenient to set that up in your `setupView()` method in `Application.cfc` - see **[Taking Actions on Every Request](#taking-actions-on-every-request)** below.

As of release 3.5, FW/1 also recognizes view and layout files with `.lc` and `.lucee` file extensions to support Lucee's new dialect.

### Rendering Data to the Caller

If you want to return plain text, XML, or JSON from a request instead of rendering an HTML view, you can use the `renderData()` API to bypass views and layouts completely and automatically return data rendered as JSON, XML, or plain text to your caller, with the correct content type automatically set. See **[Controllers for REST APIs](#controllers-for-rest-apis)** below for more details.

Designing Controllers
---
Controllers are the pounding heart of an MVC application and FW/1 provides quite a bit of flexibility in this area. The most basic convention is that when FW/1 is asked for `section.item` it will look for `controllers/section.cfc` and attempt to call the `item()` method on it, passing in the request context as an argument called `rc` and the HTTP headers as an argument called `headers` (_new in 4.0_). Controllers may call into the application model as needed, then render a view. Similarly, when asked for `module:section.item` it will look for `subsystems/module/controllers/section.cfc` and attempt to call the `item()` method on it.

Most of the time, you will see controller methods defined like this:

    function item( rc ) {
        ...
    }

A controller that wants to inspect the HTTP headers can be defined like this:

    function item( rc, headers ) {
        ...
    }

The `headers` argument is a struct of HTTP headers, and is the same as you would get from calling `getHttpRequestData().headers`. This argument was introduced in release 4.0 to better support REST APIs.

Controllers are cached in FW/1's application cache so controller methods need to be written with thread safety in mind (i.e., use `var` to declare variables properly!). Any `setXxx()` methods on a controller CFC may be used by FW/1 to autowire beans from the bean factory into the controller when it is created. In addition, if you add `accessors=true` to your controller's `component` tag, you can declare dependencies with the `property` keyword and those will be autowired by FW/1. Do not specify a type or a default on dependency declarations. `property`-based injection is the preferred approach.

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

_Note: if you are using the **Alternative Application Structure** then `before()` and `after()` would be defined in your version of `MyApplication.cfc` which extends the framework, rather than in the actual `Application.cfc` file._

### Using onMissingMethod() to Implement Items

FW/1 supports `onMissingMethod()`, i.e., if a desired method is not present but `onMissingMethod()` is defined, FW/1 will call the method anyway. That applies to all three potential controller methods: `before`, `item`, and `after`. That means you must be a little careful if you implement `onMissingMethod()` since it will be called whenever FW/1 needs a method that isn't already defined. Calls to `onMissingMethod()` are passed three arguments in `missingMethodArguments`: `rc`, `headers` (_new in 4.0_) and `method` which is the type of the method being invoked (`"before"`, `"item"` - literally, regardless of the actual requested _item_, `"after"`). If you are going to use `onMissingMethod()`, you should either check `missingMethodArguments.method` or define `before()` and `after()` methods explicitly, even if they are empty.

### Using onMissingView() to Handle Missing Views

FW/1 provides a default `onMissingView()` method that throws an exception (view not found). This allows you to provide your own handler for when a view is not present for a specific request. Whatever `onMissingView()` returns is used as the core view and, unless layouts are disabled, it will be wrapped in layouts and then displayed to the user. Make sure you return a string value! Calls to `onMissingView()` are passed the `rc` so you can look at `rc.action` to see which action failed to find a view and `request.missingView` if you need to know the specific view that was not found (see [Request Scope](reference-manual.html#request-scope) in the Reference Manual for more details).

Be aware that `onMissingView()` will be called if your application throws an exception and you have not provided a view for the default error handler (`main.error` - if your `defaultSection` is `main`). This can lead to exceptions being masked and instead appearing as if you have a missing view!

### Taking Actions on Every Request

FW/1 provides direct support for handling a specific request's lifecycle based on an action (either supplied explicitly or implicitly) but relies on your `Application.cfc` for general lifecycle events. That's why FW/1 expects you to write per-request logic in `setupRequest()` (or `before()` if you need to interact with `rc`), per-session logic in `setupSession()` and application initialization logic in `setupApplication()`. In addition there is  `setupView()` which is called just before view rendering begins to allow you to set up data for your views that needs to be globally available, but may depend on the results of running controllers or services.

_Note: if you are using the **Alternative Application Structure**, these methods will be in your equivalent of `MyApplication.cfc`, not the actual `Application.cfc` file._

If you have some logic that is meant to be run on every request, that does not need to reference the request context, the best way is generally to implement `setupRequest()` in your `Application.cfc` and have it queue up the desired controller method by name, like this:

    function setupRequest() {
        controller( 'security.checkAuthorization' );
    }

This queues up a call to that controller at the start of the request processing, calling `before()`, `checkAuthorization()`, and `after()` as appropriate, if those methods are present in `controllers/security.cfc`.

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

If you need to immediately halt execution of a controller and prevent any further controllers or services from being called, use the `abortController()` method. See the [Reference Manual](reference-manual.html#public-void-function--abortcontroller) for more details of `abortController()`, in particular how it interacts with exception-handling code in your controllers.

### Controllers for REST APIs

You can return data directly to the caller, bypassing views and layouts, using the `renderData()` function.

    variables.fw.renderData().data( resultData ).type( contentType );

Calling this function does not exit from your controller, but tells FW/1 that instead of looking for a view to render, the `resultData` value should be converted to the specified `contentType` and that should be the result of the complete HTTP request (or the `contentType` may instead be a custom renderer -- see below).

`contentType` may be `"html"`, `"json"`, `"jsonp"`, `"rawjson"`, `"xml"`, or `"text"` (or a function / closure -- see below). The `Content-Type` HTTP header is automatically set to:

* `text/html; charset=utf-8`
* `application/json; charset=utf-8`
* `application/javascript; charset=utf-8`
* `application/json; charset=utf-8`
* `text/xml; charset=utf-8`
* `text/plain; charset=utf-8`

respectively. For JSON and JSONP, the `resultData` value is converted to
a string by calling `serializeJSON()` (so use `"rawjson"` if your
`resultData` value is already a valid JSON string); for XML, the
`resultData` value is expected to be either a valid XML string or an XML
object (constructed via CFML's various `xml...()` functions); for plain
text and HTML, the `resultData` value must be a string. _`"html"`, `"jsonp"` and `"rawjson"` were added in 3.1._

For JSONP, you must also specify the `jsonpCallback` argument:

    variables.fw.renderData().data( resultData ).type( contentType ).jsonpCallback( callback );

You can also specify an HTTP status code. The default is 200:

    variables.fw.renderData().data( resultData ).type( contentType ).statusCode( 403 );

When you use `renderData()`, no matching view is required for the action being executed.

As of release 4.0, you can use the new "builder syntax" shown above for all arguments to `renderData()` -- and the inline argument calls (FW/1 3.5 and earlier) should be considered deprecated, although only the `statusCode` and `jsonpCallback` arguments will trigger warnings to the console in 4.0. In a future release, these will require a framework setting in order to be used and the `type` and `data` arguments will cause deprecation warnings.

The builder syntax supports:

* `data()` to set the data payload to be rendered
* `type()` to set the content type
* `header()` to add an HTTP response header (this is an new feature in release 4.0)
* `statusCode()` to set the HTTP status code
* `statusText()` to set the HTTP status message (this is a new feature in release 4.0)
* `jsonpCallback()` to set the JSONP callback

Once you have called `renderData()`, you can either chain builder calls onto that call to set these values, or you can call the `renderer()` function (_new in 4.0_) to get the builder to set these values:

    variables.fw.renderData().data( result ).type( "json" );
    if ( someCondition ) {
        variables.fw.renderer().header( "X-Result", "Condition Happened" );
    }

As of release 4.0, FW/1 can accept JSON data or URL-encoded data in the body of a POST or PUT. To enable this, set `decodeRequestBody` to `true` in your framework configuration. FW/1 assumes the JSON data, or URL-encoded data, will decode to a struct, and that will be appended to the request context, overriding any URL variables of the same name as elements of the decoded struct.

#### Custom Data Rendering

In addition to the string values for the `contentType`, you may specify a function or closure that behaves as follows:

* It accepts a struct as an argument, containing all the values set by the builder syntax (`data`, `type`, `statusCode`, `statusText`, `jsonpCallback`, as appropriate).
* It returns a struct containing `contentType`, `output`, and optionally a `writer` key.
* It returns the desired value of the `Content-Type` HTTP header as the `contentType` key.
* It renders `resultData` however you wish and returns that as the `output` key.
* If the `content` needs to be delivered to the browser using something more sophisticated than `writeOutput()`, the `writer` key should specify a function or closure to handle that.

The optional `writer` function (or closure) is called as follows:

* It is passed the `output` value from the returned struct.
* It is called instead of calling `writeOutput()`, so FW/1 expects it to perform whatever content delivery is needed (setting additional headers, encoding and writing the response body, etc).

Internally, the standard six content types are implemented as rendering functions in `one.cfc` (as `render_{type}(struct renderData)`). For example, `render_json()` looks like this:

    function render_json( struct renderData ) {
        return {
            contentType = 'application/json; charset=utf-8',
            output = serializeJSON( renderData.data )
        };
    }

Thus you also have the option of overriding one of the standard rendering types by defining your own version of the function in `Application.cfc` (or the application CFC that extends `framework.one` if you are using the **Alternative Application Structure**). Similarly, rather than pass a function directly to type `type()` builder, you could define it in your `Application.cfc` as `render_{type}` and then pass that *type* as a string to `type()`, since FW/1 looks up the render function by name if a string is passed.

#### OPTIONS Support

When making REST calls from JavaScript, some browsers will send an `OPTIONS` HTTP request to determine what HTTP methods are supported, as well as what headers are allowed etc. As of 4.0, FW/1 supports this via the `preflightOptions` setting. By default this is `false` but when you set it `true`, FW/1 will intercept `OPTIONS` requests, determine which routes match, and therefore which HTTP methods are actually supported, and return an empty text response, a 200 status code, and a set of headers that specify what's supported / acceptable:

* `Access-Control-Allow-Origin` - By default FW/1 returns `*` here. The `optionsAccessControl.origin` setting will override this.
* `Access-Control-Allow-Methods` - Determined by inspecting the matching routes, e.g., `GET, POST, OPTIONS`.
* `Access-Control-Allow-Headers` - By default FW/1 returns `Accept, Authorization, Content-Type` here. The `optionsAccessControl.headers` setting will override this.
* `Access-Control-Allow-Credentials` - By default FW/1 returns `true` here. The `optionsAccessControl.credentials` setting will override this.
* `Access-Control-Max-Age` - By default FW/1 returns `1728000` here (20 days, in seconds). The `optionsAccessControl.maxAge` setting will override this.

If you want to handle `OPTIONS` yourself, you can omit `preflightOptions` (or set it `false`) and provide an explicit `$OPTIONS*` route declaration that determines how to respond.

Designing Services and Domain Objects
---
Services - and domain objects - should encapsulate all of the business logic in your application. Where possible, most of the application logic should be in the domain objects, making them smart objects, and services can take care of orchestrating work that reaches across multiple domain objects.

Controllers should call methods on domain objects and services to do all the heavylifting in your application, passing specific elements of the request context as arguments. FW/1's `populate()` API is designed to allow you to store arbitrary elements of the request context in domain objects.

It is expected that you'll be using a bean factory, and your services will be autowired into your controllers, making it easier to call them directly, without having to worry about how and where to construct those CFCs. Using a bean factory means that your domain objects can also be managed, with services autowired into them as necessary, so your controllers can simply ask the bean factory for a new domain object (or ask a service for it), `populate()` it from the request context, call methods on the domain object, or pass them to services as necessary.

Services should not know anything about the framework. Service methods should not "reach out" into the request scope to interact with FW/1 - or any other scopes! - they should simply have some declared arguments, perform some operation and return some data.

### Services, Domain Objects and Persistence

There are many ways to organize how you save and load data. You could use the ORM that comes with ColdFusion, Lucee, or Railo, you could write your own data mapping service, you could write custom SQL for every domain object. Regardless of how you choose to handle your persistence, encapsulating it in a service CFC is probably a good idea. For convenience it is often worth injecting your persistence service into your domain object so you can have a convenient `domainObject.save()` call to use from your controller, even if it just delegates to the persistence service internally:

    component accessors=true {
        property dataService;
        //...
        function save() {
            variables.dataService.save( this );
        }
    }

If you use the ORM, bear in mind that it acts as a bean factory and expects to manage your domain objects -- rather than you having DI/1 manage them. This means that domain objects created via the ORM (via `entityNew()`, `entityLoad()` etc) will not have any dependencies wired in. Since such domain objects will often need access to services in your main bean factory, one approach you can use is to obtain FW/1's bean factory via the `framework.facade` (new in 4.0.0):

    var myService = new framework.facade().getBeanFactory().getBean( "someService" );

Using Bean Factories
---
By default, FW/1 will use DI/1 to manage your controllers and your model. You don't have to do anything for that to happen automatically.

_Note: in previous releases of FW/1 (prior to 3.0), you had to create the bean factory yourself and tell FW/1 about it, in `setupApplication()`. If you are migrating a 2.x application that uses a bean factory to 3.0, review the **[Bean Factory Configuration](#bean-factory-configuration)** section below to see how to set up your bean factory the same way._

FW/1 & DI/1 expect to find a `controllers` folder and a `model` folder in the base of the application tree. Under DI/1's conventions, `controllers/section.cfc` becomes `sectionController`, `model/beans/foo.cfc` becomes `fooBean`, and `model/services/bar.cfc` becomes `barService`. Other pluralized subfolders under `model` are treated in a similar fashion. By default, everything is treated as a singleton - a single, unique instance, essentially cached in `application` scope - except for CFCs in the `beans` subfolder.

In general, managing dependencies is as simple as adding `accessors=true` to your `component` tag, and declaring dependencies with the `property` keyword, like this:

    component accessors=true {
        property userService;
        property securityService;
        //...
    }

This will make `variables.userService` and `variables.securityService` available, based on `model/services/user.cfc` and `model/services/security.cfc`. You could also use long form CFC names like `userservice.cfc` and `securityservice.cfc` if you wanted. See the next section for more details on configuring DI/1.

If you let FW/1 use DI/1 to automatically manage your beans, and you are using subsystems, FW/1 will also use it to manage your subsystems' beans. See [Using Subsystems](using-subsystems.html) for more details.

As of release 3.5, DI/1 will also recognize component files with a `.lc` or `.lucee` file extension, to support Lucee 5's new dialect.

### Transients, Bean Factory, Framework

All of the above talks about singletons being injected into your controllers and services. There are three important cases that are not standard singletons:

* transients - your domain objects (in the `beans` subfolder of the `model`), each time you request one of these from the bean factory, you get a fresh instance, fully populated.
* bean factory - sometimes you need access to the bean factory directly (such as for obtaining a transient) and whilst you can get at it inside your controllers via `variables.fw.getBeanFactory()` it's better to have the bean factory injected by declaring `property beanFactory;` (which can be used in both controllers and services), then you can call `variables.beanFactory.getBean()` whenevr you need a transient.
* framework - if you let FW/1 use DI/1 (or AOP/1), it will create an alias of `fw` for the FW/1 instance and if your controllers have a constructor argument that matches, the framework will be passed into `init( fw )`; if you are using a different bean factory, or you don't want to bother with a constructor, you can declare `property framework;` and FW/1 will inject itself into your controller.

### Bean Factory Configuration

Although FW/1 uses DI/1 by default, it also has out-of-the-box support for AOP/1 and WireBox. It can also support other bean factories that follow certain conventions - see **[Custom Bean Factory Support](#custom-bean-factory)** below for more details.

You tell FW/1 which bean factory to use through the `variables.framework.diEngine` configuration variable:

* `"di1"` - the default, use DI/1.
* `"aop1"` - use AOP/1; additional configuration is the same as for DI/1.
* `"wirebox"` - use WireBox, via the supplied adapter.
* `"none"` - do not use a bean factory automatically; you may still create your own bean factory in `setupApplication()` and use `setBeanFactory()` to tell FW/1 about it (this is the easiest way to migrate a 2.x application to 3.0 - but also see below).
* `"custom"` - use a non-standard bean factory - see **[Custom Bean Factory Support](#custom-bean-factory-support)** below for the requirements on such a bean factory.

Unless you choose `"none"`, FW/1 will use `variables.framework.diComponent` as the dotted-path of a CFC to construct and use as the bean factory. This configuration variable has a default based on `diEngine` as follows:

* `"di1"` - `"framework.ioc"`
* `"aop1"` - `"framework.aop"`
* `"wirebox"` - `"framework.WireBoxAdapter"`

You can override these if you have installed FW/1's `framework` folder contents elsewhere. If you specify a `diEngine` of `"custom"`, you must supply your own value for `diComponent`.

As of FW/1 3.5, if you try to manage your own bean factory via `setBeanFactory()` and forget to set `diEngine` to `"none"`, you will get an exception. You can suppress the exception using the `diOverrideAllowed` setting but you will still get a warning printed to the console, informing you that you really should set `diEngine` to `"none"` instead!

The following configuration variables are used in the construction of the bean factory component:

* `diLocations` - for `diEngine` `"di1"` or `"aop1"`, this is the first argument to the constructor and represents a list of folders to be scanned; for `diEngine` `"wirebox"`, this may be used to set the `scanLocations()` on WireBox's binder if `diConfig` is a struct (`diLocations` is ignored if `diConfig` is a string); for `diEngine` `"custom"`, this is the first argument to the constructor.
* `diConfig` - for `diEngine` `"di1"` or `"aop1"`, this is the second argument to the constructor and represents configuration settings for DI/1 (or AOP/1); for `diEngine` `"wirebox"`, this is either the `properties` argument to the constructor (if it is a struct) or the dotted path to a binder CFC (if it is a string); for `diEngine` `"custom"`, this is the second argument to the constructor.

_Specifying a string for `diConfig` with WireBox is new in 3.5 and is the recommended way to configure FW/1 to use WireBox._

Here's how those values are used in code to construct the bean factory:

    // di1 / aop1:
    var bf = new "#variables.framework.diComponent#"(
        variables.framework.diLocations,
        variables.framework.diConfig
    );
    // wirebox -- diConfig is a struct:
    var bf = new "#variables.framework.diComponent#"(
        properties = variables.framework.diConfig
    );
    // wirebox -- diConfig is a string:
    var bf = new "#variables.framework.diComponent#"(
        variables.framework.diConfig, // binder
        variables.framework           // properties
    );
    bf.getBinder().scanLocations( variables.framework.diLocations );
    // custom:
    var bf = new "#variables.framework.diComponent#"(
        variables.framework.diLocations,
        variables.framework.diConfig
    );

If you are using subsystems and also using DI/1 as your default bean factory component, `diConfig` will be passed to subsystem bean factories when they are constructed. You can override this on a per-subsystem basis by setting `diConfig` in the specific `framework.subsystems` configuration structure. _Per-subsystem `diConfig` is new in 3.1._

#### Migrating 2.x Applications to 3.x

If you migrated through FW/1 2.5 (recommended), you'll have already dealt with the features that were deprecated in 2.5 and removed in 3.0. The other changes were that `org.corfield.framework` moved to `framework.one` and both `getRC()` and `getRCValue()` were removed (these changes were deprecated throughout the 3.0 prerelease cycle and removed in the RC cycle).

The final change that you will run into if you were using a bean factory is how FW/1 3.0 manages this automatically now. The simplest migration is to set `variables.framework.diEngine` to `"none"` and carry on doing what you were doing (manually creating the bean factory).

The recommended migration is to set the `di*` configuration variables appropriately to allow FW/1 to take over and manage your bean factory for you. If you are using DI/1, this is likely to be easier than, say, ColdSpring (see **[Custom Bean Factory Support](#custom-bean-factory-support)** below). If your DI/1 setup is fairly simple, you won't need to do much. Here are some examples:

    // 2.x setupApplication():
    var bf = new framework.ioc("model,controllers");
    setBeanFactory(bf);
    // 3.x - no configuration necessary
    // just remove those lines from setupApplication()

    // 2.x setupApplication():
    var bf = new framework.ioc("/model,/app/controllers");
    setBeanFactory(bf);
    // 3.x - remove those lines and add this config:
    variables.framework = {
        ...
        diLocations = "/model,/app/controllers",
        ...
    };

    // 2.x setupApplication():
    var bf = new path.to.ioc("/model,/app/controllers");
    setBeanFactory(bf);
    // 3.x - remove those lines and add this config:
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
    // 3.x - remove those lines and add this config:
    variables.framework = {
        ...
        diComponent = "path.to.ioc",
        diLocations = "/model,/app/controllers",
        diConfig = { ... config ... },
        ...
    };

As of FW/1 3.5, `diLocations` can now be either an array of paths or a list of paths.

If you use a load listener and call `bf.onLoad( myListener )`, you should use `diConfig` and add `loadListener = myListener` to it instead.

If you perform more complex configuration of DI/1 (adding bean declarations etc), add a new function to your `Application.cfc` that accepts the bean factory as an argument, and then specify that as the `loadListener`:

    // 2.x setupApplication():
    var bf = new path.to.ioc(
        "/model,/app/controllers",
        { ... config ... }
    );
    bf...( ... );
    ... other bf stuff ...
    setBeanFactory(bf);
    // 3.x - move the bf configuration to a load listener function in Application.cfc:
    function factoryConfig( bf ) {
        bf...( ... );
        ... other bf stuff ...
    }
    // 3.x - then remove the 2.x code and add this config:
    variables.framework = {
        ...
        diComponent = "path.to.ioc",
        diLocations = "/model,/app/controllers",
        diConfig = { ... config ..., loadListener = factoryConfig },
        ...
    };

Alternatively, create a new CFC in your model's `services` tree (e.g., `LoadListener.cfc`) and move the bean factory configuration to an `onLoad()` method there, and specify that bean name in the configuration (this is better if you have a lot of configuration since it avoids cluttering up `Application.cfc`):

    // 3.x - move the bf configuration to a LoadListener.cfc:
    component {
        function onLoad( bf ) {
            bf...( ... );
            ... other bf stuff ...
        }
    }
    // 3.x - then remove the 2.x code and add this config:
    variables.framework = {
        ...
        diComponent = "path.to.ioc",
        diLocations = "/model,/app/controllers",
        diConfig = { ... config ..., loadListener = "loadListenerService" },
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

If the specified error handler does not exist or another exception occurs during execution of the error handler, FW/1 provides a very basic fallback error handler that simply displays the exception. If you want to change this behavior, you can either override the `failure()` method or the `onError()` method but I don't intend to "support" that so the only documentation will be in the code!

Note: If you override `onMissingView()` and forget to define a view for the error handler, FW/1 will call `onMissingView()` and that will hide the original exception.

Configuring FW/1 Applications
---
All of the configuration for FW/1 is done through a simple structure in `Application.cfc`. The default behavior for the application is as if you specified this structure (but it is strongly recommended you **omit** any settings that you do not explicitly need to change!):

    variables.framework = {
        action = 'action',
        // base has no default value -- see below
        // cfcbase has no default value -- see below
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
        preserveKeyURLKey = 'fw1pk',
        maxNumContextsPreserved = 10,
        baseURL = 'useCgiScriptName',
        generateSES = false,
        SESOmitIndex = false,
        unhandledExtensions = 'cfc,lc,lucee',
        unhandledPaths = '/flex2gateway',
        unhandledErrorCaught = false,
        applicationKey = 'framework.one',
        cacheFileExists = false,
        routes = [ ],
        perResourceError = true,
        // resourceRouteTemplates - see routes documentation
        routesCaseSensitive = true,
        noLowerCase = false,
        trace = false,
        controllersFolder = "controllers",
        layoutsFolder = "layouts",
        subsystemsFolder = "subsystems",
        viewsFolder = views",
        diOverrideAllowed = false,
        diEngine = "di1",
        diLocations = [ "model", "controllers" ],
        diConfig = { },
        diComponent = "framework.ioc",
        decodeRequestBody = false,
        preflightOptions = false,
        optionsAccessControl = { },
        environments = { }
    };

The keys in the structure have the following meanings:

* `action` - The URL or form variable used to specify the desired action (`?action=section.item`).
* `base` - Provide this if the application itself is not in the same directory as `Application.cfc` and `index.cfm`. It should either be the **relative** path to the folder containing the `views` and `layouts` from the `Application.cfc` file, or a **mapped** path to that folder. Examples: `"../myapp/"`, `"/appmapping/"`. You will also need to specify the location for `controllers` and `model` via `diLocations` if you are using a bean factory (the default), or via `cfcbase` if you have disabled the bean factory.
* `cfcbase` - Essentially deprecated, this tells FW/1 how to find the `controllers` folder if you are not using a bean factory. It is used as the dotted-path prefix for controller CFCs when FW/1 is managing them (rather than a bean factory), e.g., if `cfcbase = 'com.myapp'` then a controller would be `com.myapp.controllers.MyController`.
* `usingSubsystems` - Whether or not to use legacy style subsystems - see **[Using Subsystems](#using-subsystems)** below. This is automatically set `true` if you explicitly specify a `defaultSubsystem`. As of release 3.5, it is recommended to use the new style subsystems (and leave this as `false` or omit it).
* `defaultSubsystem` - If legacy subsystems are enabled, this is the default subsystem when none is specified in the URL or form post. It defaults to `"home"`. As of release 3.5, it is recommended to use the new style subsystems (and omit this).
* `defaultSection` - This is the default section to use when none is specified in the URL or form post. It defaults to `"main"`.
* `defaultItem` - This is the default item to use when none is specified in the URL or form post. It defauts to `"default"`.
* `subsystemDelimiter` - This specifies the delimiter between the subsystem name and the section in an action. It defaults to `":"`.
* `siteWideLayoutSubsystem` - If legacy subsystems are enabled, this specifies the subsystem that is used for the (optional) site-wide default layout. It defaults to `"common"`. As of release 3.5, it is recommended to use the new style subsystems (and omit this).
* `subsystems` - An optional struct of structs containing per-subsystem configuration data. Each key in the top-level struct is named for a subsystem. The contents of the nested structs can be anything you want for your subsystems. Retrieved by calling `getSubsystemConfig()`. Currently the only keys used by FW/1 are `baseURL` and `diConfig` which can be used to configure per-subsystem values.
* `home` - The default action when it is not specified in the URL or form post. By default, this is `defaultSection`.`defaultItem`. If you specify `home`, you are overriding (and hiding) `defaultSection` but not `defaultItem`. If `usingSubsystem` is `true`, the default for `home` is `"home:main.default"`, i.e., `defaultSubsystem & subsystemDelimiter & defaultSection & '.' & defaultItem`.
* `error` - The action to use if an exception occurs. By default this is `defaultSection.error`.
* `reload` - The URL variable used to force FW/1 to reload its application cache and re-execute `setupApplication()`.
* `password` - The value of the reload URL variable that must be specified, e.g., `?reload=true` is the default but you could specify `reload = 'refresh', password = 'fw1'` and then specifying `?refresh=fw1` would cause a reload.
* `reloadApplicationOnEveryRequest` - If this is set to `true` then FW/1 behaves as if you specified the `reload` URL variable on every request, i.e., at the start of each request, the controller/service cache is cleared and `setupApplication()` is executed.
* `preserveKeyURLKey` - In order to support multiple, concurrent flash scope uses - across redirects - for a single user, such as when they have multiple browser windows open, this value is used as a URL key that identifies which flash context should be restored for that browser window. If that doesn't make sense, don't worry about it - it's magic! This value just needs to be something unique that won't clash with any of your own URL variables. This will be ignored if you set `maxNumContextsPreserved` to `1` because with only one context, FW/1 will not use a URL variable to track flash scope across redirects.
* `maxNumContextsPreserved` - If you expect users to have more than 10 browser windows open at the same time, you'll want to set this value higher. I know, Ryan was very thorough when he implemented multiple flash contexts! Setting `maxNumContextsPreserved` to `1` will prevent the URL key from being used for redirects (since FW/1 will not need to track multiple flash contexts).
* `baseURL` - Normally, `redirect()` and `buildURL()` default to using `CGI.SCRIPT_NAME` as the basis for the URL they construct. This is the right choice for most applications but there are times when the base URL used for your application could be different. You can also specify `baseURL = "useRequestURI"` and instead of `CGI.SCRIPT_NAME`, the result of `getPageContext().getRequest().getRequestURI()` will be used to construct URLs. This is the right choice for FW/1 applications embedded inside Mura.
* `generateSES` - If true, causes `redirect()` and `buildURL()` to generate SES-style URLs with items separated by `/` (and the path info in the URL will begin `/section/item` rather than `?action=section.item` - see the [Reference Manual](reference-manual.html) for more details).
* `SESOmitIndex` - If SES URLs are enabled and this is `true`, will attempt to omit the base filename in the path when constructing URLs in `buildURL()` and `redirect()` which will generally omit `/index.cfm` from the start of the URL. Again, see the [Reference Manual](reference-manual.html) for more details.
* `unhandledExtensions` - A list of file extensions that FW/1 should not handle. By default, just requests for CFCs, e.g., `some.cfc`, are not handled by FW/1. As of release 3.5, Lucee components, with extensions of `.lc` or `.lucee` are also not handled by the framework.
* `unhandledPaths` - A list of file paths that FW/1 should not handle. By default, just requests for `/flex2gateway` are not handled by FW/1 (hey, some people are still using Flex - don't judge!). If you specify a directory path, requests for any files in that directory are then not handled by FW/1. For example, `unhandledPaths = '/flex2gateway,/404.cfm,/api'` will cause FW/1 to not handle requests from Flex, requests for the `/404.cfm` page and any requests for files in the `/api` folder.
* `unhandledErrorCaught` - By default the framework does not attempt to catch errors raised by unhandled requests but sometimes when you are migrating from a legacy application it is useful to route error handling of legacy (unhandled) requests through FW/1. The default for this option is `false`. Set it `true` to have FW/1's error handling apply to unhandled requests.
* `applicationKey` - A unique value for each FW/1 application that shares a common ColdFusion application name.
* `cacheFileExists` - If you are running on a system where disk access is slow - or you simply want to avoid several calls to `fileExists()` during requests for performance - you can set this to true and FW/1 will cache all its calls to `fileExists()`. Be aware that if the result of `fileExists()` is cached and you add a new layout or a new view, it won't be noticed until you reload the framework.
* `routes` - An array of URL path mappings. This allows you to override the conventional mapping of `/section/item` to controllers.
* `perResourceError` - Default `true`. Controls whether a wildcard route is added to each resouce template. See **[URL Routes](url-routes)** for more details.
* `resourceRouteTemplates` - see **[URL Routes](url-routes)** below.
* `routesCaseSensitive` - Default `true`. Controls whether route matches are case-sensitive or not. _New in 3.1._
* `noLowerCase` - If `true`, FW/1 will not force actions to lowercase so subsystem, section and item names will be case sensitive (in particular, filenames for controllers, views and layouts may therefore be mixed case on a case-sensitive operating system). The default is `false`. Use of this option is _not_ recommended and is not considered good practice.
* `trace` - If `true`, FW/1 will print out debugging / tracing information at the bottom of each page. This can be very useful for debugging your application! If you want to track framework behavior across redirects, you need to enable session management in your application if you use this feature. (Note that FW/1 will not print out debugging / tracing information when the `renderData()` function is used, unless the content type is `"html"`. You can still access and output debugging / tracing information in such cases by overriding the `setupTraceRender()` function. See the [Reference Manual](reference-manual.html) for more details.).
* `controllersFolder` - The name used for the controllers folder. Must be plural. Defaults to `"controllers"` but could be `"handlers"` for example. _New in 3.5._
* `layoutsFolder` - The name used for the layouts folder. Must be plural. Defaults to `"layouts"` but could be `"wrappers"` for example. _New in 3.5._
* `subsystemsFolder` - The name used for the subsystems folder. Must be plural. Defaults to `"subsystems"` but could be `"plugins"` for example. _New in 3.5._
* `viewsFolder` - The name used for the views folder. Must be plural. Defaults to `"views"` but could be `"pages"` for example. _New in 3.5._
* `diOverrideAllowed` - If `true`, FW/1 will throw an exception if you attempt to call `setBeanFactory()` twice. If `false`, FW/1 will allow you to call `setBeanFactory()` twice and override the previous Dependency Injection setting, but it will log a warning to the console. If you want FW/1 to manage your bean factory, use the `di*` settings above to configure it -- and do not call `setBeanFactory()` yourself. If you want to manage your bean factory directly, set `diEngine` to `"none"` so FW/1 doesn't also attempt to do this. _New in 3.5._
* `diEngine` - The Dependency Injection framework that FW/1 should use.
* `diLocations` - The list of folders to check for CFCs to manage; defaults to `[ "model", "controllers" ]`. If you've had to use `base` to tell FW/1 where your `views` and `layouts` are, you'll need to include that location in the paths to the folders where your CFCs are, and use a **mapped** path instead of a relative path.
* `diConfig` - Any additional configuration needed for the Dependency Injection engine; defaults to `{ }`.
* `diComponent` - The dotted-path to the CFC used for the bean factory (which has sensible defaults based on `diEngine`).
* `decodeRequestBody` - Default `false`. If `true`, FW/1 will accept JSON or URL-encoded data in the request body (commonly provided by POST / PUT operations) and decode it automatically into the request context. _New in 4.0._
* `preflightOptions` - Default `false`. If `true`, FW/1 will handle HTTP `OPTIONS` requests for you. See **[OPTIONS Support](#options-support)** above for more details. _New in 4.0._
* `optionsAccessControl` - Default `{ }`. You can use this to override the default `Access-Control-*` headers returns by FW/1's `OPTIONS` support. Valid keys are: `origin`, `headers`, `credentials`, and `maxAge`. _New in 4.0._
* `environments` - An optional struct containing per-tier and per-server configuration that should be merged into FW/1's settings. See **[Environment Control](#environment-control)** below for more details.

At runtime, this structure also contains the following key (from release 0.4 onward):

* `version` - The release number (version) of the framework.

This is set automatically by the framework and cannot be overridden (well, it shouldn't be overridden!).

URL Routes
---
In addition to the standard `/section/item` and `/module:section/item` URLs that FW/1 supports, you can also specify `routes` that are URL patterns, optionally containing variables, that map to standard `/section/item` and `/module:section/item` URLs.

To use routes, specify `variables.framework.routes` as an array of structures, where each structure specifies mappings from routes to standard URLs. The array is searched in order and the first matching route is the one selected (and any subsequent match is ignored). This allows you to control which route should be used when several possibilities match.

Placeholder variables in the route are identified either by a leading colon or by braces (specifying a variable name and a regex to restrict matches) and can appear in the URL as well, for example `{ "/product/:id" = "/product/view/id/:id" }` specifies a match for `/product/something` which will be treated as if the URL was `/product/view/id/something` - section: `product`, item: `view`, query string `id=something`. Similarly, `{ "/product/{id:[0-9]+}" = "/product/view/id/:id" }` specifies a match for `/product/42` which will be treated as if the URL was `/product/view/id/42`, and only numeric values will match the placeholder.

Routes can also be restricted to specific HTTP methods by prefixing them with `$` and the _method_, for example `{ "$POST/search" = "/main/search" }` specifies a match for a `POST` on `/search` which will be treated as if the URL was `/main/search` - section: `main`, item: `search`. A `GET` operation will not match this route.

Routes can also specify a redirect instead of a substitute URL by prefixing the URL with an HTTP status code and a colon, for example `{ "/thankyou" = "302:/main/thankyou" }` specifies a match for `/thankyou` which will cause a redirect to `/main/thankyou`.

A route of `"*"` is a wildcard that will match any request and therefore must be the last route in the array. A wildcard route may be restricted to a specific method, e.g., `"$POST*"` will match a `POST` to any URL.

Route matches are case-sensitive unless you set `routesCaseSensitive` to `false` in the FW/1 configuration.

The keyword `"$RESOURCES"` can be used as a shorthand way of specifying resource routes: `{ "$RESOURCES" = "dogs,cats,hamsters,gerbils" }`. FW/1 will interpret this as if you had specified a standard set of routes for each of the listed resources. For example, for the resource `"dogs"`, FW/1 will parse the following routes:

    { "$GET/dogs/$" = "/dogs/default" },
    { "$GET/dogs/new/$" = "/dogs/new" },
    { "$POST/dogs/$" = "/dogs/create" },
    { "$GET/dogs/:id/$" = "/dogs/show/id/:id" },
    { "$PATCH/dogs/:id/$" = "/dogs/update/id/:id", "$PUT/dogs/:id/$" = "/dogs/update/id/:id" },
    { "$DELETE/dogs/:id/$" = "/dogs/destroy/id/:id" },
    { "$*/dogs/$" = "/dogs/error" }

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
         { method = 'destroy', httpMethods = [ '$DELETE' ], includeId = true },
         { method = 'error', httpMethods = [ '$*' ] }
    ];

The latter causes the `error` handler to be invoked for any API request that matches the resource itself, but either has an unknown HTTP method or does not match the pattern of a standard route (e.g., a `DELETE`, `PUT`, or `PATCH` without an `id`). _New in 4.0_

The per-resource error handling can be turned off by setting `perResourceError` to `false` in the framework configuration. This will restore the FW/1 3.5 error handling behavior.

If you wish to change the controller methods the routes are mapped to, for instance, you can specify this array in your `Application.cfc` and then change the default method names. For example, if you want `"$GET/dogs/$"` to map to `"/dogs/index"`, you would change `method = 'default'` to `method = 'index'` in the first template struct.

A route structure may also have documentation by specifying a hint: `{ "/product/:id" = "/product/view/id/:id", hint = "Display a product" }`.

Here's an example showing all the features together:

    variables.framework.routes = [
        { "/product/:id" = "/product/view/id/:id", "/user/{id:[0-9]+}" = "/user/view/id/:id",
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
The subsystems feature allows you to modularize your FW/1 application as it grows, by breaking it down into a series of "mini FW/1 applications". It can also allow you to combine  other FW/1 applications as modules into your application. The subsystems feature was originally contributed by Ryan Cogswell and has been overhauled as part of release 3.5, based on ideas by Steve Neiland. The original documentation was written by Dutch Rapley. Read about [Using Subsystems](using-subsystems.html) to modularize or combine FW/1 applications.

Accessing the FW/1 API
===
FW/1 uses the `request` scope for some of its temporary data so that it can communicate between `Application.cfc` lifecycle methods without relying on `variables` scope (and potentially interfering with user data in variables scope). The [Reference Manual](reference-manual.html#request-scope) specifies which request scope variables are used and what you may and may not do with them.

In addition, the API of FW/1 is exposed to controllers, views and layouts in a particular way as documented below.

Controllers and the FW/1 API
---
Each controller method is passed the request context as a single argument called `rc`, of type `struct`. If access to the FW/1 API is required inside a controller, you can define an `init()` method (constructor) which takes a single argument `fw`, of type `any`, and when FW/1 creates the controller CFC, it passes itself in as the argument to `init()`. Your `init()` method should save the `fw` argument in the variables scope for use within the controller methods:

    function init( fw ) {
        variables.framework = fw;
        return this;
    }

    // make your controller bean factory aware
    function setBeanFactory( beanFactory ) {
        variables.beanFactory = beanFactory;
    }

Alternatively, you can declare a dependency on the `framework` like this:

    property beanFactory; // make your controller bean factory aware
    property framework;   // make your controller framework aware

Then you could call any framework method:

     var user = variables.beanFactory.getBean( "user" );
     variables.framework.populate( user );

This will call `setXxx()` methods on the user bean, passing in matching elements from the request context. An optional second argument may be provided that specifies the keys to populate (the default is to attempt to match against every `setXxx()` method on the bean):

    variables.framework.populate( user, 'firstName, lastName, email' );

This will call `setFirstName()`, `setLastName()` and `setEmail()` on the user bean, passing in matching elements from the request context.

Other framework methods that are useful for controllers include `setView()`, `setLayout()`, `abortController()`, and `redirect()`:

    variables.framework.redirect( action, preserve, append, path, queryString );

where `action` is the action to redirect to, `preserve` is a list of request context keys that should be preserved across the redirect (using `session` scope) and `append` is a list of request context keys that should be appended to the redirect URL. `preserve` and `append` can both be omitted and default to none, i.e., no values preserved or appended. The optional `path` argument allows you to force a new base URL to be used (instead of the default `variables.framework.baseURL` which is normally `CGI.SCRIPT_NAME`). `queryString` allows you to specify additional URL parameters and/or anchors to be added to the generated URL. See the [Reference Manual](reference-manual.html#public-void-function-redirect-string-action-string-preserve--none-string-append--none-string-path--see-below-string-querystring---string-statuscode--302-string-header---) for more details.

Views/Layouts and the FW/1 API
---
As indicated above under the "in depth" paragraph about views and layouts, the entire FW/1 API is available to views and layouts directly (effectively in the `variables` scope) because of the way views and layouts are executed. This allows views and layouts to access utility beans from the bean factory, such as formatting services, as well as render views and, if necessary, other layouts. Views and layouts also have access to the `framework` structure which contains the `action` key - which could be used for building links:

    <a href="?#framework.action#=section.item">Go to section.item</a>

But you're better off using the `buildURL()` API method:

    <a href="#buildURL( 'section.item' )#">Go to section.item</a>

You can provide additional query string values to `buildURL()`:

    <a href="#buildURL( 'section.item?arg=val' )#">Go to section.item with arg=val in URL</a>
    <a href="#buildURL( action = 'section.item', queryString = 'arg=val' )#">Go to section.item with arg=val in URL</a>

Other framework methods that are useful for views or layouts include `view()`, `layout()` and `getBeanFactory()`.

Convenience Methods in the FW/1 API
---
FW/1 provides a number of convenience methods for manipulating the action value to extract parts of the action (the `action` argument is optional in all these methods and defaults to the currently requested action):

* `getSubsystem( action )` - Returns the _module_ portion of the action, which may be empty. If the module name is empty and you are using legacy subsystems, this will return the home subsystem name.
* `getSection( action )` - Returns the _section_ portion of the action. If no section is specified, returns the default section.
* `getItem( action )` - Returns the _item_ portion of the action. If no item is specified, returns the default item.
* `getSectionAndItem( action )` - Returns the _section.item_ portion of the action, including default values if either part is not specified.
* `getFullyQualifiedAction( action )` - Returns the fully qualified _module:section.item_ version of the action, including defaults where appropriate. If the module name is empty, returns `getSectionAndItem( action )`, without the subsystem delimiter. Be careful that _section.item_ is a subsystem-relative action so if you use it inside a subsystem, it will be treated as part of the current subsystem, which is not the same as _:section.item_ (which is treated as part of the main application). This is provided mostly for backward-compatibility and use with Subsystems 1.0 applications.
* `getSubsystemSectionAndItem( action )` - Returns the fully qualified _module:section.item_ version of the action, including defaults where appropriate. If the module name is empty, the subsystem delimiter is still present (so you get _:section.item_) unlike `getFullyQualifiedAction()` above.
