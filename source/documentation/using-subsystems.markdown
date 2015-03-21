---
layout: page
title: "Using Subsystems in FW/1"
date: 2015-03-21 14:20
comments: false
sharing: false
footer: true
---
_This is the latest (3.1 - develop) documentation - for the current stable release, read the [3.0 master documentation](/documentation/3.0/using-subsystems.html)._

Subsystems give you a way of dropping one or more small FW/1 applications into an existing one. Subsystems can be used to create modules that have no dependencies on the parent application or you can use subsystems to group common functionality together.

Enabling Subsystems
---
If you set either `defaultSubsystem`, `siteWideLayoutSubsystem`, `subsystemDelimiter`, or `subsystems` in the framework configuration, then subsystems will be automatically enabled.

Otherwise, if you omit all of those to pick up their default values (of `"home"`, `"common"`, `":"`, and `{ }` respectively), you will need to explicitly set `usingSubsystems` to `true` in the framework configuration.

Once you enable subsystems, you'll need to follow a couple of additional conventions:

1. Your site's default application must also be implemented as a subsystem. By default, the framework will look in the sub-directory `home`. Setting `defaultSubsystem` overrides this default, letting you specify another folder as the default application module.
1. A sitewide layout can be specified at `common/layouts/default.cfm` (by default). If this file exists, it will be applied. Setting `siteWideLayoutSubsystem` overrides this default, letting you specify another folder to look in for `layouts/default.cfm`.

Accessing Subsystems
---
To access a subsystem in the browser, you'll have to specify it in the action:

    index.cfm?action=subsystem:section.item

If you leave off the subsystem in the url, the section and item will reference the default subsystem (by default `home:section.item`).

When creating links in your views and layouts, it's recommended that you use `buildUrl()`. You do not have to specify the current subsystem inside `buildUrl()`, it will automatically be prepended. This method is preferred. If you change the name of a subsystem, all of your links inside the subsystem will correctly reflect the change.

    buildUrl('section.item') - action=currentSubsystem:section.item

You can also link to other subsystems:

    buildUrl('otherSubsystem:section.item') - action=otherSubsystem:section.item

Configuring Subsystems
---
There is an optional method that can be declared in `Application.cfc` for configuring subsystems:

    function setupSubsystem(subsystem) {}

`setupSubsystem()` is called once for each subsystem, when a subsystem is initialized. When an application is reloaded, the initialized subsystems are cleared and `setupSubsystem()` will be called on the next request to a subsystem.

### Framework Configuration

The following options relate to subsystems:

* `usingSubsystems` - This will be `true` if you are using subsystems. You only need to set this explicitly if you don't specify any of the other subsystem configuration options below.
* `defaultSubsystem` - This is the default subsystem when none is specified in the URL or form post.
* `subsystemDelimiter` - This specifies the delimiter between the subsystem name and the action in a URL or form post. It cannot be `"."` since section and item would no longer be parsed correctly. It must be a legal URL character.
* `siteWideLayoutSubsystem` - This specifies the subsystem that is used for the (optional) site-wide default layout.

Controllers
---
Subsystem controllers are located in `subsystem/controllers` (e.g., `admin/controllers/login.cfc`, `home/controllers/main.cfc`).

Model (Services and Domain Objects)
---
Subsystem services are located in `subsystem/model/services` and domain objects in `subsystem/model/beans` (e.g., `admin/model/services/security.cfc` and `home/model/beans/user.cfc`).

Views
---
Subsystem views are located in `subsystem/views` (e.g., `admin/views/login/default.cfm` and `home/views/main/error.cfm`). 

Layouts
---
Subsystem layouts are looked up in the same order as before, but with the additional inclusion of a sitewide layout, if it exists. The default sitewide layout folder is `common`. Subsystem specific layouts are located in `subsystem/layouts` (e.g., `admin/layouts/login.cfm`).

* `subsystem/layouts/section/item.cfm`
* `subsystem/layouts/section.cfm`
* `subsystem/layouts/default.cfm`
* `common/layouts/default.cfm`

The location of the latter is determined by the `siteWideLayoutSubsystem` configuration variable.

Using Bean Factories
---
The introduction of subsystems introduces the ability to have subsystem specific bean factories. If you let FW/1 use DI/1 (or AOP/1) to manage your beans, it will also do so automatically for subsystems, creating a bean factory for each subsystem (inspecting the same folders you configured for your main application, i.e., `model` and `controllers` by default), and setting the main bean factory as the parent of each subsystem bean factory. This is the recommended approach (naturally!).

The following bean factory methods are available for subsystems:

* `setSubsystemBeanFactory( subsystem, beanFactory )` - sets up a subsystem specific bean factory 
* `hasSubsystemBeanFactory( subsystem )` - returns `true` if a subsystem specific bean factory exists for the named subsystem
* `getBeanFactory()` - returns the bean factory for the current subsystem. Alternately, it can be used to retrieve the bean factory for another subsystem by passing the name of the subsystem (e.g., `getBeanFactory( subsystem )` ).
* `getDefaultBeanFactory()` - returns the default bean factory that was passed to `setBeanFactory()` (at the top-level)

### Auto Wiring

If you did not declare a subsystem specific bean factory, the framework will attempt to auto wire beans from the default bean factory into subsystem specific controllers and model components.

If you have declared a subsystem specific bean factory, the framework will attempt to auto wire only the beans in the subsystem specific bean factory into your subsystem controllers and model components. If a subsystem specific bean factory does not contain those beans, beans from the default bean factory will only be autowired into your subsystem controllers and model components if the default bean factory has been set as the parent of the subsystem specific bean factory.

If you declare a dependency of `property beanFactory;`, the bean factory that is autowired will be the bean factory in which that managed bean exists.

### Setting Bean Factories With setupSubsystem()

With `setupSubsystem()`, it's possible to use your own convention to load subsystem specific bean factories, instead of either relying on FW/1's conventions or explicitly declaring each one in `setupApplication()` for the main application. The following example uses ColdSpring and makes the assumption that each subsystem has a bean factory config file in a common subsystem specific folder. If the config file is found, it then loads the subsystem bean factory.

    function setupSubsystem( subsystem ) {
        var bfConfigFilePath = subsystem & '/config/coldspring.xml.cfm';
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

While it's not considered a best practice, there may be a chance when you will need to access a bean factory from another subsystem. You can do this by calling `getBeanFactory(subsystem)` (e.g., `getBeanFactory('user')`).
