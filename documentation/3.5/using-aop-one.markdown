---
layout: page
title: "Using AOP/1"
date: 2015-06-19 12:00
comments: false
sharing: false
footer: true
---
AOP/1 is a simple Aspect Oriented Programming extension for [DI/1 (a.k.a Inject One)](/documentation/3.5/using-di-one.html) which allows you to define interceptors for your beans.

These interceptors can run code before or after a method is called on a bean without the need for you to alter the code in your bean.  This allows you to create generic services (such as a logger service) that is coded and configured to operate completely separate from your other services and beans.  What this means is you no longer need to mix unrelated service code together by using dedicated interceptors.

_The information below assumes that you already have a good working knowledge of DI/1._

# Getting Started with AOP/1

Create an instance of the AOP/1 extended DI/1 bean factory and specify the folder(s) you want it to search for beans.
```
var beanFactory = new framework.aop("/model");
```
So far nothing difficult since this is what we would typically see from [DI/1](/documentation/3.5/using-di-one.html).  Now, if we want to intercept method calls to an object, we need to declare the interceptors and the objects that should be intercepted.
```
var beanFactory = new framework.aop("/model");

beanFactory.intercept("pdfService", "beforeInterceptor");
beanFactory.intercept("pdfService", "afterInterceptor", "createDocument");

var ps = beanFactory.getBean("pdfService");

var document = ps.createDocument("http://seancorfield.github.io");
var pages = ps.splitPages(document);
```
In this example the _beforeInterceptor_ will intercept every call to the _pdfService_, but the _afterInterceptor_ will only intercept calls to the _createDocument_ method. Due to AOP/1 creating intercept points on the bean being intercepted, it is generally recommended to list the methods to be intercepted when declaring the interceptors so there are not unnecessary calls made on other methods.

## Creating Interceptors

A common practice for DI/1 is to place beans and services within a model folder like so:

* /model/beans/
* /model/services/

Interceptors can follow this pattern in order to make it simple for the factory to locate the interceptors with the rest of the model.

* /model/interceptors/

### Before Interceptors

Before interceptors will intercept method calls **before** they are executed.  They cannot affect the result of a method call, but they can be used to alter the arguments going to the method call and they can perform operation that you wish to be performed before the method call.  In order for an interceptor to operate as a before interceptor, it only needs the _before_ method to be defined.
```
component {
    function before(targetBean, methodName, args) {
        arguments.args.input = "before" & arguments.args.input;
    }
}
```
Because the interceptor is like any other bean handled by DI/1, dependencies can be intjected into the interceptor and used by the interceptor.
```
component {
	property logService;

    function before(targetBean, methodName, args) {
        getLogService().logMethodCall(arguments.methodName, arguments.args);
    }
}
```

### After Interceptors

Just like the name implies, after interceptors will intercept method calls **after** they are executed.  They cannot affect the arguments going to the method call, but they can monitor or alter the result of the method call.  In order for an interceptor to operate as an after interceptor, it only needs the _after_ method to be defined.
```
component {
    property logService;

    function after(targetBean, methodName, args, result) {
        if (structKeyExists(arguments, "result) && !isNull(arguments.result) {
            getLogService().logMethodCallResult(arguments.methodName, arguments.args, arguments.result);
        }
    }
}
```
Should you wish to alter the result being returned, all that is needed is to return something from the _after_ method.
```
component {
    function after(targetBean, methodName, args, result) {
        if (structKeyExists(arguments, "result) && !isNull(arguments.result) {
            return arguments.result & "After";
        }
    }
}
```

### onError Interceptors

onError interceptors allow errors that occur during the execution of intercepted method calls to be handled outside the normal flow of model execution.  This can be used for situations where the normal error handling of your application will not produce the desired result.
```
component {
    property logService;

    function onError(targetBean, methodName, args, exception) {
        getLogService().logException(arguments.methodName, arguments.args, arguments.exception);
    }
}
```

### Around Interceptors

Around interceptors are an interesting interceptor, because unlike other interceptor types, an around interceptor can actually stop execution of an intercepted method.  This accomplished because, unlike before, after, and onError interceptors which are called externally in the order they are defined in their stacks, the around interceptors always call the next interceptor in their stack.

