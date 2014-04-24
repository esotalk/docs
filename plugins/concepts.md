# Concepts

- [Lifecycle](#lifecycle)
- [Events](#events)
- [Actions](#actions)
- [Views](#views)
- [Resources](#resources)
- [Settings](#settings)

A plugin class should be defined in `plugin.php`. The class name must be the name of the plugin (same as the plugin folder) prefixed with `ETPlugin_`. It must extend the `ETPlugin` class. It will contain methods which encapsulate all of the first-level plugin logic — installation, settings, initialization, and hooking into esoTalk's code.

**Defining A Plugin Class**

	class ETPlugin_ExamplePlugin extends ETPlugin {
		// ...
	}

<a name="lifecycle"></a>
## Lifecycle

### Boot

On every page load, all enabled plugin classes are instantiated before even the session and database objects are created (see [Request Lifecycle](/docs/framework#request-lifecycle).) Immediately after each plugin instantiated, its `boot` method is called. This is an opportunity to perform any low-level bootstrapping tasks, such as [registering classes with the factory](/docs/plugins/common-tasks#controllers-models).

### Init

Later on in the request lifecycle, after all of esoTalk's core objects have been instantiated, and just before the request is dispatched to the controller, the `init` method is called on each plugins. This is a chance to perform a number of tasks:

- Define default [language translations](/docs/localization)
- Register [labels](/docs/plugins/conversations#labels), [gambits](/docs/plugins/search#gambits), [activity types](/docs/plugins/activity#types), and [last action types](/docs/plugins/members#last-action-types)
- Include and [override render functions](/docs/plugins/common-tasks#render)

### Setup

Every time a plugin is enabled, the `setup` method is called. It is also called automatically when a new version of the plugin is detected. This method provides an opportunity to do things such as [manipulate the database schema](/docs/database#schema) and initialize filesystem structures.

The old version number of the plugin is passed as an argument to the `setup` method. If the plugin is being enabled for the first time, this will be empty. If it is being upgraded from an old version, its value will be the old version number. This argument can be compared with the current version of the plugin to regulate upgrade tasks.

The `setup` method must return `true` in order for the plugin to be enabled. If the return value is not `true`, an error message will be displayed with the contents of the return value, and the plugin will not be enabled.

### Disable

The `disable` method is called when the plugin is disabled. It should perform any non-destructive cleanup tasks, such as removing temporary caches.

### Uninstall

The `uninstall` method is called when the plugin is uninstalled. It should perform any permanent cleanup tasks, such as removing database tables.

<a name="events"></a>
## Events

Events are central to how plugins work in esoTalk. At many specific points throughout esoTalk's code, events are triggered. For example, after a post is edited in the `ETPostModel` class, the "editPostAfter" event is triggered. Plugins have the ability to listen for these events, and to perform work when they are triggered.

To listen for and subsequently handle events, strategically-named class methods are used. Event handler methods begin with the `handler_` prefix, followed by the name of the event to listen for.

**Handling The `editPostAfter` Event**

	public function handler_editPostAfter($sender, $post)
	{
		// ...
	}

The first parameter (`$sender`) passed to the event handler is the instance of the class that triggered it (in this case, the post model.) Subsequent parameters depend on the event, and are detailed in the [List of Events](/docs/plugins/events).

> Parameters prefixed with an ampersand (`&`) are referenced variables, meaning any alterations made locally will also be made to the original copy used in the core.

<!-- -->

> Some events may also be affected by the values returned from handlers. These are described in more detail per-event in the List of Events. 

The same event can be triggered by multiple different classes. For example, the `init` event is triggered by every `ETController` subclass. Often, however, you may only wish to handle an event triggered by a certain class. This can be achieved by putting the factory name of the class after the `handler_` prefix.

**Handling The `init` Event For A Certain Controller**

	public function handler_conversationController_init($sender)
	{
		// ...
	}

### Triggering

You can also trigger your own events within your plugin code, allowing other plugins to extend your own.

**Triggering An Event**

	$this->trigger("eventName", array($parameter1, $parameter2));

Events triggered by a plugin class can be handled generally using `handler_eventName`, or handled exclusively using `handler_ETPlugin_ExamplePlugin_eventName`.


<a name="actions"></a>
## Actions

As outlined in [Controllers & Views](/docs/controllers), the request URL (e.g. member/name/Toby) maps to a specific method, or action, (name) in a specific controller (ETMemberController). Plugins have the ability to override default action methods and define new ones. 

Like event handlers, overriding actions is achieved using strategically-named class methods. Action methods begin with the `action_` prefix, followed by the factory name of the controller, an underscore, and then the name of the action to handle (i.e. the second parameter of the URL.)

**Overriding An Action**

	public function action_conversationController_index($controller, $conversationId)
	{
		// ...

		// You can still run the default index action if you like.
		$controller->index();
	}

New actions that aren't already handled by a core controller can also be defined in the same way. However, if multiple actions are being defined which are relevant to a new kind of entity, registering a new controller may be more suitable. 

### Action Events

Before and after the controller action is called, two events are triggered. These events can be handled using `handler_{controllerName}_{method}_before` and `handler_{controllerName}_{method}_after` class methods respectively. The before event is special in that if it returns `false`, the dispatch process will be halted and the action will not be called. 

**Handling An Action Before Event**

	public function handler_conversationController_index_before($controller, $conversationId)
	{
		return false;
	}

<a name="views"></a>
## Views

esoTalk's core views are located in `core/views`, as discussed in [Controllers & Views](/docs/controllers#views). These views can be easily overridden by plugins. Simply place the appropriate indentical file/folder structure in the `views` directory of a plugin, and esoTalk will scan for and find it automatically when the view is rendered.

**File/Folder Structure To Override A View**

	ExamplePlugin/
	--	views/
	------	conversation/
	----------	index.php

Custom views may also be placed in this folder, and rendered using the full path to the view file. This can be retrieved using the `view` method on the plugin class.

**Rendering A Custom View**

	$controller->render($this->view("custom"));

<a name="resources"></a>
## Resources

A plugin's publicly-accessible resources (CSS/JS files, images, etc.) are stored in the `resources` folder. To make use of them (see [Asset Management](/docs/plugins/common-tasks#assets)) they must be referred to by their full path. This can be retrieved using the `resource` method on the plugin class.

**Getting The Path To A Plugin Resource**

	echo $this->resource("custom.css"); // addons/plugins/ExamplePlugin/resources/custom.css

<a name="settings"></a>
## Settings

Plugins may provide a settings sheet which is displayed when the Settings button on the Plugins administration interface is clicked. This is a great place to allow the administrator to set basic configuration options.

In order to enable the Settings button and display a sheet, a plugin must have a `settings` method. This works similarly to any normal controller action, with one important difference: instead of rendering the view, it should return the full path to the view as a string (using the `view` method.)

For an example of how to construct a settings form and save configuration values in the context of a settings sheet, see [Common Tasks](/docs/plugins/common-tasks#settings).

For more complex administration interfaces, a [custom administration pane](/docs/plugins/common-tasks#admin-panes) may be more suitable.
