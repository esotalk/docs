# Search

- [Introduction](#introduction)
- [Gambits](#gambits)

<a name="introduction"></a>
## Introduction

One of esoTalk's most powerful features is its search system. The search system makes use of a concept called "gambits". A gambit starts with a hash (`#`) and represents an independent piece of search criteria — for example, the `#sticky` gambit means "only match conversations that are sticky." The search string can contain multiple gambits, separated by plus characters (`+`).

The `ETSearchModel` class is responsible for deconstructing a search string into individual gambits, and then performing a search for conversations based on those gambits' criteria. Searches are performed by the following steps:

1. The `getConversationIDs` method is called, passing a list of channel IDs to show results from and a search string.
2. The search string is parsed and split into terms. When a term is matched to a gambit, the gambit's callback function is called.
3. Gambit callback functions add "conversation ID filters" to narrow the range of conversations being searched, or may alter other parts of the search query.
4. Using the applied ID filters, a final list of conversation IDs is retrieved and returned.
5. The `getResults` method can then be called with this list, and full details are retireved for each of the conversations.

Confusing? Let's break it down using an example search string that means "find sticky conversations that I've contributed to":

> \#sticky + #contributor:Toby

The search model's `getConversationIDs` method will be called by the conversations controller. The search string will be parsed and split into two gambits: `sticky` and `contributor:Toby`. (The preceeding hash is discarded.) It will match these to two default gambits that are registered with the search model which have the callback functions `gambitSticky` and `gambitContributor`.

In `gambitSticky`, we simply add a WHERE condition to the search model's SQL query object.

	public static function gambitSticky(&$search, $term, $negate)
	{
		$search->sql->where("sticky=".($negate ? "0" : "1"));
	}

`gambitContributor` is a bit more complex. First, we parse the `contributor:Toby` term and find the ID of the member with the name "Toby". Then, we need to add an "ID filter". An ID filter is an independent SQL query which SELECTs a list of conversation IDs that match a certain criteria. In this case, we want our ID filter query to select the conversation IDs of all posts by this member.

	public static function gambitContributor(&$search, $term, $negate)
	{
		// Get the name of the member.
		$term = trim(substr($term, strlen(T("gambit.contributor:"))));

		// If the user is referring to themselves, then we already have their member ID.
		if ($term == T("gambit.myself")) $q = (int)ET::$session->userId;

		// Otherwise, make a query to find the member ID of the specified member name.
		else $q = ET::SQL()->select("memberId")->from("member")->where("username=:username")->bind(":username", $term)->get();

		// Apply the ID filter.
		$sql = ET::SQL()
			->select("DISTINCT conversationId")
			->from("post")
			->where("memberId IN ($q)");
		$search->addIDFilter($sql, $negate);
	}

Once all of the search terms have been parsed and their gambit callback functions have been executed, the search model will progressively run all of the ID filter queries to narrow down a list of matching conversation IDs. Finally, the results will be retrieved from search model's SQL query object (which is where the `sticky` gambit was applied.)

<a name="gambits"></a>
## Gambits

Gambits can be registered using the search model's static `addGambit` function. This function accepts two arguments:

- A condition to run through `eval()` to determine a match. `$term` represents the search term, in lowercase, in the `eval()` context. The condition should return a boolean value: `true` means a match, `false` means no match. Example: `return $term == "sticky";`
- The function to call if the gambit is matched. The function will be called with the parameters `callback($search, $term, $negate)`.

Within the callback, the SQL query object on the search model (`$search->sql`) may be manipulated, or ID filters may be added using the `$search->addIDFilter` method. For more information, see the [API Reference](/api/class-ETSearchModel.html).

**Registering A Gambit**

	public function init()
	{
		ET::define("gambit.bookmarked", "bookmarked");
	}

	// Register the "bookmarked" gambit with the search model.
	public function handler_conversationsController_init($sender)
	{
		ET::searchModel(); // Load the search model so we can use the static class
		ETSearchModel::addGambit('return $term == strtolower(T("gambit.bookmarked"));', array($this, "gambitBookmarked"));
	}

	public static function gambitBookmarked(&$search, $term, $negate)
	{
		if (!ET::$session->user or $negate) return;

		$sql = ET::SQL()
			->select("DISTINCT conversationId")
			->from("member_conversation")
			->where("type='member'")
			->where("id=:memberId")
			->where("bookmarked=1")
			->bind(":memberId", ET::$session->userId);

		$search->addIDFilter($sql);
	}

### Aliases

Aliases can be created to simplify the use of complex gambits. For example, the `dead` gambit is actually just an alias for `active >30 day`.

**Registering An Alias**

	public function handler_conversationsController_init($sender)
	{
		ET::searchModel(); // Load the search model so we can use the static class
		ETSearchModel::addAlias("popular", "has >50 replies");
	}

### Gambits Menu

To make gambits usable, they should be added to the gambits menu that appears when the search field gains focus. The gambit menu is constructed by the conversations controller, not the search model, as it is part of the user interface. It can be altered by handling the `constructGambitsMenu` event.

The gambits menu is structured as a multi-dimensional array rather than a traditional [menu object](/docs/menus); in this way, gambits can be grouped into categories according to the type of criteria they represent. There are a few gambit categories by default: `main`, `time`, `member`, `replies`, and `misc`. You can add gambits to any one of these categories, or create your own.

Each key-value pair within a category represents a gambit. The key is the gambit; the value is an array containing a CSS class and an icon to display the gambit with.

**Adding A Gambit To The Menu**

	public function handler_conversationsController_constructGambitsMenu($controller, &$gambits)
	{
		addToArrayString($gambits["main"], T("gambit.bookmarked"), array("gambit-bookmarked", "icon-bookmark"));
	}
