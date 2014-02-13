# Configuration

- [Custom CSS](#custom-css)
- [Friendly URLs](#friendly-urls)

esoTalk has many configurable options that allow you to get your forum working just the way you want. Some of the basic options can be set in the **Forum Settings** part of the Administration panel.

Configuration options specific to your esoTalk installation are stored in the `config/config.php` file. The installer will create this file and fill out your database information, forum title, and other essential values.

There are a number of more advanced options that you may wish to change. These are listed in the `core/config.defaults.php`. Writing your custom values options directly to this file is not recommended because your changes will be overwritten each time you upgrade esoTalk.

Instead, copy the line you wish to change, paste it in your forum's `config/config.php` file, and set your custom value there. For example, if you wish to turn debug mode on, add the following line to `config/config.php`:

	$config["esoTalk.debug"] = true;

<a name="custom-css"></a>
## Custom CSS

You may wish to apply some custom CSS styles to your forum without wanting to go to the trouble of [creating a new skin](/docs/skins). You can do so by adding your custom CSS to the `config/custom.css` file. If this file is not empty, esoTalk will automatically load it in each page.

<a name="friendly-urls"></a>
## Friendly URLs

esoTalk supports three forms of URLs:

| esoTalk.urls.friendly | esoTalk.urls.rewrite | URL format |
| --- | --- | --- |
| `false` | `false` | forum.com/?p=member/123 |
| `true` | `false` | forum.com/index.php/member/123 |
| `true` | `true` | forum.com/member/123 |

The third option requires your webserver to rewrite URLs to index.php. If you're using Apache and mod_rewrite is enabled, the esoTalk installer will automatically create a .htaccess file for you. If you're using another webserver, you'll need to manually configure URL rewriting.

### Apache

	<IfModule mod_rewrite.c>
	RewriteEngine On
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^(.*)$ index.php/$1 [QSA,L]
	</IfModule>

### Nginx

	location / {
		try_files $uri $uri/ /index.php/$args;
	}
	
### Lighttpd

	url.rewrite-if-not-file = (
		"^(.*)$" => "index.php/$1"
	)