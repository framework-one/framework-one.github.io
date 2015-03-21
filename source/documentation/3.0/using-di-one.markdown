---
layout: page
title: "Using DI/1"
date: 2015-02-22 14:20
comments: false
sharing: false
footer: true
---
DI/1 - a.k.a Inject One - is a simple, convention-based Dependency Injection framework. 

DI/1 searches specified directories for CFCs and treats them as singletons or non-singletons (transients) based on naming conventions for the CFCs themselves, or the folders in which they are found. You can override the conventions by configuration if needed.

# Getting Started with DI/1

Create an instance of the DI/1 bean factory and specify the folder(s) you want it to search for beans (CFCs):

    var beanFactory = new ioc("/model");

CFCs found in a folder called *beans* are assumed to be transients; otherwise CFCs are assumed to be singletons. If CFC names are unique, you can use that name to get the bean out of the factory:

    var userManager = beanFactory.getBean("userManager");

All beans are also given an alias which is the name of the CFC followed by (the singular form of) the folder name in which it was found, e.g., /model/beans/product.cfc would get the alias "productBean". If no other CFC is called product.cfc in the folders that you asked DI/1 to search, you can use "product" or "productBean" to reference that bean. By default, DI/1 assumes all beans are singletons unless they are found in a folder called *beans* (in which case DI/1 assumes those are transients). A singleton has just a single instance and DI/1 will cache that instance. A transient is created afresh every time you ask DI/1 for an instance.

If a CFC has a constructor (a method called **init()**), DI/1 will use the argument names to look up beans and call the constructor with those beans. If a CFC has setter methods, DI/1 will use their names to look up beans and call the setters with those beans. If a CFC has property declarations and implicit setters are enabled, DI/1 will use their names to look up beans and call the implicit setters with those beans. This is called autowiring. By the time you get a bean back from DI/1, it should be fully populated. You can also specify an "init-method" function name that DI/1 should call after a bean has had its dependencies injected - see **Configuration** below.

If DI/1 cannot find a matching bean for a constructor argument, it will throw an exception. If DI/1 cannot find a matching bean for a setter method or property, it will log the failure and ignore it (by default), and the corresponding variable will not be populated. You can configure DI/1 to be strict about matching bean names - see the configuration section below - in which case it will throw an exception.

