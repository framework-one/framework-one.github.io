---
layout: page
title: "Using DI/1"
date: 2016-04-07 11:30
comments: false
sharing: false
footer: true
---
_This is documentation for the upcoming 4.0 release. For the current release, see [this documentation](/documentation/)._

DI/1 - a.k.a Inject One - is a simple, convention-based Dependency Injection framework. 

DI/1 searches specified directories for CFCs and treats them as singletons or non-singletons (transients) based on naming conventions for the CFCs themselves, or the folders in which they are found. You can override the conventions by configuration if needed.

As of release 3.5, DI/1 also looks for `.lc` and `.lucee` files, as well as `.cfc` files, to support Lucee 5's new dialect.

### Terminology
- **Bean**: A CFC that you want to create. Any file with a .cfc extension can be a bean
- **Transient** or **non-singleton** : A bean that will freshly created each time you call `getBean()`, it could be used for only the lifespan of the request, e.g. a basket object ready to be populated with items.
- **Singleton**: A cfc that only one exists in the system, each time you call `getBean()` you will get the SAME bean, not a new one, for example a Service that creates basket objects.
- **Bean Factory**: A service that creates beans for you, so you don't have to use `new` or `createObject`, and populates them with any dependencies.


* TOC
{:toc}

### Terminology
- **Bean**: A CFC that you want to create. Any file with a .cfc extension can be a bean
- **Transient** or **non-singleton** : A bean that will freshly created each time you call `getBean()`, it could be used for only the lifespan of the request, e.g. a basket object ready to be populated with items.
- **Singleton**: A cfc that only one exists in the system, each time you call `getBean()` you will get the SAME bean, not a new one, for example a Service that creates basket objects.
- **Bean Factory**: A service that creates beans for you, so you don't have to use `new` or `createObject`, and populates them with any dependencies.

# Getting Started with DI/1

Most users will be using DI/1 as the default bean factory for FW/1, but you can also use DI/1 in a non-FW/1 application, and that can be a good way to start adding structure and automation to an existing legacy project if you're not ready for an MVC framework. Both approaches will be covered below.

## Getting Started with DI/1 and FW/1

By default, FW/1 creates an instance of DI/1 to use as its bean factory, to manage controller CFCs and also CFCs that are part of your application model. You control how FW/1 uses DI/1 through the `diEngine`, `diComponent`, `diLocations`, and `diConfig` settings.

You should only need to call `getBean()` to get a transient -- by default, a CFC found in a folder called `beans`:

    var user = fw.getBeanFactory().getBean( "user" ); // or "userBean"

