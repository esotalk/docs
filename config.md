# Configuration

As outlined in [Getting Started → Configuration](/docs/configuration), the configuration settings for an esoTalk installation are stored in the `core/config.defaults.php` and `config/config.php` files, with values defined in the latter taking precedence.

The esoTalk framework provides ways to easily access the values of these configuration settings, and to write new values to the installation's `config/config.php` file.

**Accessing A Configuration Value Or Returning A Default**

	$title = C("esoTalk.forumTitle", "Default Title");

> The `C()` function is an alias for [`ET::config()`](/api/class-ET.html#_config).

**Writing New Configuration Values**

	ET::writeConfig(array(
		"esoTalk.forumTitle" => "New Title",
		"esoTalk.language" => "Pirate"
	));
