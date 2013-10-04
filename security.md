# Security

- [Sanitizing Data](#sanitizing-data)
- [CSRF Protection](#csrf-protection)

The esoTalk framework provides a number of functions and strategies that make security very easy to implement.

<a name="sanitizing-data"></a>
## Sanitizing Data

**Sanitizing Data Being Rendered In A HTML Response**

	echo sanitizeHTML("<b>Test</b>"); // &lt;b&gt;Test&lt;/b&gt;

**Sanitizing Data Being Outputted Within HTTP Headers**

	$headers = "Location: ".sanitizeForHTTP($url);
	
**Sanitizing Data Being Used For A File Name**

	$path = "dir/".sanitizeFileName($filename);

<a name="csrf-protection"></a>
## CSRF Protection

CSRF protection is implemented using a randomly-generated token. This token is available as a property on the ETSession class, via `ET::$session->token`.

**Creating A Link With CSRF Protection**

	$url = URL("conversation/delete/".$conversationId."?token=".ET::$session->token);
	
The token can then be checked to make sure the request is valid using the `ETController::validateToken` method. This method will assume that the token is submitted in the request data with the "token" key. If token validation fails, a [notification message](/docs/messages) will be added and the method will return false.

**Checking If The Request Is Valid**

	if (!$this->validateToken()) return;

CSRF protection is automated when setting up a form using [ETForm](/docs/forms). The token is added as a hidden input to the form automatically; all you have to do is check for a valid post back before processing any data:

**Checking If A Form Submission Is Valid**

	if ($form->validPostBack("save")) {
		// token is valid
	}

[ETAjax](/docs/javascript#ajax) automatically sends the token in the request data, so you don't need to add it manually to every AJAX request.