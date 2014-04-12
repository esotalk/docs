# Menus

Sometimes it is necessary to be able to dynamically construct lists of HTML links or buttons that can be altered before they are outputted. This is especially necessary in giving plugins the power to add functionality to the user interface.

The **ETMenu** class provides an easy way to collect, manipulate, and output a list of items. A new instance of the class should be created using the `ETFactory`.

**Creating A New Menu**

	$menu = ETFactory::make("menu");
	
Items may be appended to or removed from the menu by specifying a string to identify the menu item, and the HTML content of the menu item.

**Adding A Menu Item**

	$menu->add("login", "<a href='".URL("user/login")."'>Log in</a>");

**Removing A Menu Item**

	$menu->remove("login");

Menu items can also be added at a specific position by providing a third argument. This can be either an absolute index at which to insert the item, or a position relative to another menu item.

**Adding A Menu Item At A Specific Position**

	$menu->add("foo", "Foo");
	$menu->add("bar", "Bar", 0);
	$menu->add("baz", "Baz", array("after" => "bar"));
	$menu->add("qux", "Qux", array("before" => "bar"));
	// Order of items would be: Qux, Bar, Baz, Foo

Separator items can be added using the `separator` method. A separator will be outputted as an empty `<li class='sep'>` element.

**Adding A Separator**

	$menu->separator(array("after" => "bar"));
	
One or more menu items may be flagged as "highlighted"; these items are outputted with the `selected` class. This is useful, for example, if constructing a list of tabs where one tab is active.

**Highlighting A Menu Item**

	$menu->highlight("baz");

The menu HTML can be retrieved as a string of `<li>` elements using the `getContents` method. Sometimes it may be necessary to check if any menu items have been collected using the `count` method.

**Retrieving Menu HTML**

	if ($menu->count()) {
		echo "<ul>".$menu->getContents()."</ul>";
	}
	
For the menu constructed over the last few examples, the output will be:

	<ul>
		<li class='item-qux'>Qux</li>
		<li class='item-bar'>Bar</li>
		<li class='sep'></li>
		<li class='item-baz selected'>Baz</li>
		<li class='item-foo'>Foo</li>
	</ul>

A [JavaScript API](/docs/javascript#popups) is available for converting plain menus into popup menus after the page has loaded.

