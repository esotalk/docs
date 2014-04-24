# Formatting

esoTalk provides a class called `ETFormat` which is responsible for converting plain-text input to HTML output. The formatter must first be initialized with a string of text to be formatted. To format the string, simply call the `format` method. You can then retrieve the formatted HTML using the `get` method. Methods can be chained.

**Formatting A String**

	$text = "Foo\n\nBar\n\ngoogle.com";

	$formatter = ET::formatter();
	$formatter->init($text)->format();

	$html = $formatter->get(); // <p>Foo</p><p>Bar</p><p><a href='http://google.com'>google.com</a>

By default, the following formatting methods are called in order by the `format` method. They can also be called individually.

| Method | Description |
| --- | --- |
| `mentions` | Convert `@mentions` into user profile links |
| `quotes` | Convert `[quote]` tags into `<blockquote>`s |
| `links` | Convert URLs and email addresses into hyperlinks |
| `lists` | Convert bulleted and numbered lists into `<ul>`s and `<ol>`s |
| `whitespace` | Convert whitespace into `<p>`s and `<br>`s |

Additional tasks can be performed on the string before HTML is retrieved.

**Highlighting Words In A String**

	$formatter->highlight(array("foo", "bar"));

**Clipping A String**

	$formatter->clip(200); // characters

The formatter has an "inline" mode which restricts what is formatted. In inline mode, only unobtrusive inline formatting (hyperlinks) will take place. To turn inline mode on, use the `inline` method on the formatter after initialization.

**Turning On Inline Mode**

	$formatter->inline(true);
