# Languages

- [Packaging](#packaging)
- [Details](#details)
- [Translation](#translation)
- [Submission](#submission)

A language is a collection of files which can be dropped in to any esoTalk installation to translate it into another language. Languages are stored in the `addons/languages` folder.

<a name="packaging"></a>
## Packaging

A language has the following file structure:

| File/Folder | Description |
| -- | -- |
| `ExampleLanguage/` | The name of the plugin in StudlyCase. |
| `--icon.png` | A 16x16 icon which is displayed on the Administration page. |
| `--definitions.php` | The main language file, where all of the languages's core translations are located. |
| `--definitions.{plugin}.php` | Additional files containing translations for specific plugins. |
| `--index.html` | An empty file to prevent the directory from being publicly listed. |

<a name="details"></a>
## Details

A languages's `definitions.php` file must contain an array of details about the language (name, author, etc.) These details should be added to the `ET::$languageInfo` array. The key must correspond to the name of the languages's folder.

**Defining Language Details**

	ET::$languageInfo["ExampleLanguage"] = array(
		"locale"      => "en-US",
		"name"        => "Example Language",
		"description" => "An example language pack.",
		"version"     => "1.0",
		"author"      => "Toby Zerner",
		"authorEmail" => "toby@esotalk.org",
		"authorURL"   => "http://esotalk.org",
		"license"     => "GPLv2"
	);

<a name="translation"></a>
## Translation

The best way to get started in creating a new language pack is to copy and paste the whole `English` folder. Rename it to something else (we'll use `Pirate`.) Now, open up `definitions.php`.

First things first — let's change all instances of the word "English" to our new name, "Pirate". The most important one is just after `ET::$languageInfo` on the fifth line. You can change all the other details as you please!

Looking further down, you will see a whole bunch of lines that look something like this:

	$definitions["Cancel"] = "Cancel";

The string on the left-hand side is called the "key". Don't change it! The part that you want to edit is after the `=` symbol. So if I wanted cancel buttons to say "NARRR", I would type:

	$definitions["Cancel"] = "NARRR";

Done! (Make sure you leave the quotes intact and the semi-colon at the end of the line.)

Sometimes you'll see funny things like `%s` symbols or some programming jargon. This usually just means that something more meaningful will be substituted in later. A good rule of thumb is to leave it be! For example:

	$definitions["%s joined the forum."] = "%s joined the crew.";

<a name="submission"></a>
## Submission

Ideally, you should version-control your language and create a repository on [GitHub](http://github.com). The `definitions.php` file should be at the top-level of the repository; there should be no parent folder.

Languages can be submitted for public listing in the [Languages](/languages) channel on the esoTalk support forum. Read [this conversation]() for language submission guidelines.
