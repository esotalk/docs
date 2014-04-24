# JavaScript

- [Data](#data)
- [AJAX](#ajax)
- [Sheets](#sheets)
- [Popups](#popups)
- [Messages](#messages)
- [Tooltips](#tooltips)

> esoTalk's JavaScript is built on the [jQuery framework](http://jquery.com), as well as a number of jQuery plugins.

<!-- -->

> For information about including JavaScript files on the page, see [Controllers](/docs/controllers#asset-management). 

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

In esoTalk, "sheets" are modal dialogs. They are usually constructed surrounding a particular task, such as logging in, registering, or managing members who are allowed to view a conversation.

A single sheet should be defined in a view, as marked-up below. The sheet can then be rendered both in the context of the "default" repsonse type, or on its own (and thus as a modal dialog) through the "view" or "ajax" response types (see [Response Types](/docs/controllers#response-types).)

**HTML Mark-Up For A Sheet**

	<div id='exampleSheet' class='sheet'>
		<div class='sheetContent'>

			<h3>Sheet Title</h3>

			<div class='sheetBody'>Sheet Body</div>

			<div class='buttons'>
				<a href='#' class='button'>OK</a>
			</div>

		</div>
	</div>

After a controller method has been defined which renders the sheet view (see [Controllers](/docs/controllers)), the sheet can be loaded and hidden using methods on the `ETSheet` object. The `loadSheet` method accepts four parameters:

1. The ID of the sheet, which must correspond to the ID of the outer `<div class='sheet'>` in the view
2. The URL that will render the sheet
3. A callback to be run immediately after the sheet is displayed
4. An object containing data to POST with the request (optional)

**Asynchronously Loading A Sheet**

	ETSheet.loadSheet("exampleSheet", "controller/example.ajax", function() {
		// callback run immediately after the sheet is displayed
	});

**Hiding A Sheet**

	ETSheet.hideSheet("exampleSheet");

### Forms

A sheet that contains a form can be submitted via AJAX so that the page doesn't have to be reloaded. This is achieved with the help of a small jQuery plugin called `ajaxForm`. This plugin binds to a form's submit event, serializing all of the form's data and passing it to a user-defined callback. In code:

**Asynchronously Loading A Sheet And Handling Its Form**

	// Define a function that asynchronously loads the sheet with form data.
	function loadExampleSheet(formData) {

		ETSheet.loadSheet("exampleSheet", "controller/example.ajax", function() {

			$(this).find("form").ajaxForm(
				"submit", // The name of the button that should be "clicked" when this form is submitted
				loadExampleSheet // Callback to pass form data to
			);

		}, formData);

	}

If the form's submission is a success, then there are two things that the controller may do to respond:

* [Redirect]() to another page, or
* NOT render the sheet view, which will cause the sheet to be closed.

Of course, additional JSON data can be returned in the response, and you may like to handle this data. This can be done by passing another callback function as the final argument to `ETSheet.loadSheet`.

**Processing Additional Data When Loading A Sheet**

	ETSheet.loadSheet("exampleSheet", "controller/example.ajax", function() {
		// ...
	}, formData, function(data) {
		if (data.example) alert(data.example);
	});

<a name="popups"></a>
## Popups

In the interest of degradability, esoTalk uses JavaScript to transform lists of links and controls into togglable popup menus. A standard list of controls rendered in a view, typically using the ETMenu class, may look like this:

	<ul id='exampleControls' class='controls'>
		<li><a href='#'>Edit</a></li>
		<li><a href='#'>Delete</a></li>
	</ul>

This list can be transformed into a single `<a class='button'>` element using a jQuery plugin called `popup`. When the button is clicked, the list shows as a popup menu below the button. 

**Transforming A List Into A Popup**

	$("#exampleControls").popup({
		alignment: "left",
		content: "<i class='icon-cog'></i> <span class='text'>"+T("Controls")+"</span> <i class='icon-caret-down'></i>"
	}).appendTo("body");

<a name="messages"></a>
## Messages

esoTalk provides a central area on the page for messages to be displayed, as discussed in [Messages](/docs/messages). Messages can also be displayed and destroyed in JavaScript. 

**Showing A Message**

	ETMessages.showMessage("message text", {className: "warning dismissable", id: "exampleMessage"});

**Hiding A Message**

	ETMessages.hideMessage("exampleMessage");

<a name="tooltips"></a>
## Tooltips

esoTalk's tooltip library improves upon the default browser tooltips shown by setting the title attribute on any element. It allows customisation of positioning and timing. 

**Applying A Custom Tooltip To An Element**

	// HTML
	<a href='#' title='This is a tooltip' rel='tooltip'>Foo</a>

	// JS
	$("a[rel=tooltip]").tooltip({
		className: "withArrow withArrowBottom", // apply a class name to the tooltip element
		alignment: "left", // align the tooltip relative to the element. null (center) | left | right
		offset: [0, 5], // offset the tooltip by [left, top]
		delay: 250 // milliseconds before the tooltip is shown after the element is hovered
	});
