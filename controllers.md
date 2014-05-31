# Controllers & Views

- [Introduction](#introduction)
- [Rendering Views](#views)
- [Response Types](#response-types)
- [Asset Management](#assets)

## Introduction

In esoTalk, **controllers** are responsible for processing input and sending a response back to the browser. Following on from the [Request Lifecycle](/docs/framework#request-lifecycle), the basic process through which this is achieved is as follows:

1. **The request string is parsed.** (the "conversation/edit/123" part of "http://esotalk.org/forum/conversation/edit/123")
	- The first part maps to the *controller*.
	- The second part maps to the controller *method*, prefixed with `action_`, if it exists. Otherwise, we default to the `action_index` method.
	- Any subsequent parts map to *arguments* which are passed to the controller method.
2. **The controller object is created.** The appropriate class is instantiated using the `ETFactory`.
3. **The method and arguments are dispatched to the controller.** For "conversation/edit/123", we will be calling the `action_edit` method of the "conversation" controller, passing "123" as a function argument.
4. **The controller method processes input and sends a response**, usually using utility functions inherited from the `ETController` class.

**Defining A Controller**

	class FooController extends ETController {
	
		// Accessible via foo/index/Toby or simply foo/Toby
		public function action_index($name = "")
		{
			echo "Your name is ".($name ? $name : "Anonymous");
		}

	}
	
**Registering A Controller**

	ETFactory::registerController("foo", "FooController", "FooController.class.php");

<a name="views"></a>
## Rendering Views

**Views** contain the HTML of esoTalk, separating the controller logic from the presentation logic. The [ETController](/api/class-ETController.html) class provides functions to easily include and output views.

The [ETController::render](/api/class-ETController.html#_render) function takes the name of a view as an argument and, by default, passes the contents of this view to the master "layout" view (located at `core/views/default.master.php`.)

**Rendering A View**

	$this->render("foo/bar"); // core/views/foo/bar.php will be included in the content area of the master view
	
**Passing Data To A View**

	$this->data("name", "Toby");
	$this->render("member/profile");

**Accessing Data In A View**

	<h1><?php echo $data["name"]; ?>'s Profile</h1>

**Redirecting To Another Location**

	$this->redirect("conversation/123");

**Rendering A 404 Not Found Message**
	
	$this->render404("The member was not found.");

<a name="response-types"></a>
## Response Types

By default, views are rendered within the master view, suitable for a client accessing a normal HTML page in their browser. However, especially with XMLHTTPRequest calls, a different response format (such as JSON) may be desired.

esoTalk allows a **response type** to be specified in the *method* part of the request URI, following a period (`.`) character. The appropriate value will be assigned to the `ETController::$responseType` property, and the `ETController::render` method will behave according to the value of this property.

### Default

The default response type (`RESPONSE_TYPE_DEFAULT`) renders the specified view within the master view with a Content-Type of `text/html`. If your method is not intended to have special API or XMLHTTPRequest functionality, you may wish to force the default response type to prevent others from being elicited.

**Forcing The Default Response Type**

	$this->responseType = RESPONSE_TYPE_DEFAULT;

### View

Sometimes you may wish to retrieve only the contents of the specific view being rendered, without the master view around it (for example, when displaying a [Sheet](/docs/javascript#sheets) with XMLHTTPRequest.) This is possible with the "view" response type (`RESPONSE_TYPE_VIEW`).

**Example:** <http://esotalk.org/forum/conversation/index.view/3>

### JSON

As you would expect, the "json" response type (`RESPONSE_TYPE_JSON`) renders data as a JSON object with a Content-Type of `application/json`. Data can be added to the JSON object using the `ETController::json` method.

**Example:** <http://esotalk.org/forum/conversation/index.json/3>

**Forcing a JSON Response And Adding Data**

	$this->responseType = RESPONSE_TYPE_JSON;
	$this->json("member", $member);
	$this->render();

### Ajax

The "ajax" response type (`RESPONSE_TYPE_AJAX`) is a combination of the "view" and "json" response types. It is especially useful when making XMLHTTPRequest calls where the contents of a view needs to be fetched along with additional pieces of data.

Like the "json" response type, it renders data as a JSON object, **but it also renders the specified view into the "view" key of that JSON object.**

**Example:** <http://esotalk.org/forum/conversation/index.ajax/3>

**Rendering A JSON Object Containing A View And Other Data**

	if ($this->responseType === RESPONSE_TYPE_AJAX) {
		$this->json("error", "Your login details were incorrect");
		$this->render("user/login");
	}
	
This may return:

	{"error":"Your login details were incorrect","view":"<form>...</form>"}

<a name="assets"></a>
## Asset Management

The `ETController` class provides many methods that can be used to manipulate the response. Most notably, it is very easy to add links to assets such as CSS and JavaScript files to the `<head>` of the master view.

**Adding A CSS File**

	$this->addCSSFile("core/skin/base.css");
	
**Adding a JavaScript File**

	$this->addJSFile("core/js/search.js");
	
Even better, esoTalk intelligently minifies, aggregates, and caches all of the included CSS and JavaScript files. A second argument on both methods, if set to `true`, indicates a "global" asset — that is, an asset that will be included on all esoTalk pages. By distinguishing global assets, esoTalk can separate the minified CSS and JavaScript assets into two files: one is constant across all pages and so can be cached very effectively, and the other contains page-specific assets. For example, the cached CSS files when loading a conversation look like this:

	<link rel='stylesheet' href='/forum/cache/css/base,styles.css?1366507361'>
	<link rel='stylesheet' href='/forum/cache/css/bbcode,likes,fineuploader,attachments.css?1366424435'>
