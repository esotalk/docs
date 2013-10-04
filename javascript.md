# JavaScript

- [Data](#data)
- [AJAX](#ajax)
- [Sheets](#sheets)
- [Popups](#popups)
- [Messages](#messages)
- [Tooltips](#tooltips)

> esoTalk's JavaScript is built on the [jQuery framework](http://jquery.com), as well as a number of jQuery plugins.

<a name="data"></a>
## Data

ETController provides a way to pass data to the page so that it will be easily accessible in the page's JavaScript. Simply call the `addJSVar` method and pass a key and a value. The data will be accessible in JavaScript via the `ET` object.

**Adding Data To Be Accessible Via JavaScript**

	$conversationId = 123;
	$this->addJSVar("conversationId", $conversationId);
	
**Accessing JavaScript Data**
	
	var conversationId = ET.conversationId; // 123
	
You can also conveniently add language definitions to be accessible via JavaScript using the `addJSLanguage` method. This method accepts unlimited arguments as language definition keys. Translation can then be performed in JavaScript using the `T` function, which works the same way as the [PHP function](/docs/localization).

**Adding Language Definitions To Be Accessible Via JavaScript**

	$this->addJSLanguage("Followed", "Unfollowed", "Sticky");

<a name="ajax"></a>
## AJAX

In the spirit of being fast and responsive, esoTalk is an AJAX-heavy application. esoTalk provides a wrapper around [jQuery's ajax method](http://api.jquery.com/jQuery.ajax/) called `ETAjax`. This wrapper automatically adds essential data to every AJAX request (like the [session token](/docs/security#csrf-protection)), automates handling of various components that may be returned in an AJAX request (like messages and redirects), and handles disconnection errors (showing an error message and allowing the user to reattempt the failed AJAX request.)

To take full advantage of this functionality, you must request the [AJAX response type](/docs/controllers#response-types). Simply append `.ajax` to the controller method in the URL. You can read more about response types in [Controllers](/docs/controllers#response-types).

**Making An AJAX Request**

	$.ETAjax({
		url: "conversation/reply.ajax/123",
		type: "post",
		data: {content: "HELLO THAR"},
		success: function(data) {
			// data = {postId: "136", starOnReply: true, view: "...", messages: []}
		}
	});

On top of the [options available to jQuery's ajax method](http://api.jquery.com/jQuery.ajax/), you may specify the following options: 

| Option | Type | Description |
| --- | --- | --- |
| `url` | String | The URL, relative to the forum's base URL, to make the AJAX request to. |
| `data` | Object or String | Data to be sent to the server. |
| `type` | String | The type of request to make ("POST" or "GET"). Default: "GET" | 
| `id` | String | An optional identifier for the AJAX request. If specified, all previous AJAX requests with the same ID will be aborted. |
| `global` | Boolean | Whether or not this AJAX request will trigger global event handlers – that is, whether or not it will show the generic "loading" indicator at the top of the screen. Default: true |
| `success` | Function | A function to be called if the request succeeds. The data returned from the server is passed as the first argument. |
| `error` | Function | A function to be called if the request fails. |
| `complete` | Function | A function to be called when the request finishes (after success and error callbacks are executed). |
| `beforeSend` | Function | A pre-request callback function. |

<a name="sheets"></a>
## Sheets