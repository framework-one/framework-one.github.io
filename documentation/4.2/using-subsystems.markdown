---
layout: documentation
title: "Using Subsystems in FW/1"
date: 2017-07-01 19:40
comments: false
sharing: false
footer: true
---
Subsystems give you a way of modularizing your FW/1 application as it grows. They also provide a way to incorporate other FW/1 applications directly into an existing one. Subsystems can be used to create modules that have no dependencies on the parent application or you can use subsystems to group common functionality together.

* TOC
{:toc}

Subsystems 1.0 vs 2.0
---
Subsystems have existed in FW/1 for many years as an optional feature you needed to enable (via the `framework.usingSubsystems` configuration setting). The conventions used for those subsystem-based applications were:

* Each subsystem was in a top-level folder of the application.
* Your "main" application was just a subsystem, like any other (by default it was the `"home"` subsystem).
* In addition to the normal three-tier cascading layout for each subsystem, you could specify a subsystem that contained a fourth tier layout to wrap every page (via `siteWideLayoutSubsystem`).

This style of subsystems is referred to as "Subsystems 1.0" or "legacy style subsystems" throughout this documentation.

Based on a proposal by Steven Neiland, FW/1 3.5 offers a new way to deal with subsystems that can be used instead of the above approach. The conventions used for this new style of subsystem-based applications are:

* Subsystems live in a `subsystems` folder.
* Your "main" application is just a regular FW/1 application -- the top-level application.
* In addition to the normal three-tier cascading layout for each subsystem, your main application's site-wide layout is also applied (i.e., `layouts/default.cfm`).

This new style of subsystems is referred to as "Subsystems 2.0", but is treated as the normal way to add modules to your growing application throughout this documentation. You do not need to enable anything for this style of subsystems.

Enabling Subsystems
---
To take advantage of (the new style of) subsystems, you do not need to do anything: requests for actions like _module:section.item_ will cause FW/1 to automatically look for a subsystem called _module_ in the `subsystems` folder of your application. In particular, you _must not_ set `defaultSubsystem` or `siteWideLayoutSubsystem` in your FW/1 configuration and you should either omit `usingSubsystems` or set it explicitly to `false`.

If you set either `defaultSubsystem` or `siteWideLayoutSubsystem` in the framework configuration, then legacy subsystems will be automatically enabled. If you want to use legacy subsystems without changing either of those from their defaults, you can set `usingSubsystems` to `true`.

_Note: in earlier versions of FW/1, if you set either `subsystemDelimiter` or `subsystems` in the framework configuration, that also automatically enabled (legacy) subsystems. Since those are both applicable to the new style of subsystems, they no longer cause `usingSubsystems` to be set to `true`. **This is a breaking change for legacy subsystem-based applications that did not explicitly set `usingSubsystems` to `true` but instead relied on setting either of those configuration items to enable subsystems!**_

If you are allowing DI/1 to manage your model, any services or beans defined under the top-level `model` folder will be available to all subsystems as well, as FW/1 uses a parent/child relationship between the bean factory that manages the top-level `model` folder and each bean factory that manages a subsystem-specific `model` folder.

Your main application will not have access to any services or beans defined in subsystem-specific `model` folders.

If you explicitly enable legacy subsystems, you'll need to follow a couple of additional conventions:

1. Your subsystems will live loose in the top-level folder, alongside `controllers`, `layouts`, and `views` instead of neatly inside a `subsystems` folder.
1. Your site's main application must also be implemented as a subsystem. By default, the framework will look in the sub-directory `home`. Setting `defaultSubsystem` overrides this default, letting you specify another folder as the default application module.
1. A sitewide layout can be specified at `common/layouts/default.cfm` (by default). If this file exists, it will be applied. Setting `siteWideLayoutSubsystem` overrides this default, letting you specify another folder to look in for `layouts/default.cfm`.

Accessing Subsystems
---
To access a subsystem in the browser, you'll have to specify it in the action:

    index.cfm?action=module:section.item

If you leave off the subsystem in the url, the section and item will reference the main application (in legacy subsystems, this would be the `home` subsystem).

When creating links in your views and layouts, it's recommended that you use `buildURL()`. You do not have to specify the current subsystem inside `buildURL()`: it will automatically be prepended. This method is preferred. If you change the name of a subsystem, all of your links inside the subsystem will correctly reflect the change.

    // inside any subsystem
    buildURL('section.item') - action=currentSubsystem:section.item
    // inside your main application (2.0):
    buildURL('section.item') - action=section.item
    // inside your main application (1.0):
    buildURL('section.item') - action=homeSubsystem:section.item

You can also link to other subsystems:

    buildURL('otherSubsystem:section.item') - action=otherSubsystem:section.item
    // link to your main application (2.0):
    buildURL(':section.item') - action=section.item

Configuring Subsystems
---
There is an optional method that can be declared in `Application.cfc` for configuring subsystems:

    function setupSubsystem(subsystem) {}

`setupSubsystem()` is called once for each subsystem, when a subsystem is initialized. When an application is reloaded, the initialized subsystems are cleared and `setupSubsystem()` will be called on the next request to a subsystem.

### Framework Configuration

The following options relate to subsystems:

* `subsystemDelimiter` - This specifies the delimiter between the subsystem name and the action in a URL or form post. It cannot be `"."` since section and item would no longer be parsed correctly. It must be a legal URL character.
* `subsystems` - This provides subsystem-specific configuration.

