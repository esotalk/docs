# Common Tasks

- [Asset Management](#assets)
- [Render Functions](#render)
- [Plugin Settings](#settings)
- [Administration Panes](#admin-panes)
- [Controllers & Models](#controllers-models)

<a name="assets"></a>
## Asset Management

Usually the `init` event (or any event prefixed with `init`) is a good time to [add resources](/docs/controllers#assets) to the page. Depending on the pages on which you need the resources to be loaded, you can restrict your event handler to a specific controller.

**Adding Global Resources**

	public function handler_init($controller)
	{
		$controller->addCSSFile($this->resource("sidebar.css"), true);
		$controller->addJSFile($this->resource("sidebar.js"), true);
	}
	
**Adding Resources Specific To A Controller**

	public function handler_conversationController_init($controller)
	{
		$controller->addCSSFile($this->resource("emoticons.css"));
	}

<a name="render"></a>
## Render Functions

esoTalk contains a [number of functions](https://github.com/esotalk/esoTalk/blob/master/core/lib/functions.render.php) which are used to render and format reusable user interface elements such as stars, avatars, labels, and member names. The default implementations of these are defined in `core/lib/functions.render.php`.

You can override any one of these functions with your own implementation by simply defining it before the `functions.render.php` file is included — the `boot` function is such a place. 

**Overriding The `avatar` Function**

	public function boot()
	{
		// Make sure the avatar function doesn't already exist, just in case another plugin has overridden it. 
		if (!function_exists("avatar")) {
			function avatar(...)
			{
				//...
			}
		}
	}

<a name="settings"></a>
## Plugin Settings

Setting up a basic settings sheet for your plugin is easy. First, [define a `settings` method](/docs/plugins/concepts#settings) in your plugin class. In it, you should [create an `ETForm` object](/docs/forms) and handle a submission by [writing configuration values](/docs/config).

**Defining A Settings Method**

	public function settings($controller)
	{
		// Set up the settings form.
		$form = ETFactory::make("form");
		$form->action = URL("admin/plugins/settings/ExamplePlugin");
		$form->setValue("description", C("plugin.ExamplePlugin.description"));

		// If the form was submitted...
		if ($form->validPostBack()) {

			// Save configuration values.
			ET::writeConfig(array(
				"plugin.ExamplePlugin.description" => $form->getValue("description")
			));

			$controller->message(T("message.changesSaved"), "success autoDismiss");
			$controller->redirect(URL("admin/plugins"));
		}

		$controller->data("settingsForm", $form);
		return $this->view("settings");
	}
	
Next, you'll need to create a `views/settings.php` in your plugin folder and construct your plugin's settings sheet. Form elements can be outputted using [methods on your form object](/docs/forms#elements).
	
**Marking Up A Settings Sheet**

	<?php
	$form = $data["attachmentsSettingsForm"];
	echo $form->open();
	?>
	<div class='section'>
		<ul class='form'>
			<li>
				<label>Description</label>
				<?php echo $form->input("description", "text"); ?>
			</li>
		</ul>
	</div>
	<div class='buttons'>
		<?php echo $form->saveButton(); ?>
	</div>
	<?php echo $form->close(); ?>

<a name="admin-panes"></a>
## Administration Panes

When your plugin requires an administration interface that is too complex for a single sheet, you can add a whole new administration pane. 

Each administration pane has its own controller which extends the `ETAdminController` class. It works just like any other [controller](/docs/controllers), except for two differences:

1. Views are rendered within a wrapper  view that shows the administration menu on the side (only for the default [response type](/docs/controllers#response-types).)
2. It responds to a URL prefixed with "admin/".

Your administration controller must be [registered](/docs/framework#factory) using the `ETFactory::registerAdminController` method.

**Registering An Administration Controller**

	public function boot()
	{
		ETFactory::registerAdminController("example", "ETExampleAdminController", $this->file("ETExampleAdminController.class.php")); // Responds to admin/example/...
	}

You'll probably also want to add an item to the administration [menu](/docs/menus). This can be achieved by listening for the `initAdmin` event.

**Adding An Item To The Administration Menu**

	public function handler_initAdmin($controller, $menu)
	{
		$menu->add("example", "<a href='".URL("admin/example")."'><i class='icon-cog'></i> Example</a>");
	}

<a name="controllers-models"></a>
## Controllers & Models

If your plugin introduces a new type of entity (e.g. attachments, events,) it is good practice to group all of the logic surrounding this entity into its own [controller](/docs/controllers) and [model](/docs/models).

These can be registered with the factory in the boot method.

**Registering A Controller & Model**

	public function boot()
	{
		ETFactory::register("exampleModel", "ExampleModel", $this->file("ExampleModel.class.php"));
		ETFactory::registerController("example", "ExampleController", $this->file("ExampleController.class.php"));
	}

Your model should extend the `ETModel` class to inherit [basic model functionality](/api/class-ETModel.html). You can get an instance of the model within your controller using the `ETFactory::make` method. 

**Getting An Instance Of A Custom Model**

	$model = ETFactory::make("exampleModel");

You can access your plugin object (and thus its methods like `view` and `resource`) from within a controller or view through the `ET::$plugins` array.

**Accessing The Plugin Object From Within A Controller**

	$plugin = ET::$plugins["ExamplePlugin"];
	$this->render($plugin->view("example"));
