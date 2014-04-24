# Plugins

- [Packaging](#packaging)
- [Details](#details)
- [Dependencies](#dependencies)
- [Submission](#submission)

A plugin is a collection of code and resources which can be dropped in to any esoTalk installation to add additional functionality. Plugins are stored in the `addons/plugins` folder.

In addition to reading this documentation, a good way to learn how plugins work is to look through the code of [existing plugins](/plugins).

<a name="packaging"></a>
## Packaging

A plugin has the following file structure:

| File/Folder | Description |
| -- | -- |
| `ExamplePlugin/` | The name of the plugin in StudlyCase. |
| `--plugin.php` | The main plugin file, where all of the plugin's information and logic is located. |
| `--icon.png` | A 16x16 icon which is displayed on the Administration > Plugins page. |
| `--index.html` | An empty file to prevent the directory from being publicly listed. |
| `--resources/` | The plugin's publicly-accessible resources (CSS/JS files, images, etc.) |
| `--views/` | The plugin's views (see [Concepts](/docs/plugins/concepts#views).) |
| `--...` | Anything else you may need: class files, libraries, includes, etc. |

<a name="details"></a>
## Details

A plugin's `plugin.php` file contains two important things: an array of details about the plugin (name, author, etc.) and a class which encapsulates all of the plugin's logic (discussed on the next page.) 

Plugin details should be added to the `ET::$pluginInfo` array at the top of `plugin.php`. The key must correspond to the name of the plugin's folder.

**Defining Plugin Details**

	ET::$pluginInfo["ExamplePlugin"] = array(
		"name"        => "Example Plugin",
		"description" => "An example plugin.",
		"version"     => "1.0",
		"author"      => "Toby Zerner",
		"authorEmail" => "toby@esotalk.org",
		"authorURL"   => "http://esotalk.org",
		"license"     => "GPLv2"
	);

<a name="dependencies"></a>
## Dependencies

Dependencies may be specified as an array under the `dependencies` key. For each item in the dependencies array, the key may be "esoTalk" or the name of a required plugin, and the value is the minimum version required. If these dependencies are not met, enabling the plugin will not be allowed.

**Specifying Plugin Dependencies**

	ET::$pluginInfo["ExamplePlugin"] = array(
		// ...
		"dependencies" => array(
			"esoTalk"       => "1.0.0g3",
			"AnotherPlugin" => "1.3"
		)
	);

<a name="submission"></a>
## Submission

Ideally, you should version-control your plugin and create a repository on [GitHub](http://github.com). The `plugin.php` file should be at the top-level of the repository; there should be no parent folder.

Plugins can be submitted for public listing in the [Plugins](/plugins) channel on the esoTalk support forum. Read [this conversation]() for plugin submission guidelines.