In addition, legacy subsystems used these options as well (these are not applicable to new style subsystems):

* `usingSubsystems` - This will be `true` if you are using legacy subsystems. You only need to set this explicitly if you don't specify either of the subsystem configuration options below.
* `defaultSubsystem` - This is the default subsystem when none is specified in the URL or form post.
* `siteWideLayoutSubsystem` - This specifies the subsystem that is used for the (optional) site-wide default layout.

Controllers
---
Subsystem controllers are located in `subsystems/module/controllers` (e.g., `subsystems/admin/controllers/login.cfc`, `subsystems/home/controllers/main.cfc`).

Legacy subsystems do not have the `subsystems/` prefix.

Model (Services and Domain Objects)
---
Subsystem services are located in `subsystems/module/model/services` and domain objects in `subsystems/module/model/beans` (e.g., `subsystems/admin/model/services/security.cfc` and `subsystems/home/model/beans/user.cfc`).

Legacy subsystems do not have the `subsystems/` prefix.

Views
---
Subsystem views are located in `subsystems/module/views` (e.g., `subsystems/admin/views/login/default.cfm` and `subsystems/home/views/main/error.cfm`).

Legacy subsystems do not have the `subsystems/` prefix.

Layouts
---
Subsystem layouts are looked up in the same order as before, but with the additional inclusion of a sitewide layout, if it exists. The default sitewide layout is `layouts/default.cfm` (for legacy subsystems, it is `common/layouts/default.cfm`). Subsystem specific layouts are located in `subsystems/module/layouts` (e.g., `subsystems/admin/layouts/login.cfm`).

* `subsystems/module/layouts/section/item.cfm`
* `subsystems/module/layouts/section.cfm`
* `subsystems/module/layouts/default.cfm`
* `layouts/default.cfm`

Legacy subsystems do not have the `subsystems/` prefix and the sitewide layout is `common/layouts/default.cfm` by default. That location can be overridden by the (legacy) `siteWideLayoutSubsystem` configuration variable.

Using Bean Factories
---
In addition to your top-level bean factory, you can have subsystem-specific bean factories as well. If you let FW/1 use DI/1 to manage your beans, it will also do so automatically for subsystems, creating a bean factory for each subsystem (inspecting the same relative folder paths you configured for your main application, i.e., `model` and `controllers` by default), and setting the main bean factory as the parent of each subsystem bean factory. This is the recommended approach (naturally!). Note that absolute and root-relative paths are ignored (i.e., those that begin with `/`) since that could cause conflicts with the parent bean factory _(that restriction is new in 3.5)_.

The following bean factory methods are available for subsystems:

* `setSubsystemBeanFactory( subsystem, beanFactory )` - sets up a subsystem specific bean factory
* `hasSubsystemBeanFactory( subsystem )` - returns `true` if a subsystem specific bean factory exists for the named subsystem
* `getBeanFactory()` - returns the bean factory for the current subsystem. Alternately, it can be used to retrieve the bean factory for another subsystem by passing the name of the subsystem (e.g., `getBeanFactory( subsystem )` ).
* `getDefaultBeanFactory()` - returns the default bean factory that was passed to `setBeanFactory()` (at the top-level)

If you provide `diConfig` in the subsystem-specific configuration structure (`framework.subsystems[module]`) that will be passed to DI/1 as its configuration, otherwise the same configuration used for the default bean factory (`framework.diConfig`) will be passed. _New in 3.1._

### Auto Wiring

If you do not have a subsystem-specific bean factory, the framework will attempt to auto wire beans from the default bean factory into subsystem specific controllers and model components.

If you do have a subsystem-specific bean factory, the framework will attempt to auto wire only the beans in the subsystem-specific bean factory into your subsystem controllers and model components. If a subsystem-specific bean factory does not contain those beans, beans from the default bean factory will only be autowired into your subsystem controllers and model components if the default bean factory has been set as the parent of the subsystem-specific bean factory.

If you declare a dependency of `property beanFactory;`, the bean factory that is autowired will be the bean factory in which that managed bean exists.

### Setting Bean Factories With setupSubsystem()

With `setupSubsystem()`, it's possible to use your own convention to load subsystem specific bean factories, instead of either relying on FW/1's conventions or explicitly declaring each one in `setupApplication()` for the main application. The following example uses ColdSpring and makes the assumption that each subsystem has a bean factory config file in a common subsystem specific folder. If the config file is found, it then loads the subsystem bean factory.

    function setupSubsystem( subsystem ) {
        var bfConfigFilePath = '/subsystems/' & subsystem & '/config/coldspring.xml.cfm';
        // conditionally load bean factory for this subsystem by convention:
        try {
            if ( fileExists(expandPath('./') & bfConfigFilePath) ) {
                var bf = new coldspring.beans.DefaultXmlBeanFactory();
                bf.loadBeans( bfConfigFilePath );
                bf.setParent( getDefaultBeanFactory() );
                setSubsystemBeanFactory( subsystem, bf );
            }
        } catch ( any e ) {
            // ignore exceptions caused by bad paths etc
        }
    }

### Accessing Other Bean Factories From A Subsystem

If you have a default bean factory, you can access it in your controllers and views from any subsystem with `getDefaultBeanFactory()`.

While it's not considered a best practice, there may be a situation where you need to access a bean factory from another subsystem. You can do this by calling `getBeanFactory( subsystem )` (e.g., `getBeanFactory( 'user' )`).