CFCs found in other folders (e.g., `services`) are treated as singletons and will be autowired into each other, based on `property` declarations (if you have `accessors=true` on your `component`), `setXxx()` methods, and constructor arguments (`init()`). Read [Using Bean Factories](developing-applications.html#using-bean-factories) in the Developing Applications Guide for more detail.

## Getting Started with DI/1 Standalone

If you want to use DI/1 outside of FW/1, here's how you should do it: create an instance of the DI/1 bean factory and specify the folder(s) you want it to search for beans (CFCs):

    var beanFactory = new framework.ioc("/model");
    // or multiple folders:
    var beanFactory = new framework.ioc("/model,/common/model");
    // or an array:
    var beanFactory = new framework.ioc(["/model", "/common/model"]);

If CFC names are unique, you can use that name to get the bean out of the factory:

    var userManager = beanFactory.getBean("userManager");

See below for how DI/1 handles CFCs that have the same name, found in different folders.

## Basic DI/1 Conventions

CFCs found in a folder called `beans` are assumed to be transients; otherwise CFCs are assumed to be singletons, this includes beans found in folders under the `beans` folder. A singleton has just a single instance and DI/1 will cache that instance. A transient is created afresh every time you ask DI/1 for an instance.

The name of a bean is the name of the CFC (without the path information or file extension). All beans are also given an alias which is the name of the CFC followed by (the singular form of) the folder name in which it was found, e.g., `/model/beans/product.cfc` would get the alias `"productBean"`. If no other CFC is called `product.cfc` in the folders that you asked DI/1 to search, you can use `"product"` or `"productBean"` to reference that bean (in your `property` declarations or `getBean()` calls).

If a CFC has a constructor (a method called `init()`), DI/1 will use the argument names to look up beans and call the constructor with those beans. If a CFC has setter methods, DI/1 will use their names to look up beans and call the setters with those beans. If a CFC has `property` declarations and implicit setters are enabled (`accessors=true` on `component`), DI/1 will use their names to look up beans and call the implicit setters with those beans. This is called autowiring. By the time you get a bean back from DI/1, it should be fully populated. You can also specify an `"init-method"` function name that DI/1 should call after a bean has had its dependencies injected - see **[Configuration](#configuration)** below. When using `property` to declare a dependency, do not specify a `type` or a `default`: DI/1 assumes that typed properties (and defaulted properties) are intended to generate specific getters and setters on transients or for ORM integration, rather than just dependencies. You can override this default behavior - see **[Configuration](#configuration)** below.

    // usermanager - managers/user.cfc or usermanager.cfc
    component accessors=true {
    
        property roleService; // autowire services/role.cfc
        
        function setLoggingService( loggingService ) { // autorwire services/logging.cfc
            variables.logger = loggingService.getLogger( "user" );
        }
        
        function init( userdao ) { // autowire daos/user.cfc
            variables.userdao = userdao;
            return this;
        }
        
    }

When you get this `usermanager` bean from DI/1 -- either by calling `getBean( "usermanager" )` directly or autowired into another bean via `property usermanager;` (or a setter or constructor argument), it will already have `roleService`, `loggingService`, and `userDAO` autowired into it.

If DI/1 cannot find a matching bean for a constructor argument, it will throw an exception. If DI/1 cannot find a matching bean for a setter method or property, it will log the failure and ignore it (by default), and the corresponding variable will not be populated. You can configure DI/1 to be strict about matching bean names - see the configuration section below - in which case it will throw an exception.

As of FW/1 4.0 (DI/1 1.2), you can specify a second argument to `getBean()` that provides constructor arguments that should be used instead of beans in the factory:

    var user = fw.getBeanFactory().getBean( "user", { name : "Sean", email : "sean@corfield.org" } );

This will use `name` and `email` as overrides so that they _hide_ any beans of the same name when DI/1 calls the `init()` method. This can be particularly valuable when you are migrating legacy code to DI/1 and want it to manage bean creation while still providing constructor arguments in the (legacy) code.

Note that DI/1 will inject only singletons via setters or properties. Injecting transients in those situations often leads to unexpected results (consider a transient `invoice` bean that has a `setCustomer()` method when you also have a transient `customer` bean - you almost certainly don't want DI/1 to automatically create a customer instance and inject it every time you ask DI/1 for a new invoice bean!). If a constructor argument matches a transient bean, DI/1 will still create an instance since it has to finish constructing the original bean. _Note: this is unclear and there is an open issue against the documentation to reword this once some testing has verified exactly how DI/1 behaves with combinations of transients and singletons in constructor arguments!_

## Acceptable Folder Paths

In general, you should use webroot-relative folders - starting with `/` - or mappings - also starting with `/` - as the constructor arguments to `ioc`. If you pass a full file system path, DI/1 will only be able to deduce the dotted-name of CFCs found there if it points into the webroot tree. Similarly, if you pass a relative folder path, it must point into the webroot tree. If DI/1 cannot deduce the dotted name of a CFC, it will throw an exception.

# Beyond Convention

For many applications, DI/1's default conventions will be sufficient but it also supports more advanced usage through three avenues:

* Configuration -- a `config` struct that can be provided to the `ioc()` constructor. In a FW/1 application, this is specified as `diConfig` in the framework settings.
* Customization -- several public methods that can add additional beans to the factory.
* Overriding -- extending `ioc.cfc` to provide your own versions of several key methods. In a FW/1 application, you will be able to specify your extended CFC using the `diComponent` framework setting.

Each of these approaches will be discussed in the following sections.

## Configuration

When the bean factory is created, you can optionally supply a struct containing configuration for DI/1. At present, DI/1 understands the follow configuration options:

* `constants` - struct - defaults to `{}`. DI/1 will use any name/value pairs specified here to provide _beans_ that resolve to the specified values. This can be used to provide resolution for constructor arguments that need values which are not actual beans. See also `declare()` in the **Customization** section that follows.
* `exclude` - array - defaults to `[]`. DI/1 will ignore any CFCs whose file path contains the strings in this array. DI/1 always excludes paths containing `/WEB-INF` and `/Application.cfc`, as well as various FW/1 and DI/1 framework files. The strings are not case-sensitive.
* `initMethod` - string - If specified, identifies a method name on beans that DI/1 will attempt to call (with no arguments) on each bean after its dependencies have been injected.
* `liberal` - boolean - default to `false`. If `true`, treat folder names ending in `ies` as plurals (of names ending in `y`, e.g., `libraries` would be treated as the plural of `librarie` by default, but with `liberal : true`, it would be treated as the plural of `library`).
* `loadListener` - any - If specified, DI/1 will register it as a load listener by calling `this.onLoad( loadListener )`. It can be a CFC instance, the name of a bean, or a function or closure.
* `omitDirectoryAliases` - boolean - defaults to `false`. If `true`, use CFC names as bean names directly, without appending the singular directory name as a suffix. If your CFC names are not unique, you will get an exception.
* `omitDefaultedProperties` - boolean - defaults to `true`. If `false`, property declarations that specify a default will still be treated as dependency declarations and autowired. If `true`, property declarations that specify a default will be ignored for injection. This is useful when you are using `property` declarations on transients solely for generating setters and getters, rather than for declaring dependencies. _New in 4.0._
* `omitTypedProperties` - boolean - defaults to `true`. If `false`, property declarations that specify a type (other than `any`) will still be treated as dependency declarations and autowired. If `true`, property declarations that specify a type will be ignored for injection. This is useful when you are working with the ORM (since those property declarations will typeically have types and should not be treated as dependencies). _The default changed from `false` to `true` in 4.0._
* `recurse` - boolean - defaults to `true`. Controls whether DI/1 searches subfolders recursively or not.
* `singletonPattern` - string - no default. Specifies a regular expression that DI/1 uses to determine whether a bean is singleton or not, based on its name. The `beans` folder convention and the `transients` configuration below still apply so nothing in those folders will be considered a singleton, even if its name matches the pattern.
* `singulars` - struct - defaults to `{}`. DI/1 will use any name/value pairs specified here to translate folder names to a singular variety, e.g., `pride = 'lion'` will convert the *plural* folder `pride` to the *singular* name `lion` and therefore a `simba.cfc` within the `pride` folder will get the alias `simbaLion`. This also allows for other folders to behave as if they were called `beans` by treating their singular name as `bean`. One of the DI/1 unit tests maps `sheep` to `bean` for this reason. This won't work if the CFCs in `sheep` have the same name as the CFCs in a `beans` folder however.
* `strict` - boolean - defaults to `false`. If `true`, DI/1 will throw an exception if it cannot resolve a bean implied by a constructor argument, setter name or property name. If `false`, DI/1 simply calls `logMissingBean()` which writes the failure to the Java console. See `missingBean()` in **Overriding DI/1 Behavior** below for more details.
* `transients` - array - defaults to `[]`. DI/1 will consider any CFCs found in these folders to be transient, rather than singleton. The conversion to a singular form will still take place to create the alias for each CFC. For example, if `singulars = { pride = 'lion' }` and `transients = [ 'pride' ]` then any CFCs in the `pride` folder will be treated as transients and their alias will end in `Lion`.
* `transientPattern` - string - no default. Specifies a regular expression that DI/1 uses to determine whether a bean is transient or not, based on its name. The `beans` folder convention and the `transients` configuration below still apply so CFCs in those folders will be still considered transients, in addition to any name that matches the pattern.

The examples below all show `diConfig` for FW/1 (and `diLocations` in some examples). If you are using DI/1 standalone, you can imagine creating the instance of DI/1 like this, in order to map those examples to your usage:

    var beanFactory = new framework.ioc( diLocations, diConfig );

### Configuring "Constant" Beans

The `constants` config element is a struct containing mappings from bean names to specific constant values. This allows you to specify non-CFC values for constructor arguments, setters and properties (but is most commonly used for constructor arguments). The value may be of any type and any reference to that bean name will return the specified value as a singleton.

These values may also be added after DI/1 has been initialized using the `declare()` method as shown below (in **Customization**).

### Specifying Additional Transient Beans

By default, any CFC in the `beans` folder is considered a transient and everything else is considered a singleton. There are three ways to specify other CFCs should be considered transient:

* `config.singulars` allows you to specify irregular plural folder names and their singular mappings, e.g., `geese : "goose"`
* `config.transients` allows you to specify additional folders whose contents are transient (i.e., in addition to the `beans` folder convention)
* `config.singletonPattern` allows you to specify a regular expression which limits which beans are considered singletons
* `config.transientPattern` allows you to specify a regular expression which limits which beans are considered transients

For `config.singulars`, any folder name whose singular name is `bean` will cause CFCs to get an alias that ends in `Bean` and will be considered transients (see below for examples). For `config.transients`, the singular transformation will still be applied to create the alias, but the CFCs will be considered transients anyway. For `config.singletonPattern`, CFCs will also be considered transients if their name **does not match** the regular expression pattern supplied. For `config.transientPattern`, CFCs will also be considered transients if their name **does match** the regular expression pattern supplied. You cannot specify both `config.singletonPattern` and `config.transientPattern`.

For example:

    diConfig : { singulars : { objects : "bean" }, transients : [ "models" ] }

This will cause CFCs found in the `objects` folder to be treated as if they were in the `beans` folder (their alias will end with `Bean` and they will be considered transients because of that) and CFCs found in the `models` folder to be treated as transients too (but their alias will end with `Model`, the singular of `models`).

    diConfig : { singulars : { services : "manager" }, transients : [ "objects" ] }

This, on the other hand, will cause CFCs found in the `services` folder to be treated as if they were in the `managers` folder (their alias will end with `Manager` and they will be considered singletons because of that) and CFCs found in the `objects` folder to be treated as transients (their alias will end with `Object`, the singular of `objects`).

    diConfig :{ singletonPattern : "(Service|Factory)$" }

In addition to any CFCs found in a folder called `beans`, any CFC whose name does not end in `Service` or `Factory` will be considered a transient.

    diConfig : { transientPattern : "(Entity)$" }

In addition to any CFCs found in a folder called `beans`, any CFC whose name ends in `Entity` will be considered a transient.

## Customization

In addition to the configuration that lets you specify additional "constant" beans and define how the conventions work, DI/1 provides four ways to programmatically add beans to the bean factory:

* Add an alias for a bean. This can be useful if the deduced name of a bean (based on the file system path) doesn't match a property that you want injected. You can add an alias that matches the property name, and DI/1 will resolve it to the underlying bean.
* Add a named value as a bean. This can be any constant, data structure, or object. This can be useful when you want to add properties whose values have to be computed in a load listener, rather than supplied as "constants" as part of DI/1's configuration.
* Declare a bean based on a CFC that is outside the folders you asked DI/1 to manage for you. When asked for this bean DI/1 will create an instance, calling the constructor, and injecting any declared properties. You can choose whether these instances are singletons or transients. You can also provide overrides for named constructor arguments and properties.
* Declare that a bean should be obtained from a (another) factory (which, itself, can be one of your managed beans or an object you created directly). You specify the fectory, the method to call on it, and the names of beans that should be passed as the arguments to that method. Like the declaration approach above, you can provide overrides for those beans.

There is a corresponding method on DI/1 for each of these, but as of FW/1 4.0 (DI/1 1.2), there is a better way to declare beans to the bean factory, using a new "builder" syntax:

    beanFactory.declare( beanName ).aliasFor( existingBean );
    beanFactory.declare( beanName ).asValue( beanValue );
    beanFactory.declare( beanName ).instanceOf( dottedPath );
    beanFactory.declare( beanName ).fromFactory( factory, methodName );

For the last two declarations, you can also call `.asSingleton()` or `.asTransient()` to specify whether the bean should be a singleton or transient. Also for the last two declarations, you can call `.withOverrides()` to specify a struct containing bean / value pairs that should be used for constructor arguments and property injections, instead of existing beans in the factory.

For the factory declaration, you can call `.withArguments()` to specify an array of bean names that should be looked up and passed as arguments to the `factory`'s `methodName`.

All of the above declarations return the declaration itself, so you can chain the modifier functions:

    beanFactory.declare( "generated" ).fromFactory( factory, "gen" )
        .withArguments( [ "rand256", "gaussDistStrategy" ] );

When DI/1 is asked for the `"generated"` bean, it will call the `gen()` method on the `factory`, passing two arguments: the values of the beans `"rand256"` and `"gaussDistStrategy"` respectively.

In addition, you can call `.done()` on a declaration to get the bean factory back so that you can chain declarations:

    beanFactory.declare( "abbrev" ).aliasFor( "longBeanName" ).done()
        .declare( "answer" ).asValue( 42 ).done()
        .declare( "copyright" ).asValue( 2016 );

See **Customization Examples** below for more examples and explanations.

Here are the four direct methods, as present in earlier versions of DI/1 (these may be deprecated in a future release):

* `addAlias( beanName, existingBean )` -- tells DI/1 that `beanName` should resolve to the same bean as `existingBean`.
* `addbean( beanName, beanValue )` -- tells DI/1 that `beanName` should resolve to the specified `beanValue` (which can be a constant or a data structure or an object).
* `declareBean( beanName, dottedPath, isSingleton, overrides )` -- tells DI/1 that `beanName` should resolve to an instance of the specified `dottedPath`, and whether it should be treated as a singleton or a transient. In addition, you can provide specific bean/value pairs to be used for the construction and autowiring of that bean, which will override any beans known to the bean factory.
* `factoryBean( beanName, factory, methodName, args, overrides )` -- tells DI/1 that `beanName` should be resolved by calling `methodName` on the `factory` object and passing arguments looked up by name (`args` contains an array of bean names to use). As with `declareBean()`, you can provide bean/value pairs for construction and autowiring.
As of FW/1 4.0, the `factory` can be a function or closure (and `method` omitted).

The recommended way to perform this programmatic customization is inside a load listener. A load listener is a function, closure, or method that accepts a bean factory as an argument and performs all the customization you need on that bean factory. You can register a load listener either by specifying it in the `config` struct, above, as `loadListener`, or by calling `onLoad( listener )` on the bean factory itself.

* If `loadListener` is a CFC instance, DI/1 will call `loadListener.onLoad( beanFactory )`, passing the DI/1 instance in as an argument.
* If `loadListener` is a bean name, DI/1 will call `beanFactory.getBean( loadListener ).onLoad( beanFactory )`, where `beanFactory` is the DI/1 instance.
* If `loadListener` is a function or closure, DI/1 will call `loadListener( beanFactory )`. Note that if `loadListener` is a method on a CFC, it will be called out of context so it will not have access to the `variables` scope or `this` scope of that CFC instance and therefore also won't have access to other methods of that CFC.

See the next section for examples of customizing the bean factory.

### Customization Examples

You can add an alias for a bean:

    beanFactory.declare("alsoKnownAs").aliasFor("navigation");

That will tell DI/1 that `alsoKnownAs` is an alias for the bean identified by `navigation` so `getBean("alsoKnownAs")` will behave the same as `getBean("navigation")`.

You can programmatically add new bean instances - or named values:

    beanFactory.declare("magicvalue").asValue(42);
    beanFactory.declare("logger").asValue(new LogFactory("log4j"));

After these calls, `getBean("magicvalue")` will return the value 42 and `getBean("logger")` will return the CFC instance you provided. That means that any properties, setter methods or constructor arguments that refer to `magicvalue` or `logger` will get those values injected.

You can also programmatically declare new beans to be managed by DI/1:

    beanFactory.declare("navigation").instanceOf("site.utils.navigation");

That will tell DI/1 that `/site/utils/navigation.cfc` should be managed as a singleton with name `navigation`. You can declare transients by adding a call to `.asTransient()`.

When declaring a bean, you can also optionally provide a set of overrides for named beans, so that constructor arguments or properties will take on specified values, rather than what is managed by the bean factory. This is useful for creating variants of a single bean:

    beanFactory.declare("datasource").instanceOf("util.DataSource")
        .withOverrides( { dsn = "main" } );
    beanFactory.declare("admindata").instanceOf("util.DataSource")
        .withOverrides( { dsn = "admindb" } );

You can declare a factory bean - like Spring/ColdSpring - as follow:

    beanFactory.declare("generated").fromFactory(factory, "method")
        .withArguments( [ ..args.. ] ).withOverrides( { ... } );

This tells DI/1 that when you call `getBean("generated")`, instead of trying to create the bean itself, it should call `factory.method(..args..)` to get the bean instance. If you don't call `.withArguments()` then the method is called with no arguments.

### Using Load Listeners

The easiest way to organize all your customization of the DI/1 bean factory is to use a load listener. I prefer to have a CFC, called `LoadListener.cfc`, somewhere in my `/model` folder that contains an `onLoad()` method, and I declare this in the `config` struct when I create DI/1:

    // if letting FW/1 create DI/1 for you:
    variables.framework = {
        ...
        diLocations : [ "/model", "/controllers" ],
        diConfig : { loadListener : "LoadListener" },
        ...
    };

    // if creating DI/1 explicitly:
    var bf = new framework.ioc( [ "/model", "/controllers" ], { loadListener : "LoadListener" } );

This tells DI/1 that when it has first discovered all the beans in the folders you specified (after construction but before any further operations take place on the bean factory), it should call `getBean( "LoadListener" )` and then call `onLoad( this )` on that bean (i.e., passing itself into your method).

Your load listener CFC would look like this:

    // LoadListener.cfc:
    component {
        function onLoad( beanFactory ) {
            beanFactory.declare( ... ).asValue( ... ).done()
                .declare( ... ).asValue( ... ).done()
                .declare( ... ).instanceOf( ... ).done()
                ...done()
                // eagerly load all the singletons:
                .load();
        }
    }

That last step is optional, but I like to avoid lazy loading of singletons in applications that see heavy load (DI/1 deliberately avoids locks so heavy load can cause singletons to be constructed multiple times if you don't eagerly load the singletons at startup).

If you are using DI/1 standalone, you can also manually add load listeners by calling `onLoad()` on the bean factory itself, as long as you do it before any other operations are performed:

    // only when creating DI/1 explicitly:
    var bf = new framework.ioc( [ "/model", "/controllers" ] );
    bf.onLoad( "LoadListener" );

If you let FW/1 manage your DI/1 bean factory for you, you need to specify the load listener as part of the configuration (`diConfig`) as shown above.

Note that you cannot add load listeners inside your load listener method itself! If you want to add more load listeners, your best choice is to extend `ioc.cfc` and in your `init()` function, after calling `super.init()`, call `this.onLoad()` to register them before you `return this;`. You can easily tell FW/1 to use you extended version of DI/1 with the `variables.framework.diComponent` setting.

Note that a load listener can also be passed as a function or closure, or as a CFC instance, so if you only wanted to eagerly load the singletons, you could declare it inline like this:

    variables.framework = {
        ...
        diConfig : { loadListener : function( di1 ) { di1.load(); } },
        ...
    };

## Overriding

Customizing the behavior of DI/1 by overriding its methods should probably be considered a "last resort" so before you go down this path, ask on Slack or on the mailing list if there is a way to achieve your goals without doing this. As an example of overridden behavior, the Clojure integration provided by `ioclj.cfc` could be a useful model for you. That overrides the constructor (to deal with finding Clojure code and modifying the metadata) and `getBeanInfo()` to provide metadata about Clojure code that is loaded.

DI/1 provides no specific public extension points but it does provide a few `private` extension points that are considered documented and supported. These are described in detail near the end of this document and they are primarily intended to allow you to customized how CFC instances are actually constructed, how metadata is obtained, and what to do if DI/1 cannot locate a bean that you have requested.

In addition, there is a hook for modifying beans as they are initialized -- `setupInitMethod()` -- which is called after a bean has been fully populated but prior to calling the `"init-method"` (if any is specified).

See **[Overriding DI/1 Behavior](#overriding-di1-behavior)** for more detail.

# Other Public Methods

This section describes some of the other methods that DI/1 provides, which applications may find useful. See the **Reference Manual** section below for full details of the function signatures.

Given a struct of values (such as form scope or URL scope), you can ask DI/1 to inject those values as properties into a given bean:

    bean = beanFactory.injectProperties(myBeanInstance, form);
    user = beanFactory.injectProperties("user", userAttributes);

The first call will loop over the form scope and, for each key in that scope, call a setter on `myBeanInstance`. The second call asks DI/1 to create a `user` bean and populate it by calling a setter for each element of the struct `userAttributes`. You may also use a dotted-path to a CFC as the first argument in which case DI/1 will use `createObject` to instantiate it and *will not call the constructor*. Caution: DI/1 assumes you know what you're doing and will call a setter for *every* member of the struct passed in!

You can ask if the bean factory knows about a particular bean using the `containsBean()` method:

    if ( beanFactory.containsBean("productService") ) ...

(although you probably shouldn't need to do this unless you are building some sort of framework plugin that needs to check what is available to it at runtime!).

You can force all singletons to be reloaded using the `load()` method:

    beanFactory.load();

That will empty the bean cache and then call `getBean()` on every bean that DI/1 knows about. Note: it does not call `load()` on any parent bean factory (see below) and it does not perform a new search on the folders (so it won't see newly written CFCs). To force the search to be performed again, create a new instance of the bean factory as shown above.

Metadata can be queried using the following methods:

    if ( beanFactory.isSingleton("someBean") ) ...
    info = beanFactory.getBeanInfo("someBean");
    if ( beanFactory.hasParent() ) ...

I would expect these only to be useful to framework authors. The first two methods walk up into parent bean factories, if present, the third indicates whether the bean factory has a parent. If you omit the bean name for `getBeanInfo()` you get back a struct with a key `beanInfo` that refers to metadata for all of the beans known in the factory. If there is a parent bean factory, its metadata is returned under a key `parent` in that struct.

`getBeanInfo()` can be called with a `beanName` argument - the default - or with a `regex` argument which will return metadata about all the beans in the factory whose names match the regular expression, in a struct with the single key `beanInfo`, whose value will be a struct with a key for each matching bean.

`getBeanInfo()` can also be called with no arguments, in which case it will return metadata for all the beans in the factory (in the `beanInfo` key of the result) and metadata for all the beans in the factory's parent, if any, in the `parent` key of the result. Optionally, you may specify an argument of `flatten = true` and the `parent` structures will be merged (recursively through the parents) into `beanInfo`, producing a flat struct.

`getConfig()` can be called to get a copy of the bean factory's configuration, in case you need to have conditional behavior in your load listeners.

# Parent Bean Factories

If your application is assembled from multiple modules, you may have a main bean factory containing shared CFCs and each module may also have a bean factory. You can tell a module's bean factory about the shared CFCs in the main bean factory using the `setParent()` method:

    var moduleBeanFactory = new framework.ioc("/moduleModel");
    moduleBeanFactory.setParent( mainBeanFactory );

This causes DI/1 to ask its parent bean factory about any beans that are requested but unknown (within the moduleBeanFactory). Because DI/1 uses only `containsBean(name)` and `getBean(name)` the parent bean factory does not need to be another DI/1 instance - it can be any bean factory that provides that API.

Note that this is done automatically by FW/1 in an application with subsystems: each subsystem has its own bean factory with the main application's bean factory as the parent.

# Bean Factory Aware

If you need access to the bean factory itself within one of your CFCs, either declare a constructor argument called `beanFactory`, provide a `setBeanFactory( any beanFactory )` setter or declare `property beanFactory;` (with implicit setters enabled). DI/1 declares itself as a bean called `beanFactory` and will inject itself where any such dependencies appear.

# Reference Manual

This section lists every public method in DI/1, along with a brief explanation. After the listing of public methods, some additional discussion is provided about extending DI/1 and overriding its behavior.

## Public Methods

This section describes the supported public API of DI/1.

### public any function init( any folders, struct config = { } )

The constructor. `folders` can be a list or array of paths to search for CFCs to manage. It is recommended to use webroot-relative paths or mappings (both starting with `/`) for the paths. The optional `config` struct can supply various configuration settings to override the default behavior.

### public any function addAlias( string aliasName, string beanName )

Add an alias for a given bean name. Prefer the new "builder" syntax of the `declare()` method.

### public any function addBean( string beanName, any beanValue )

Tell DI/1 that the given bean name should resolve to the supplied value. The value may be any type of data. Prefer the new "builder" syntax of the `declare()` method.

### public boolean function containsBean( string beanName )

Returns `true` if DI/1 knows about the given bean name. If DI/1 doesn't know about it directly, but has been given a parent bean factory, it will ask the parent (by calling `containsBean()` on the parent).

### public boolean function hasParent()

Returns `true` if DI/1 has been given a parent bean factory.

### public any function declare( string beanName )

Declare a new bean in the bean factory. Returns a "builder" on which you can call `aliasFor`, `asValue`, `instanceOf`, `fromFactory`, `asSingleton`, `asTransient`, `withArguments`, and/or `withOverrides`.

### public any function declareBean( string beanName, string dottedPath, boolean isSingleton = true, struct overrides = { } )

Tell DI/1 that the given bean name should resolve to an instance of the named CFC. Prefer the new "builder" syntax of the `declare()` method.

### public any function factoryBean( string beanName, any factory, string methodName = "", array args = [ ], struct overrides = { } )

Tell DI/1 that the given bean name should be resolved by called the specified `methodName` on the `factory` bean, with the specified arguments (`args` by name). Prefer the new "builder" syntax of the `declare()` method.

### public any function getBean( string beanName, struct constructorArgs = { } )

Ask DI/1 for the specified bean. If `constructArgs` are provided, they are treated as overrides and will _hide_ any beans of the same name that DI/1 is managing, just for the initialization of the requested bean.

### public any function getBeanInfo( string beanName = '', boolean flatten = false, string regex = '' )

Return metadata about the named bean. If no bean name is given, return a struct containing metadata about all beans that DI/1 knows about. By default, any parent bean factory's metadata is returned as a nested struct but `flatten` will recursively unroll the chain of parent metadata (so the `structKeyList()` of the returned metadata will be all beans, not just the ones in the child bean factory). In addition, you can specify a `regex` to return metadata just for beans whose name matches the regular expression.

### public struct function getConfig()

Return a (shallow) copy of DI/1's configuration struct, with all the defaults populated.

### public string function getVersion()

Return a string indicating the version of DI/1 being used.

### public boolean function isSingleton( string beanName )

Return `true` if the specified bean is known to be a singleton. If the bean is known to be a non-singleton, this will return `false`. If the bean is not known in the current bean factory but a parent bean factory exists, the parent will be asked `isSingleton(beanName)`. If the bean is not known at all (or asking the parent fails), this will return `false`.

### public any function injectProperties( any bean, struct properties )

Given a bean (either a name, a dotted path to a CFC, or an actual instance), inject each of the (non-null) `properties` specified by calling the matching setter on the bean. If a bean name is given, it will be fully constructed by DI/1 first, and then populated. If a dotted path to a CFC is given, it will be created -- but its constructor will **not** be called -- and then it will be populated. If an actual bean instance is given, it will be populated. If calling any setter fails, that exception will propagate and other properties of the bean will not be populated. In general, use of this method is not recommended as it is not very flexible and you need to know what you are doing!

### public any function load()

This causes DI/1 to flush all its caches and to go through all the singleton beans it knows about and force full resolution of each. This can be used to reload a bean factory (although it is safer to just create a new instance of DI/1 and start over, because `load()` does not reload any parent bean factories), but the most common usage is to avoid lazy loading of beans (e.g., if you want to "warm up" your application after a deployment before putting it back in the cluster). Calling `load()` at the end of a load listener will also avoid any potential thread safety issues caused by lazy loading beans under heavy load in an application.

### public any function onLoad( any listener )

Register a load listener that DI/1 should call after the factory has initialized itself. This is the recommanded way to provide additional bean declarations since it ensures they all run before the constructed instance of DI/1 is returned to your program. The `listener` may be a bean name, in which case DI/1 will look it up internally, or an instance of a CFC, or a user-defined function or closure. If the `listener` resolves to a CFC instance, DI/1 will call `onLoad()` on that instance and pass itself in as the only argument. If the `listener` is a function or closure, DI/1 will call it and pass itself in as the only argument. You can then manipulate the bean factory further as part of its initialization. A single load listener is usually registered via the `loadListener` element of the `config` struct when DI/1 is initialized but `onLoad()` can be called multiple times to register additional listeners. When muliple listeners are registered, they are called in the reverse order of registration.

### public any function setParent( any parent )

Tell DI/1 that the given `parent` object should be treated as a parent bean factory. Several calls in DI/1 will delegate to a parent bean factory if the specified information is not present in the child bean factory. This allows common beans to be shared with several child factories, for example with a main application that has subsystems.

## Overriding DI/1 Behavior

If you want to override the methods in DI/1, such as `logMissingBean()`, you can create your own CFC that extends `ioc.cfc` and overrides the desired methods. Then use your CFC instead of `ioc.cfc`. If any particular use case becomes common, we can discuss incorporating it into DI/1 as a configuration option.

One possible use case is overriding the constructor to provide your own `init()` method that does additional configurtion (although using a load listener is probably a better way to do this in general). The Clojure integration provided by `ioclj.cfc` takes this approach because it needs to do additional setup work, as well as registering its own load listener (in addition to any `config.loadListener`), so that it can modify the bean metadata at startup, after the (CFC) beans have been discovered by DI/1.

The following supported extension points are provided:

* `private void function setupInitMethod( string name, any bean )` - this is called for each bean after its dependencies have been injected prior to calling `initMethod` (if specified).
* `private any function construct( string dottedPath )` - this is called to construct each CFC: the default implementation is `return createObject( "component", dottedPath );`.
* `private any function metadata( string dottedPath )` - this is called to obtain the metdata for each CFC: the default implementation is `return getComponentMetadata( dottedPath );` although it wraps that in `try/catch` and attempts to provide a more useful exception message in the case that `getComponentMetadata()` fails. An example from Adam Tuttle is the ability to silently ignore beans that have syntax errors during development, so the rest of the beans are loaded: you would override `metadata()` and have it wrap a call to `super.metadata( dottedPath )` in `try/catch` and return an empty struct if an exception is thrown.
* `private void function logMissingBean( string beanName, string resolvingBeanName = "" )` - this is called from `missingBean()` to log DI/1's inability to find a dependency: the default implementation writes a message to the application server's console log.
* `private any function missingBean( string beanName, string resolvingBeanName = "", boolean dependency = true )` - this is called when DI/1 cannot find a dependency and, new in FW/1 4.0 / DI/1 1.2, also when `getBean()` cannot find the specified bean. The default implementation is explained below.

### Overriding missingBean()

`missingBean()` is called when DI/1 cannot find a bean. The default behavior when called from `getBean()` is to throw a "bean not found" exception. The default behavior when called during dependency resolution is to either throw a "bean not found" exception (when running in `strict` mode) or just call `logMissingBean()`. Prior to FW/1 4.0 (DI/1 1.2), `getBean()` threw the exception directly, and during dependency resolution the result of calling `missingBean()` was ignored.

If you override `missingBean()` you could delegate bean lookup / creation to your own convention-based bean factory and return your own bean. You could decide whether to invoke that just for failed calls to `getBean()` (when `dependency` is `false`) or also for bean lookup during dependency resolution (when `dependency` is `true`). The `resolvingBeanName` argument is provided to allow for better error messages during failures to resolve dependencies and is not expected to affect the behavior of any override. If your `missingBean()` does not throw an exception, whatever result it returns will be used in place of the bean that DI/1 could not find. If you return nothing (`return;`), then DI/1 will not attempt to use the result: for dependency resolution that means the dependency will simply by ignored (and nothing injected); for `getBean()` calls that means that `getBean()` itself will return nothing (instead of throwing an exception - its default behavior) so be careful there!