Note that DI/1 will only inject singletons via setters or properties. Injecting transients in those situations often leads to unexpected results (consider a transient **invoice** bean that has a **setCustomer()** method when you also have a transient **customer** bean - you almost certainly don't want DI/1 to automatically create a customer instance and inject it every time you ask DI/1 for a new invoice bean!). If a constructor argument matches a transient bean, DI/1 will still create an instance since it has to finish constructing the original bean.

## Acceptable Folder Paths

In general, you should use webroot-relative folders - starting with **/** - or mappings - also starting with **/** - as the constructor arguments to **ioc**. If you pass a full file system path, DI/1 will only be able to deduce the dotted-name of CFCs found there if it points into the webroot tree. Similarly, if you pass a relative folder path, it must point into the webroot tree. If DI/1 cannot deduce the dotted name of a CFC, it will throw an exception.

# More Advanced Usage

This section covers the rest of the public API, how to specify additional folders as containing transients, parent bean factories and bean factory injection.

## Other Public Methods

Given a struct of values (such as form scope or URL scope), you can ask DI/1 to inject those values as properties into a given bean:
```
beanFactory.injectProperties(myBeanInstance, form);
beanFactory.injectProperties("user", userAttributes);
```
The first call will loop over the form scope and, for each key in that scope, call a setter on **myBeanInstance**. The second call asks DI/1 to create a **user** bean and populate it by calling a setter for each element of the struct **userAttributes**. You may also use a dotted-path to a CFC as the first argument in which case DI/1 will use **createObject** to instantiate it and *will not call the constructor*. Caution: DI/1 assumes you know what you're doing and will call a setter for *every* member of the struct passed in!

You can programmatically add new bean instances - or named values:
```
beanFactory.addBean("magicvalue", 42);
beanFactory.addBean("logger", new LogFactory("log4j"));
```
After these calls, **getBean("magicvalue")** will return the value 42 and **getBean("logger")** will return the CFC instance you provided. That means that any properties, setter methods or constructor arguments that refer to **magicvalue** or **logger** will get those values injected.

You can also programmatically declare new beans to be managed by DI/1:

    beanFactory.declareBean("navigation", "site.utils.navigation", true);

That will tell DI/1 that **/site/utils/navigation.cfc** should be managed as a singleton with name **navigation**. You can declare transients by specifying **false** as the third argument. **true** is the default so it can be omitted for singletons.

When declaring a bean, you can also optionally provide a set of overrides for named beans, so that constructor arguments or properties will take on specified values, rather than what is managed by the bean factory. This is useful for creating variants of a single bean:

    beanFactory.declareBean("datasource", "util.DataSource", true, { dsn = "main" } );
    beanFactory.declareBean("admindata", "util.DataSource", true, { dsn = "admindb" } );

You can declare a factory bean - like Spring/ColdSpring - as follow:

    beanFactory.factoryBean("generated", factory, "method", [ ..args.. ], { ... } );

This tells DI/1 that when you call `getBean("generated")`, instead of trying to create the bean itself, it should call `factory.method(..args..)` to get the bean instance. `args` can be omitted (and defaults to an empty list of arguments). The last argument provides overrides for bean values, as shown above, and is optional.

You can add an alias for a bean:

    beanFactory.addAlias("alsoKnownAs", "navigation");

That will tell DI/1 that **alsoKnownAs** is an alias for the bean identified by **navigation** so **getBean("alsoKnownAs")** will behave the same as **getBean("navigation")**.

If you want code to be executed after DI/1 has discovered all the beans on disk -- for example, to configure a variety of additional "constant" or computed beans -- you can use the **onLoad()** method to specify a listener function:

    beanFactory.onLoad( loadListener );

That will register **loadListener** with DI/1 to be called after bean discovery is complete. This is a good place to put your calls to `declareBean()` and `addAlias()` if you need those to be in effect prior to the first call to `getBean()`.

* If **loadListener** is a CFC instance, DI/1 will call **loadListener.onLoad( beanFactory )**, passing the DI/1 instance in as an argument.
* If **loadListener** is a bean name, DI/1 will call **beanFactory.getBean( loadListener ).onLoad( beanFactory )**, where **beanFactory** is the DI/1 instance.
* If **loadListener** is a function or closure, DI/1 will call **loadListener( beanFactory )**. Note that if **loadListener** is a method on a CFC, it will be called out of context so it will not have access to the **variables** scope or **this** scope of that CFC instance and therefore also won't have access to other methods of that CFC.

You can ask if the bean factory knows about a particular bean using the **containsBean()** method:

    if ( beanFactory.containsBean("productService") ) ...

(although you probably shouldn't need to do this unless you are building some sort of framework plugin that needs to check what is available to it at runtime!).

You can force all singletons to be reloaded using the **load()** method:

    beanFactory.load();

That will empty the bean cache and then call **getBean()** on every bean that DI/1 knows about. Note: it does not call **load()** on any parent bean factory (see below) and it does not perform a new search on the folders (so it won't see newly written CFCs). To force the search to be performed again, create a new instance of the bean factory as shown above.

Metadata can be queried using the following methods:
```
if ( beanFactory.isSingleton("someBean") ) ...
info = beanFactory.getBeanInfo("someBean");
```
I would expect these only to be useful to framework authors. Both methods walk up into parent bean factories, if present. If you omit the bean name for **getBeanInfo()** you get back a struct with a key **beanInfo** that refers to metadata for all of the beans known in the factory. If there is a parent bean factory, its metadata is returned under a key **parent** in that struct.

`getBeanInfo()` can be called with a `beanName` argument - the default - or with a `regex` argument which will return metadata about all the beans in the factory whose names match the regular expression, in a struct with the single key `beanInfo`, whose value will be a struct with a key for each matching bean.

`getBeanInfo()` can also be called with no arguments, in which case it will return metadata for all the beans in the factory (in the `beanInfo` key of the result) and metadata for all the beans in the factory's parent, if any, in the `parent` key of the result. Optionally, you may specify an argument of `flatten = true` and the `parent` structures will be merged (recursively through the parents) into `beanInfo`, producing a flat struct.

## Specifying Additional Transient Beans

By default, any CFC in the **beans** folder is considered a transient and everything else is considered a singleton. There are three ways to specify other CFCs should be considered transient:

* **config.singulars** allows you to specify additional folders that resolve to **bean**
* **config.transients** allows you to specify additional folders whose contents are transient
* **config.singletonPattern** allows you to specify a regular expression which limits which beans are considered singletons
* **config.transientPattern** allows you to specify a regular expression which limits which beans are considered transients

In the first case, any folder name whose singular name is **bean** will cause CFCs to get an alias that ends in **Bean** and will be considered transients. In the second case, the singular transformation will still be applied to create the alias, but the CFCs will be considered transients anyway. See below for additional uses of **config.singulars**. In the third case, CFCs will also be considered transients if their name does not match the regular expression pattern supplied. In the fourth case, CFCs will also be considered transients if their name matches the regular expression pattern supplied. You cannot specify both **config.singletonPattern** and **config.transientPattern**.

For example:

    var beanFactory = new ioc( ".", { singulars = { objects = "bean" }, transients = [ "models" ] } );

This will cause CFCs found in the **objects** folder to be treated as if they were in the **beans** folder (their alias will end with **Bean** and they will be considered transients because of that) and CFCs found in the **models** folder to be treated as transients too (but their alias will end with **Model**, the singular of **models**).

    var beanFactory = new ioc( ".", { singulars = { services = "manager" }, transients = [ "objects" ] } );

This, on the other hand, will cause CFCs found in the **services** folder to be treated as if they were in the **managers** folder (their alias will end with **Manager** and they will be considered singletons because of that) and CFCs found in the **objects** folder to be treated as transients (their alias will end with **Object**, the singular of **objects**).

    var beanFactory = new ioc( ".", { singletonPattern = "(Service|Factory)$" } );

In addition to any CFCs found in a folder called **beans**, any CFC whose name does not end in **Service** or **Factory** will be considered a transient.

    var beanFactory = new ioc( ".", { transientPattern = "(Entity)$" } );

In addition to any CFCs found in a folder called **beans**, any CFC whose name ends in **Entity** will be considered a transient.

## Parent Bean Factories

If your application is assembled from multiple modules, you may have a main bean factory containing shared CFCs and each module may also have a bean factory. You can tell a module's bean factory about the shared CFCs in the main bean factory using the **setParent()** method:
```
var moduleBeanFactory = new ioc("/moduleModel");
moduleBeanFactory.setParent( mainBeanFactory );
```
This causes DI/1 to ask its parent bean factory about any beans that are requested but unknown (within the moduleBeanFactory). Because DI/1 uses only **containsBean(name)** and **getBean(name)** the parent bean factory does not need to be another DI/1 instance - it can be any bean factory that provides that API.

## Bean Factory Aware

If you need access to the bean factory itself within one of your CFCs, either declare a constructor argument called **beanFactory**, provide a **setBeanFactory( any beanFactory )** setter or declare a **beanFactory** property (with implicit setters enabled). DI/1 declares itself as a bean called **beanFactory** and will inject itself where any such dependencies appear.

# Configuration

When you create the bean factory, you can optionally supply a second argument that is a struct containing configuration for DI/1. At present, DI/1 understands the follow config options:

* **constants** - struct - defaults to **{}**. DI/1 will use any name/value pairs specified here to provide _beans_ that resolve to the specified values. This can be used to provide resolution for constructor arguments that need values which are not actual beans.
* **exclude** - array - defaults to **[]**. DI/1 will ignore any CFCs whose file path contains the strings in this array. DI/1 always excludes paths containing **/WEB-INF** and **/Application.cfc**, as well as various FW/1 and DI/1 framework files. The strings are not case-sensitive.
* **initMethod** - string - If specified, identifies a method name on beans that DI/1 will attempt to call (with no arguments) on each bean after its dependencies have been injected.
* **omitDirectoryAliases** - boolean - defaults to **false**. If **true**, use CFC names as bean names directly, without appending the singular directory name as a suffix. If your CFC names are not unique, you will get an exception.
* **omitTypedProperties** - boolean - defaults to **false**. If **true**, property declarations that specify a type will be ignored for injection. That is useful if you are working with the ORM (since those property declarations will have types and should not be treated as dependencies).
* **recurse** - boolean - defaults to **true**. Controls whether DI/1 searches subfolders recursively or not.
* **singletonPattern** - string - no default. Specifies a regular expression that DI/1 uses to determine whether a bean is singleton or not, based on its name. The **beans** folder convention and the **transients** configuration below still apply so nothing in those folders will be considered a singleton, even if its name matches the pattern.
* **singulars** - struct - defaults to **{}**. DI/1 will use any name/value pairs specified here to translate folder names to a singular variety, e.g., **pride = 'lion'** will convert the *plural* folder **pride** to the *singular* name **lion** and therefore a **simba.cfc** within the **pride** folder will get the alias **simbaLion**. This also allows for other folders to behave as if they were called **beans** by treating their singular name as **bean**. One of the DI/1 unit tests maps **sheep** to **bean** for this reason. This won't work if the CFCs in **sheep** have the same name as the CFCs in **beans** however.
* **strict** - boolean - defaults to **false**. If **true**, DI/1 will throw an exception if it cannot resolve a bean implied by a constructor argument, setter name or property name. If **false**, DI/1 simply calls **logMissingBean()** which writes the failure to the Java console.
* **transients** - array - defaults to **[]**. DI/1 will consider any CFCs found in these folders to be transient, rather than singleton. The conversion to a singular form will still take place to create the alias for each CFC. For example, if **singulars** = { pride = 'lion' } and **transients** = [ 'pride' ] then any CFCs in the **pride** folder will be treated as transients and their alias will end in **Lion**.
* **transientPattern** - string - no default. Specifies a regular expression that DI/1 uses to determine whether a bean is transient or not, based on its name. The **beans** folder convention and the **transients** configuration below still apply so CFCs in those folders will be still considered transients, in addition to any name that matches the pattern.

## Configuring "Constant" Beans

As noted above, the optional config argument to the **ioc** constructor is a struct containing various parameters that alter the behavior of DI/1. The **constants** config element is a struct containing mappings from bean names to specific constant values. This allows you to specify non-CFC values for constructor arguments, setters and properties (but is most commonly used for constructor arguments). The value may be of any type and any reference to that bean name will return the specified value as a singleton.

These values may be added after DI/1 has been initialized using the **addBean()** method as shown above.

## Overriding DI/1 Behavior

If you want to override the methods in DI/1, such as **logMissingBean()**, you can create your own CFC that extends **ioc.cfc** and overrides the desired methods. Then use your CFC instead of **ioc.cfc**. If any particular use case becomes common, we can discuss incorporating it into DI/1 as a configuration option.

A particular extension point that is provided is:

    private void function setupInitMethod( string name, any bean )

This is called for each bean after its dependencies have been injected prior to calling **initMethod** (if specified).

Two related extension points that can be useful as well are:

    private any function construct( string dottedPath )
    
    private any function metadata( string dottedPath )

These can be overridden if you want to change the behavior of how beans are created and how metadata is obtained for beans. An example from Adam Tuttle is the ability to silently ignore beans that have syntax errors during development, so the rest of the beans are loaded: you would override `metadata()` and have it wrap a call to `super.metadata( dottedPath )` in `try/catch` and return an empty struct if an exception is thrown.