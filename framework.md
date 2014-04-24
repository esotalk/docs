# Framework Overview

- [ET](#et)
- [Creating Objects](#factory)
- [Request Lifecycle](#request-lifecycle)

esoTalk is built upon a pseudo-framework tailored specifically to suit its needs. This allows the codebase to be efficient and concise. This section of the documentation covers the workings of this framework and its various components.

> The esoTalk framework was built with inspiration from [Garden](https://github.com/vanillaforums/Garden).

<a name="et"></a>
## ET

The center of esoTalk's framework is an important class called **ET**. This class has [numerous static functions and properties](/api/class-ET.html) that serve as a central way to access various things which are needed around the application, such as singletons and configuration values.

<a name="factory"></a>
## Creating Objects

**ETFactory** is a class which facilitates the instantiation of new objects. Utility classes and controllers must be registered with the factory using the [`ETFactory::register`](/api/class-ETFactory.html#_register) method, after which they can be instantiated using [`ETFactory::make`](/api/class-ETFactory.html#_make). This enables plugins to replace default classes with their own custom versions; for more information, see [Plugins](/docs/plugins).

**Registering A Class With The Factory**

	ETFactory::register("foo", "FooClass", "foo.class.php");
	
**Instantiating An Object**

	$foo = ETFactory::make("foo");
	
The `ET` class provides a wrapper around `ETFactory` to lazily instantiate and access singleton objects.

**Lazily Instantiating A Singleton Using The ET Class**

	$foo1 = ET::getInstance("foo");
	$foo2 = ET::getInstance("foo");
	// $foo1 === $foo2

<a name="request-lifecycle"></a>
## Request Lifecycle

With the above basic concepts under your belt, you can now begin to understand the lifecycle of each request to the framework. This process takes place in `core/bootstrap.php`:

1. **Set up the environment.** Define constants, set up error handling, etc.
2. **Include the configuration files.**
3. **Register default classes** with the `ETFactory`, including helpers, models, and controllers.
4. **Include and instantiate plugins.**
5. **Instantiate the database and session objects**, and set them as properties of the static `ET` class.
6. **Include the skin and language.**
7. **Parse the request string and dispatch to the appropriate controller.**