Calling the next interceptor in the stack for around interceptors is accomplished by calling the _proceed()_ method.  The _proceed_ method is automatically added to any interceptor that has an _around_ method.  An around interceptor can stop the execution chain by simply not calling the _proceed_ method.  The around interceptors can preform the actions of both before and after interceptors as well.
```
component {
    property logService;
    property userService;

    function around(targetBean, methodName, args) {
        // Perform 'before' arguments manipulation.
        arguments.args.name = getUserService().getCurrentUser().getName();

        if (getUserService().getCurrentUser().hasPermission("administrator")) {
            var result = proceed(arguments.targetBean, arguments.methodName, arguments.args);

            if (!isNull(result))
            {
                getLogService().logMethodCallResult(arguments.methodName, arguments.args, result);

				    return result;
				}
        }
    }
}
```
Since an around interceptor may intercept multiple methods, the method must be able to handle any type of result being returned (including void/null).  The example above demonstrates handling when a result is present and demonstrates how the execution chain can be stopped by not calling the _proceed_ method if the current user is not an 'administrator'.

# Advanced Usage & Understanding

The following section explains additional features and concepts that may prove useful when implementing AOP/1.

## Loading Interceptors Via Configuration

AOP/1 extends DI/1 so it has access to the _config_ parameter of the constructor.
```
var interceptors = [{beanName = "stringUtilityService", interceptorName = "afterInterceptor"}];
var factory = new framework.aop(folders, {interceptors = interceptors});
```
The _interceptors_ configuration is just an array of structures that define the interceptors to be loaded like so:
```
var interceptors =
[
    {beanName = "stringUtilityService", interceptorName = "beforeInterceptor", methods = "forward,reverse,split"},
    {beanName = "stringUtilityService", interceptorName = "afterInterceptor"},
    {beanName = "stringUtilityService", interceptorName = "afterInterceptor2", methods = ""},
    {beanName = "stringUtilityService", interceptorName = "afterInterceptor3", methods = "*"},
    {beanName = "stringUtilityService", interceptorName = "aroundInterceptor", methods = "reverse"}
];
```
When the _methods_ key is missing from the interceptor definition or it contains an empty value or asterisk, then AOP/1 assumes that all methods on the bean should be intercepted.

## Helper Methods

**isLast()**
This method is automatically added to any **around** interceptor and will tell you if the interceptor is the last in the execution chain.

**translateArgs(any targetBean, string methodName, struct args, boolean replace)**
This method is automatically added to any interceptor and will attempt translate position based arguments into name based arguments.  This method has a _replace_ argument that when set to **true** will replace the _args_ with a copy of named arguments.

## Intercepting Cross Object Calls & Private Methods

Unlike some other AOP frameworks, AOP/1 has the ability to intercept cross object calls.  What this means is, if you are intercepting methods (_method1, method2, method3_) on _myService_ and _method2_ actually makes a call to _method3_, then AOP/1 will intercept the call from _method2_ to _method3_ in addition to original call to _method2_.

In addition to intercepting cross object method calls, AOP/1 can also intercept calls to private methods.

## Multiple Interceptor

You may find yourself creating an interceptor that performs multiple of similar tasks and it is logical to group multiple different interceptor types together.  This can be accomplished by simply creating the correct methods in the same component.  For instance, if you have an interceptor that you want to perform both before and after interceptions, then you simply add both the _before_ and _after_ methods to the component.  AOP/1 will place an interceptor in multiple execution stacks if it has more than one interceptor type method present.

## Stack Execution

Stacks are executed in the following order.

* before
* around
* after
* onError

All the stacks will only execute if there is an interceptor of their type present.  If the stack is emtpy, nothing is executed.  The **onError** stack only executes if there is an error in the execution of the other stacks.  The **before** and **after** stacks execute like a queue and will execute from start to finish regardless of changes to the arguments or result, skipping any interceptors that do not match the currently intercepted bean method.  The **around** stack executes more like a chain.  The chain execution can be stopped by not calling the _proceed()_ method.


