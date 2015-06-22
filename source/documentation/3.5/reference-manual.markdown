---
layout: page
title: "FW/1 Reference Manual"
date: 2015-06-22 14:00
comments: false
sharing: false
footer: true
---
_This is the upcoming (3.5 - clojure) documentation - for the current stable release, read the [3.0 master documentation](/documentation/3.0/reference-manual.html)._

This page provides a description of all the APIs and components involved in a FW/1 application. Please also read the [Roadmap](/documentation/3.5/roadmap.html) to see how things may change in the future.

FW/1 Controllers
---
A controller in a FW/1 application does not need to extend any base component.

A controller method is passed a single struct called `rc` (request context). This structure initially contains all the URL and form scope variables passed into the request. Form scope takes precedence (i.e., when the same key appears in both URL and form scope, the value for that key in the request context is taken from form scope).

A controller communicates data to other parts of a FW/1 application by adding data to `rc`.

The following public methods are significant in a controller (and are all optional):

* `void before(struct rc)` - called at the start of each request to this `section`
* `void item(struct rc)` - called next, for `section.item`
* `void after(struct rc)` - called at the end of each request to this `section` before views are rendered.

If the controller needs to invoke FW/1 API methods (see **framework.one** below), the controller must either depend on `property framework;` (and have `accessors=true` on the `component` tag) or have a constructor (a method called `init()`) that accepts an instance of the FW/1 CFC and saves it for use in other methods. The following is the recommended way to write your controller's constructor:

    function init(fw) {
        variables.fw = fw;
        return this;
    }

Within other controller methods, you can then invoke FW/1 API methods using `variables.fw.apimethod(args)` if you use this constructor approach, or `variables.framework.apimethod(args)` if you use the `property framework;` approach.

Your `Application.cfc` is also considered a controller and if it defines `before()` or `after()` methods, those will be called at the start and end of the controller lifecycle. Unlike other controllers, it does not need an `init()` method and instead of referring to the FW/1 API methods via `variables.fw...` you can just use the API methods directly - unqualified - since `Application.cfc` extends the framework and all those methods are available implicitly.

A controller is instantiated on the first request to an _item_ in that _section_ and is cached in `application` scope. You can ask FW/1 to reload the cache at any point (by default, you add `?reload=true` to the URL).

You can abort processing of controllers and services by calling `variables.fw.abortController()` which throws an exception that is caught by the framework. Execution of the current controller is immediately aborted and execution continues with `setupView()`. If your controller code catches and swallows the exception, execution of your controller code will proceed from your catch statement until it returns but at that point the framework will not execute any further controllers and execution continues with `setupView()`.

FW/1 Views
---
All views execute as included files within the context of a FW/1 request, i.e., inside the current request's instance of `Application.cfc`. That means that all methods inside FW/1 and inside your `Application.cfc` are available directly inside your views. See the public API documentation below for what you can and should use of those methods! This means that you can have view helper methods in your `Application.cfc`, either directly or via an include of a library file.

This process also means that you are _mostly_ free of thread safety issues (because each request automatically has its own context). That said, you need to cognizant of two things when storing data in the `variables` scope of a view:

* The `variables` scope of a view is the `variables` scope of FW/1 / `Application.cfc` - so don't overwrite anything the framework might need!
* Data stored in `variables` scope by one view is accessible to other views _executed in the same request_ such as those views rendered by calls to `view()` in layouts.

You can avoid those concerns by using `local` scope for any new variables introduced inside the view. In addition to `local` scope, each view also has a local `rc` variable which is the request context struct. This is the recommended way for views to communicate if they have a need to do so (e.g., setting a page title in a view so that a layout or later view can render it). `rc` contains whatever data the controller(s) have provided, as well as what was originally in the URL and/or form scopes.

The following variables are also available but `should not be relied upon` as they are implementation details that may change in the future:

* `viewPath` / `layoutPath` - The path to the view or layout file.
* `$` - An integration point with Mura (only relevant if you're writing Mura plugins with FW/1).
* `args` - A struct of additional data passed into the `view()` call.
* `response` - A local variable that will be overwritten by the output of the rendered view.

Any other variables assigned to by a view (without a scope qualifier - or explicitly with `variables` scope) are part of the FW/1 instance.

If no matching view file exists for a request, `onMissingView()` is called and whatever is returned is used as the text of the view, and layouts are applied (unless they are suppressed). The default implementation is to throw an exception but by overriding this method you can create any behavior you want for requests that have no specific view, e.g., you can return a default view or pretty much anything you want.

As noted in the [Developing Applications Manual](/documentation/3.5/developing-applications.html), `onMissingView()` will be called if your application throws an exception and you have not provided a view for the default error handler (`main.error` - if `defaultSection` is `main`). This can lead to exceptions being masked and instead appearing as if you have a missing view!

FW/1 Layouts
---
Everything that applies to views above also applies to layouts. The variables that are available to layouts are the same as for views without `args` and with just one addition:

* `body` - This contains the rendered views / layouts so far (as the layouts cascade).

Layouts have exactly the same access to the FW/1 API as views and all the same considerations apply.

Request Scope
---
FW/1 uses the `request` scope fairly extensively to pass data between parts of the application in order to keep things simple and decoupled (from a developer's point of view). This section documents all of the request scope variables used by the framework. In general, you should not reference any of these `request` scope variables: there are supported API methods that you should use instead for the data that you might want to access (and those are documented in the following list).

Request variables:

* `request._fw1` - An opaque structure intended to hide most of the internal request scope variables that FW/1 uses.
* `request.action` - The action specified in the URL or form (after expansion to the fully qualified form). This can be obtained by calling `getFullyQualifiedAction()`.
* `request.base` - The canonical path to the application directory (where the `views/` and `layouts/` folders are). There is no API method to access this.
* `request.cfcbase` - If your controllers are not managed by a bean factory, this is the dot-separated path prefix for controller CFCs. There is no API method to access this.
* `request.context` - The request context, the main "data bus" through the application. This is available in controller methods as the argument `rc` and in the views and layouts as the (local) variable `rc`.
* `request.event` - When an error action is executed, this holds the `Application.cfc` event in which the exception occurred (it is the argument to the `onError()` method). An API method may be added in future to access this.
* `request.exception` - When an error action is executed, this holds the original exception that occurred (it is the argument to the `onError()` method). An API method may be added in future to access this.
* `request.failedAction` - When an error action is executed, this holds the original action being processed when the exception occurred. If the exception occurred before the point at which the action is determined, `request.failedAction` will not be defined. An API method may be added in future to access this.
* `request.failedCfcName` - If an exception occurs during execution of a controller, this holds the name of the failed controller CFC. This can be accessed in the error action to provide more details of where the exception occurred. An API method may be added in the future to access this.
* `request.failedMethod` - If an exception occurs during execution of a controller, this holds the name of the failed method (on the controller CFC). This can be accessed in the error action to provide more details of where the exception occurred. An API method may be added in the future to access this.
* `request.item` - The item portion of the action. This can be obtained by calling `getItem()` (with no argument).
* `request.layout` - This is a boolean that indicates whether layouts should be rendered. It can be set in a view or layout to prevent any further layouts from being processed. **This is the only `request` scope variable which a developer should be updating directly.** An API method may be added in future for this.
* `request.missingView` - When `onMissingView()` is triggered, this hold the name of the view that was not found. An API method may be added in future to access this.
* `request.section` - The section portion of the action. This can be obtained by calling `getSection()` (with no argument).
* `request.subsystem` - The subsystem portion of the action, if appropriate. This can be obtained by calling `getSubsystem()` (with no argument).
* `request.subsystembase` - The path from the main application directory to the current subsystem's directory (where the `views/` and `layouts/` folders are). This can be obtained by calling `getSubsystemBase()`.

Where there is no API method, there is generally an underlying assumption that a developer should not need access to that data. If it turns out that developers are commonly referring to the underlying `request` scope variables due to various needs, the addition of API methods might be considered.

framework.one
===
This section documents every public method in FW/1's core file.

These methods are callable from outside the framework and are intended to be used with controllers, layouts and views or may be intended to be overridden in you `Application.cfc`. Technically, methods in FW/1 do not need to be `public` to be called from within layouts, views or `Application.cfc` but for the most part, the convention is: if it's `public`, it's documented and guaranteed to remain part of the API; if it's `private`, whilst it may be called within layouts, views and `Application.cfc` it is not recommended.

### public void function abortController()

Attempts to immediately abort execution of the current controller by throwing an exception (which is caught by the framework). If your controller catches this exception, execution will continue until your controller returns. No further controller or service methods will be called. Execution will continue with the `setupView()` lifecycle method and views and layouts will then be rendered.

### public boolean function actionSpecifiesSubsystem( string action )

Returns `true` if the application is using subsystems and the action contains a colon `:` (default - `framework.subsystemDelimiter`). Returns `false` if the application is not using subsystems or the action does not contain a colon `:` (default - `framework.subsystemDelimiter`).

### addRoute( any routes, string target, any methods = [ ], string statusCode = '' )

Allows you to programmatically add a new route to FW/1's known route mappings. `routes` can be either an array of route patterns or a single route pattern. `target` is the URL to map those routes to. `methods` is optional and can be either an array of HTTP methods or a single HTTP method for which those routes should be mapped. `statusCode` is optional and if present is prefixed to the target URL (and represents the HTTP status returned when the routes are processed.

### public string function buildCustomURL( string uri )

Used in views and layouts to generate route-based links. Generates a URL prefix, the same way `buildURL()` does, and appends `uri`.

### public string function buildURL( string action = '.', string path = _see below_, any queryString = '' )

Used in views and layouts to generate links to FW/1 actions. Produces "traditional" links if `variables.framework.generateSES` is `false` and the current request used a "traditional" URL. Produces "SES" links if `variables.framework.generateSES` is `true` or the current request used an "SES" URL. The optional `path` argument allows you to override the default base URL used for links in the same way that `redirect()` allows below. Note that specifying `path` will disable generation of "SES" links! `path` will default to the result of calling `getBaseURL()` which in turn defaults to `variables.framework.baseURL`, unless overridden by a subsystem-specific configuration.

The optional `queryString` argument allows you to specify URL variables (and values) that should be added to the generated URL. In general, variable / value pairs should be specified with an equals `=` and separated with an ampersand `&` and they will be appended either as-is, for "traditional" link generation, or converted to "SES" format as appropriate. The "SES" conversion can be overridden by preceding a sequence of variable / value pairs with `?` in which case such arguments will be appended as-is for both forms of generated link. Finally, an HTML anchor may be specified, preceded by `#` and that will be appended to the final URL. `queryString` also accepts a structure, which is converted to an HTML escaped query string.

As a shortcut, the `action` may include the `queryString` value, separated by `?` like this: `'section.item?arg=val'` (with all the same considerations for embedded `&` and `?` and `#` characters). `action` can be `'.'`, in which case a link to the current section and item is returned.

Here are some examples:

    buildURL( 'product.list' )

Will generate:

* `/index.cfm?action=product.list` - in "traditional" mode
* `/index.cfm/product/list` - in "SES" mode
* `/product/list` - in "SES" mode with `SESOmitIndex` set to `true`

    buildURL( action = 'product.detail', queryString = 'id=42?img=large##overview' )

Will generate:

* `/index.cfm?action=product.detail&id=42&img=large#overview` - in "traditional" mode
* `/index.cfm/product/detail/id/42?img=large#overview` - in "SES" mode
* `/product/detail/id/42?img=large#overview` - in "SES" mode with `SESOmitIndex` set to `true`

    buildURL( 'product.detail?id=42?img=large##overview' )

Will also generate:

* `/index.cfm?action=product.detail&id=42&img=large#overview` - in "traditional" mode
* `/index.cfm/product/detail/id/42?img=large#overview` - in "SES" mode
* `/product/detail/id/42?img=large#overview` - in "SES" mode with `SESOmitIndex` set to `true`

    buildURL( '.list' )

Will generate a URL based on the current section - a section-relative link to the current section's `list` item.

    buildURL( action = 'product.detail', queryString = { id = 76, img = 'small' } )

Will generate:

* `/index.cfm?action=product.detail&id=76&img=small` - in "traditional" mode
* `/index.cfm/product/detail/id/76/img/small` - in "SES" mode
* `/product/detail/id/76/img/small` - in "SES" mode with `SESOmitIndex` set to `true`

### public void function controller( string action )

Call this from your `Application.cfc` methods to add to the queue of controllers that will be called by the framework. The `action` is used to identify the controller that should be called, e.g., `"app1:section.item"`, `"section.item"` or `"section"` (which will call the default item with that section).

A typical example is to trigger a security controller method invoked from setupRequest()`, e.g.,

    function setupRequest() {
        controller( 'security.checkAuthorization' );
    }

You may only queue up additional controllers prior to the start of controller execution (in `onRequest()`). If you attempt to queue up additional controllers in controller methods (or later in the request), you will get an exception because at that point all controllers have been queued up and/or executed.

Just like the implicit controller invocation, `before()`, `item()`, and `after()` are all invoked appropriately if present. If multiple methods are queued up from a single controller, `before()` and `after()` are executed just once (for each controller).

### public string customizeViewOrLayoutPath( struct pathInfo, string type, string fullPath )

By default, this simply returns `fullPath`.

It can be overridden to provide customized handling of view and layout locations. Everywhere that FW/1 needs to figure out the location of a view or layout based on conventions, it calls this method. See the `skinning` example in the FW/1 distribution that shows how this method can be used to provide automatic overrides of the default conventions.

The arguments are as follows:

* `pathInfo` - This struct contains two keys: `base` which is the application-relative path to the location of the (default) `views` and `layouts` folders; `path` which is the relative path below those folders. For example, for a request of `"sub:section.item"`, this structure will contain: `base = "sub/"`, `path = "section/item"`. The `base` will have already been adjusted to wherever the views / layouts are supposed to be if you have `framework.base` set.
* `type` - Either `"view"` or `"layout"`.
* `fullPath` - The default location of the view or layout: `"#pathInfo.base##type#s/#pathInfo.path#.cfm"`. For the example above, this would be `"sub/views/section/item.cfm"` for a view.

Additional documentation will be provided for this feature in due course. Probably an entire section on skinning applications with FW/1.

### public void function disableFrameworkTrace()

Disable framework tracing for this request.

### public void function disableLayout()

Disable layouts for this request. Equivalent to `request.layout = false;`.

### public void fuction enableFrameworkTrace()

Enable framework tracing for this request.

### public void fuction enableLayout()

Enable layouts for this request. Equivalent to `request.layout = true;`.

### public void function frameworkTrace( string message [, any value ] )

Adds the `message` and optional `value` to the framework trace data, for rendering at the end of the request.

### public string function getAction()

Returns the name of the action variable in the URL or form post (`variables.framework.action`).

### public string function getBaseURL()

Returns the configured `variables.framework.baseURL` value. Can be overridden in `Application.cfc` to provide a customized value, e.g., per request.

### public any function getBeanFactory( string subsystem = "" )

Returns whatever the framework has been told is a bean factory (which will be an instance of DI/1 by default). If you are using subsystems, this will return a subsystem-specific bean factory if one exists for the specified subsystem, or for the subsystem of the current request if no subsystem is specified in the call. Otherwise it will return the default bean factory.

More specifically, if you are not using subsystems:

* `getBeanFactory()` - returns the bean factory for the application.
* `getBeanFactory(subsystem)` - `subsystem` is ignored and this returns the bean factory for the application.

If you are using subsystems:

* `getBeanFactory()` - returns the subsystem bean factory for the subsystem of the current request if one exists, otherwise the default bean factory for the application.
* `getBeanFactory(subsystem)` - returns the subsystem bean factory for the named `subsystem` if one exists, otherwise the default bean factory for the application.

If no application bean factory can be found, this will throw an exception. Use `hasBeanFactory()` and/or `hasSubsystemBeanFactory( subsystem )` to determine whether this call will successfully return a bean factory.

### public struct function getConfig()

Returns a copy of the `framework` structure containing the configuration settings specified in `Application.cfc`. This allows controllers to inspect the FW/1 configuration settings if necessary.

### public any function getDefaultBeanFactory()

Returns whatever the framework has been told is a bean factory. This will return the default, top-level bean factory for the application. If no such bean factory exists, this will throw an exception. Use `hasDefaultBeanFactory()` to determine whether the default, top-level bean factory exists for the application.

### public string function getDefaultSubsystem()

If the application is not using subsystems, this returns an empty string. If the current request's action specifies a subsystem, return that. Otherwise return the default subsystem configured for the application.

If the application is using subsystems and the current request's action does not specify a subsystem and there is no default subsystem configured, this will throw an exception.

### public string function getEnvironment()

Returns an empty string by default. If you want to use the **Environment Control** feature, you should override this in `Application.cfc` and have it return the appropriate _"tier"_ or _"tier-server"_ string. See **Environment Control** in the [Developing Applications Manual](/documentation/3.5/developing-applications.html) for more detail.

### public string function getEnvVar( string name )

Returns the value of the specified environment variable name (i.e., from the shell environment in which your CFML server was started).

### public array function getFrameworkTrace()

Returns an array of the current request's framework trace data. See `setupTraceRender()` for one reasonable use.

### public string function getFullyQualifiedAction( string action = request.action )

If the application is not using subsystems, this behaves the same as `getSectionAndItem( action )`.

If the application is using subsystems, this returns the specified action formatted as _subsystem:section.item_.

### public string function getHostname()

Returns the server's local hostname (via Java's InetAddress class). Intended to be used with **Environment Control**.

### public string function getItem( string action = request.action )

Returns the item portion of the specified action - or of the current request's action if no action is specified. Returns the default item if no item is present in the specified action.

### public string function getRoute()

Returns the route that was used to initiate the current request (if any). Returns an empty string if the current request was not initiated via a matched route.

### public array function getRoutes()

Returns (a copy of) `variables.framework.routes`. This can be overridden in `Application.cfc` if you want to generate routes dynamically.

### public array function getResourceRouteTemplates()

Returns (a copy of) `variables.framework.resourceRouteTemplates`. This can be overridden in `Application.cfc` if you want to generate resource templates for routes dynamically.

### public string function getSection( string action = request.action )

Returns the section portion of the specified `action` - or of the current request's action if no action is specified. Returns the default section if no section is present in the specified action.

### public string function getSectionAndItem( string action = request.action )

Returns the specified `action` - or the current request's action if no action is specified - formatted as _section.item_. Automatically strips the subsystem from the `action`, if present. Automatically adds the default section if not present. Automatically adds the default item if not present.

### public string function getSubsystem( string action = request.action )

Returns the subsystem portion of the specified `action` - or of the current request's action if no action is specified. Returns the default subsystem if no subsystem is present in the specified action. If the application is not using subsystems, this returns the default subsystem name (which is an empty string by default).

### public string function getSubsystemBase()

Returns the path to the folder where the current request's subsystem `views/` and `layouts/` subfolders can be found. This can be useful for customer versions of `buildURL()` etc.

### public any function getSubsystemBeanFactory( string subsystem )

Returns whatever the framework has been told is a bean factory. This will return the bean factory for the named subsystem. If no such bean factory exists, this will throw an exception. Use `hasSubsystemBeanFactory( subsystem )` to determine whether a bean factory exists for the named subsystem. If this is the first reference to this subsystem, the subsystem will be initialized.

### public struct function getSubsystemConfig( string subsystem )

Returns the configuration for the named subsystem, as a copy of `variables.framework.subsystems[subsystem]`. If no configuration exists for the named subsystem, an empty struct is returned. FW/1 uses this to retrieve the per-subsystem `baseURL` value, if any, as part of `buildURL()` and `redirect()`, as well as `diConfig` if you are using DI/1 to manage subsystem bean factories automatically. _`diConfig` is new in 3.1._

### public boolean function hasBeanFactory()

Returns `true` if a default, top-level bean factory exists. If using subsystems, returns `true` if a bean factory exists for the subsystem of the current request. Otherwise returns `false`. If `hasBeanFactory()` returns `true`, calling `getBeanFactory()` will return a bean factory.

### public boolean function hasDefaultBeanFactory()

Returns `true` if a default, top-level bean factory exists. Otherwise returns `false`. If `hasDefaultBeanFactory()` returns `true`, calling `getDefaultBeanFactory()` will return a bean factory.

### public boolean function hasSubsystemBeanFactory( string subsystem )

Returns `true` if a bean factory exists for the named subsystem. Otherwise returns `false`. If `hasSubsystemBeanFactory( subsystem )` returns `true`, calling `getBeanFactory( subsystem )` and `getSubsystemBeanFactory( subsystem )` will both return a bean factory (for the named subsystem).

### public boolean function isCurrentAction( string action )

Returns `true` if the `action` passed in matches the currently executing action. This can be useful to figure out which tab to highlight in navigation or make other choices based on actions.

### public boolean isUnhandledRequest( string targetPath )

By default, returns `true` if the specified `targetPath` (as in `onRequest()`) has an unhandled extension (from `variables.framework.unhandledExtensions`) or matches an unhandled path (from `variables.framework.unhandledPaths`). You can override this to dynamically tell FW/1 not to handle specific requests. If you want to still apply the default checks as well as your own custom checks, don't forget to call `super.isUnhandledRequest(targetPath)`.

### public string function layout( string path, string body )

This function renders a layout and could be called inside a view or a layout, although it is recommended to rely on the conventions for layouts where possible. If you decide you need to render a layout directly, you can invoke it like this:

    writeOutput( layout( 'main/nav-template', nav_menu ) );

Rendering views, using the `view()` method, is supported, documented and the recommended way to build composite pages. Layouts should simply wrap views, in a cascade from item to section to site.

### public function onApplicationStart()

Part of the standard CFML lifecycle, this method is called automatically by the CFML engine at application startup. You should not override this (even tho' it is `public`). Use `setupApplication()` to perform application-specific initialization.

If you override this method, you **must** call `super.onApplicationStart()` or FW/1 will fail to work correctly.

### public function onError( any exception, string event )

The standard CFML error handler, this method is called automatically by the CFML engine in the event of an uncaught exception. By default, FW/1 will try to execute the action specified by `variables.framework.error`. The action that caused the exception, if known, will be available in `request.failedAction`. The exception and event are available as `request.exception` and `request.event` respectively. If the error action fails, FW/1 tries to display the original exception and event in a simple error view (using the private `failure()` method).

You may override this method in `Application.cfc` if you wish to provide different error handling behavior. You may call `super.onError( exception, event )` if appropriate. You may also consider overriding the `failure()` method, but that is not documented and not supported.

### public string function onMissingView( struct rc )

Called when a view is not found for a request. The default behavior is to call `viewNotFound()` which throws an exception.

You may override this method to provide alternative behavior when a view is not found. You should either throw an exception or return a string that represents the view that should be rendered instead. For example:

    function onMissingView( rc ) {
       return view( 'page/notFound' );
    }

### public void function onPopulateError( any cfc, string property, struct rc )

Called when an exception occurs during an attempt to populate the named `property` of the specified `cfc` if no keys were specified for `populate()` and `trustKeys` was `true`. This method does nothing, effectively causing the exception to be ignored.

If you intend to call `populate()` with no keys specified and you tell it to trust what it finds in the request context, you may wish to override `onPopulateError()` and do something like log such failed attempts. See `populate()` below.

### public function onRequest( string targetPage )

Part of the standard CFML lifecycle, this method is called automatically by the CFML engine to handle each request. You should not override this (even tho' it is `public`).

If you override this method, you **must** call `super.onRequest( targetPage )`.

### public function onRequestEnd( string targetPage )

Part of the standard CFML lifecycle, this method is called automatically by the CFML engine at the end of each request. You should not override this (even tho' it is `public`). Use `setupResponse()` to perform request-specific finalization.

If you override this method, you **must** call `super.onRequestEnd( targetPage )`.

### public function onRequestStart( string targetPage )

Part of the standard CFML lifecycle, this method is called automatically by the CFML engine at the beginning of each request. You should not override this (even tho' it is `public`). Use `setupRequest()` to perform request-specific initialization.

If you override this method, you **must** call `super.onRequestStart( targetPage )`.

### public function onSessionStart()

Part of the standard CFML lifecycle, this method is called automatically by the CFML engine when each new session is created. You should not override this (even tho' it is `public`). Use `setupSession()` to perform session-specific initialization.

If you override this method, you **must** call `super.onSessionStart()`.

### public any function populate( any cfc, string keys = "", boolean trustKeys = false, boolean trim = false, boolean deep = false, any properties = '' )

Automatically populates an object with data from the request context, or from the `properties` struct if provided. For any public method `setKey()` on the object or any declared `property key;`, if that `key` exists in the source data structure, call the setter with that value. If the optional list of `keys` is provided, only attempt to call setters for the specified keys in the request context. Mainly useful for populating beans from form posts. Whitespace is permitted in the list of keys for clarity, e.g., `"firstname, lastname, email"`. This approach relies on setter methods and properties in the object. It won't detect and use `onMissingMethod()`.

You can also specify `trim = true` and FW/1 will call `trim()` on each item before calling the setter. Setting `deep = true` will allow FW/1 to populate nested properties on objects (in child objects).

For `populate()` to work with `onMissingMethod()` you need to specify `trustKeys = true`. If you specify the list of `keys`, `populate()` will not test whether the setter exists, it will just call it - which means that `onMissingMethod()` and property-based setters will be invoked automatically. If you omit the `keys`, be careful because `populate()` will cycle through the entire request context and attempt to set properties on the object for everything! A `try/catch` is used to suppress any exceptions in this case but you need to be aware that this may be a little dangerous if you have a lot of data in your request context that does not match the properties of the object! If an exception is caught, `onPopulateError()` is called with the object, property name and the request context structure as its three arguments. The default behavior is to simply ignore the exception but you can override that if you want - see `onPopulateError()` above.

`populate()` returns the `cfc` passed in.

#### Populating Child Objects

If `trustKeys = true` or if `deep = true` the `populate()` method will try to populate data on child objects of the `cfc` argument.  In order to populate a child component "dot" notation is used in the request context's properties.  For example, to set the `firstName` property on a `Contact` component that is a child of the `cfc` that is being passed into `populate()` the following key would be added to the request context:

    contact.firstname

This would cause `populate()` to traverse the `cfc` argument like so:

    cfc
    |-- getContact
    |    |-- setFirstName

Child properties can also be nested many levels deep. The following key would populate the `line1` property of an `address` CFC that is a child of the `contact` CFC that is a child of the `cfc` argument passed into the `populate()` method:

    contact.address.line1

This would cause `populate()` to traverse the `cfc` argument like so:

    cfc
    |-- getContact
    |    |-- getAddress
    |    |    |-- setLine1

### public struct function processRoutes( string path, array routes, string httpMethod = request._fw1.cgiRequestMethod )

Given a `path`, and an array of `routes`, and an optional `httpMethod`, process the routes to see if any matched and return a struct with a `matched` flag and if that's `true` also `route` and `path` values. Route matching is case-sensitive by default but this can be overridden by setting `routesCaseSensitive` to `false` in FW/1's configuration.

### public void function redirect( string action, string preserve = "none", string append = "none", string path = _see below_, string queryString = '', string statusCode = '302', string header = '' )

This constructs a URL based on the `action` and optional `path` and redirects to it. If `preserve` is `"all"`, the entire contents of the request context are saved to `session` scope across the redirect (and restored back to the request context automatically after the redirect). If `preserve` is a list of keys, just those elements of the request context are preserved. If `append` is `"all"`, all simple values in the request context are appended to the constructed URL as a query string before the redirect. If `append` is a list of keys, just those elements of the request context are appended (if they are simple values). The `statusCode` argument lets you specify the HTTP status code for the redirect so you can override the default value of `302`.

If `path` is specified, that base URL is used instead of the default, as per `buildURL()` above.

The `queryString` argument may be used to append additional URL variables / values to the constructed URL, as explained in `buildURL()` above. Or the query string value may be combined with the `action` also as explained in `buildURL()` above.

For example:

    variables.fw.redirect( action = 'blog.entry', append = 'id', queryString = '##comment' )

Will generate:

* `/index.cfm?action=blog.entry&id={rc.id}#comment` - in "traditional" mode
* `/index.cfm/blog/entry/id/{rc.id}#comment` - in "SES" mode
* `/blog/entry/id/{rc.id}#comment` - in "SES" mode with `SESOmitIndex` set to `true`

If `header` is provided (as a non-empty string), instead of performing an actual redirect, FW/1 will set the named HTTP header to the target URL and then abort the controller lifecycle. This allows custom handling of "redirects" in AJAX-based applications.

### Public void function redirectCustomURL( string uri, string preserve = 'none', string statusCode = '302', string header = '' )

This is to `redirect()` as `buildCustomURL()` is to `buildURL()`.

### public void function renderData( string type, any data, numeric statusCode = 200, jsonpCallback = "" )

Call this from your controller to tell FW/1 to skip views and layouts
and instead render `data` in the specified content `type` format. `type`
may be `"json"`, `"jsonp"`, `"rawjson"`, `"xml"`, or `"text"`.

For `"json"` and `"jsonp"`, FW/1 calls `serializeJSON( data )` to
generate the result of the HTTP request and sets the `Content-Type`
header to `application/javascript; charset=utf-8`.

For `"jsonp"`, you must provide a non-empty value for the
`jsonpCallback` argument. _New in 3.1._

For `"rawjson"`, the `data` value must be a string (and is assumed to be valid JSON already) and that is the result of the HTTP request. FW/1 sets the `Content-Type` header to `application/javascript; charset=utf-8`. _New in 3.1._

For `"xml"`, the `data` value must be either a valid XML string or an XML object (constructed via CFML's various `xml...()` functions). If `data` is an XML object, FW/1 calls `toString( data )` to generate the result of the HTTP request, otherwise the XML string is used as the result of the request. In both cases, FW/1 sets the `Content-Type` header to `text/xml; charset=utf-8`.

For `"text"`, the `data` value must be a string and that is the result of the HTTP request. FW/1 sets the `Content-Type` header to `text/plain; charset=utf-8`.

When you call `renderData()`, processing continues in your controller (so use `return;` if you want processing to stop at that point), and subsequent calls to `setView()` or `setLayout()` will have no effect (since FW/1 will ignore views and layouts for this request).

### public void function setBeanFactory( any factory )

If you are manually creating a bean factory, call this from your `setupApplication()` method to tell the framework about your primary (default) bean factory. By default FW/1 will use DI/1 as the bean factory and you won't have to worry about this.

### public void function setLayout( string action, boolean suppressOtherLayouts = false )

Call this to tell the framework to use a new action _subsystem:section.item_ as the basis of the lookup process for the layouts for the current request. This allows you to override the default convention for choosing the layouts. If you specify `suppressOtherLayouts` as `true`, then only the most specific layout will be used and the usual cascade of layouts will be turned off for this request.

### public void function setSubsystemBeanFactory( string subsystem, any factory )

Call this to tell the framework about a subsystem-specific bean factory. The bean factory must support `containsBean( name )` and `getBean( name )`. You would typically call this method from your `setupSubsystem()` method.

### public void function setupApplication()

Override this in your `Application.cfc` to provide application-specific initialization. If you want the framework to use a non-default bean factory, this is where you should call `setBeanFactory( factory )`. You do not need to call `super.setupApplication()`.

## public void function setupEnvironment( string env )

Override this in your `Application.cfc` to provide environment-specific initialization. See **Environment Control** in the [Developing Applications Manual](/documentation/3.5/developing-applications.html) for more detail.

### public void function setupRequest()

Override this in your `Application.cfc` to provide request-specific initialization. You do not need to call `super.setupRequest()`. Since you do not have access to `rc` here, you may also want to define `before()` in `Application.cfc` to act as an initialization controller, to populate the request context prior to other controllers being executed.

### public void function setupResponse( struct rc )

Override this in your `Application.cfc` to provide request-specific finalization. This is called after all views and layouts have been rendered or immediately before a redirect. You do not need to call `super.setupResponse()`. 

### public void function setupSession()

Override this in your `Application.cfc` to provide session-specific initialization. You do not need to call `super.setupSession()`.

### public void function setupSubsystem( string subsystem )

Override this in your `Application.cfc` to provide subsystem-specific initialization. If you want the framework to use non-default subsystem-specific bean factories for any subsystems, this is where you should call `setSubsystemBeanFactory( subsystem, factory )`. See the example in [Using Subsystems](/documentation/3.5/using-subsystems.html) for more details. You do not need to call `super.setupSubsystem()`.

### public void function setupTraceRender( string output = 'html' )

This is called when the framework trace is about to be rendered at the end of a request. `output` will be either `'html'` or `'data'` depending on whether FW/1 has output HTML using views and layouts or rendered data in the current request. You can override it to take control of the rendering process yourself (for whatever reason such as saving the trace data to a database perhaps or providing a fancier rendering?). You can call `getFrameworkTrace()` to obtain the framework's trace data (note that this will be a copy on Adobe ColdFusion but just a reference on Lucee and Railo), and do whatever you want with it. It's probably a good idea to call `disableFrameworkTrace()` to prevent any further additions to the framework trace data. Note: this method used to be called only when FW/1 was outputting HTML and not rendering data, however, it is now called whenever tracing is enabled. If you were using this method to render trace data in a custom format at the end of HTML requests, you will find your rendered trace data appended to your request data when `renderData()` is used. In order to maintain your current functionality, you can easily fix this by checking that the `output` argument is equal to `'html'` before rendering the trace data.

### public void function setupView( struct rc )

Override this in your `Application.cfc` to provide pre-rendering logic, e.g., putting globally available data into the request context so it is available to all views. You do not need to call `super.setupView()`. 

### public void function setView( string action )

Call this to tell the framework to use a new action _subsystem:section.item_ as the basis of the lookup process for the view and layouts for the current request. This allows you to override the default convention for choosing the view and layouts. A possible use for this is when redisplaying a form view when errors are present (i.e., from the form processing controller method, without using a redirect).

### public boolean function usingSubsystems()

Returns `true` if the application is using subsystems, i.e., `variables.framework.usingSubsystems` is `true`. Otherwise returns `false`.

### public string function view( string path, struct args = { }, any missingView = { } )

This renders a view and returns the output of that view as a string. It is intended to be used primarily inside layouts to render fragments of a page such as menus and other common elements. Elements of the *args* structure are appended to the local scope accessible inside the view file. For example:

    <cfoutput>
      <div>#view( 'common:site/header' )#</div>
      <div>#view( 'nav/menu', { selected = 'home' } )#</div>
      <div>#body#</div>
      <div>#view( 'common:site/footer' )#</div>
    </cfoutput>

This renders the `header` and `footer` items (views) from the `common` subsystem's `site` section and the `menu` item (view) from the current subsystem's `nav` section. Inside `menu.cfm`, `local.selected` would be available containing the string `"home"`.

A controller may call `view()` which can be useful if you have email templates that need to be rendered and sent as part of a request: those email templates can be treated as views and have all the associated `rc`, `local`, etc machinery applied.

If the argument `missingView` is not specified, and the specified view `path` does not exist, then `onMissingView()` will be called. If a string is passed as `missingView` and the specified view does not exist, then the value of the `missingView` argument will be returned. This allows for programmatically calculated views to be silently rendered as empty strings if they are not present. This can be useful for programmatic skins with optional elements.
