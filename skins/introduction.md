# Skins

- [Packaging](#packaging)
- [Details](#details)
- [Submission](#submission)

A skin is a collection of code and resources which can be dropped in to any esoTalk installation to change its appearance. Skins are stored in the `addons/skins` folder.

> Skins are actually just special types of [plugins](/docs/plugins)! A skin can technically do everything that a plugin can do, although it is recommended to keep functionality-changing code out of skins. Unlike plugins, only one skin can be activated at a time.

<a name="packaging"></a>
## Packaging

A skin has the following file structure:

| File/Folder | Description |
| -- | -- |
| `ExampleSkin/` | The name of the skin in StudlyCase. |
| `--skin.php` | The main skin file, where all of the skin's information and logic is located. |
| `--preview.jpg` | A 250x160 preview which is displayed on the Administration > Appearance page. |
| `--index.html` | An empty file to prevent the directory from being publicly listed. |
| `--resources/` | The skin's publicly-accessible resources (CSS/JS files, images, etc.) |
| `--views/` | The skin's views (see [Concepts](/docs/skins/concepts).) |

<a name="details"></a>
## Details

A skin's `skin.php` file contains two important things: an array of details about the skin (name, author, etc.) and a class which encapsulates all of the skin's logic (discussed on the next page.)

Skin details should be added to the `ET::$skinInfo` array at the top of `skin.php`. The key must correspond to the name of the skin's folder.

**Defining Skin Details**

	ET::$skinInfo["ExampleSkin"] = array(
		"name"        => "Example Skin",
		"description" => "An example skin.",
		"version"     => "1.0",
		"author"      => "Toby Zerner",
		"authorEmail" => "toby@esotalk.org",
		"authorURL"   => "http://esotalk.org",
		"license"     => "GPLv2"
	);

<a name="submission"></a>
## Submission

Ideally, you should version-control your skin and create a repository on [GitHub](http://github.com). The `skin.php` file should be at the top-level of the repository; there should be no parent folder.

Skins can be submitted for public listing in the [Skins](/skins) channel on the esoTalk support forum. Read [this conversation]() for skin submission guidelines.
