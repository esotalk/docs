# Messages

## Notification Messages

esoTalk provides an API to easily show standardized **notification messages** to the user. In the default skin, these messages appear at the bottom-left corner of the screen. They may be dismissed by the user, or can be set to automatically disappear.

Messages are collected by the ETController using the `message` method.

**Showing A Message**

	$this->message("This is the message text.");
	
**Applying A CSS Class To A Message**

	$this->message(T("message.conversationDeleted"), "success");
	$this->message(T("message.postNotFound"), "warning");
	$this->message("This will disappear after 20 seconds.", "success autoDismiss");
	$this->message("The user will not be able to dismiss this.", "warning undismissable");
	
Additional options may be specified by passing an array as the second argument. These include:

- `className`: a CSS class to apply to the message.
- `id`: an ID for the message. If specified, the message will overwrite any previous messages with the same ID.
- `callback`: a JavaScript function to run when the message is dismissed. 
	
An array of messages may be shown using the `messages` method. This is particularly useful in conjunction with the `ETModel::errors` method. The key of each element in the array will be used as the ID of the message if it is not numeric, while the value will be used as the message text. Like with the `message` method, a second argument can be passed to the `messages` method to specify options for all of the messages.
	
**Showing A Collection Of Messages**

	$messages = $model->errors();
	$this->messages($messages, "warning");

A [JavaScript API](/docs/javascript#messages) is available for showing and hiding messages after the page has loaded. Note that messages will automatically be shown on any ETAjax requests if the [response type](/docs/controllers#response-types) is "ajax" or "json".

## Modal Messages

Sometimes you may wish to display a **modal message** to the user, where they must dismiss a [sheet](/docs/javascript#sheets) before continuing to use the application. This can be done with the `renderMessage` method on ETController. Behind the scenes, this method simply sets up a standard view that will display the passed title and message on a sheet, and renders it.

**Showing A Modal Message**

	$this->renderMessage("Success", "This is the message");
	
The `render404` method can also be used to display a standard "Page Not Found" sheet and render the page with a 404 HTTP response code.

**Showing A Page Not Found Message**

	$this->render404(T("message.conversationNotFound"));