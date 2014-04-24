# Concepts

- [Resources](#resources)
- [Views](#views)

A skin class should be defined in `skin.php`. The class name must be the name of the skin (same as the skin folder) prefixed with `ETSkin_`. It must extend the `ETSkin` class. It will contain a few methods which allow the skin to add its resources to the page.

**Defining A Skin Class**

	class ETSkin_ExampleSkin extends ETSkin {
		// ...
	}

<a name="resources"></a>
## Resources

Resources can be added to the page by setting up an `init` [event handler](/docs/plugins/concepts#events). Just define a method called `handler_init`:

	public function handler_init($controller)
	{
		// Add resources to the $controller here...
	}

Inside of this method, you can add as many resources to the controller as you want. Just be sure to wrap any filenames with the `resource` class method.

**Adding A CSS File**

	$controller->addCSSFile($this->resource("styles.css"), true);

<a name="views"></a>
## Views

esoTalk's core views are located in `core/views`, as discussed in [Controllers & Views](/docs/controllers#views). These views can be easily overridden by skins. Simply place the appropriate indentical file/folder structure in the `views` directory of a skin, and esoTalk will scan for and find it automatically when the view is rendered.

**File/Folder Structure To Override A View**

	ExampleSkin/
	--	views/
	------	default.master.php
