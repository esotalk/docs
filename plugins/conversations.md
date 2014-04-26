# Conversations

- [Labels](#labels)
- [Conversation Controls](#controls)
- [Post Manipulation](#posts)

<a name="labels"></a>
## Labels

Labels are icons which represent the current state of a conversation, either globally or for the current user. Default labels include sticky, locked, private, draft, and muted; you can add easily your own in plugins.

Each label has a name, a condition, and an icon.

* The name is used as the CSS class name (`.label-{name}`) when the label is rendered, as well as for translation (`T("label.{name}")`).
* The condition is an SQL snippet that is evaluated to determine if a label applies to a conversation. Often a label's presence will simply correlate to the value of a boolean column on the either the `conversation` table (aliased as `c`) or `member_conversation` table (`s`). For example, the sticky label's condition is `IF(c.sticky = 1, 1, 0)`. 
* The icon is the class name of a [Font Awesome icon](http://fortawesome.github.io/Font-Awesome/3.2.1/icons/) (e.g. `icon-pushpin`).

Labels must be registered statically to the `ETConversationModel` class. This can be done in the `init` method; the conversation model must be instantiated so that the class file is included.

**Registering A Label**

	public function init()
	{
		ET::define("label.bookmarked", "Bookmarked");

		ET::conversationModel(); // Load the conversation model so we can use the static class
		ETConversationModel::addLabel("bookmarked", "IF(s.bookmarked = 1, 1, 0)", "icon-bookmark");
	}

<a name="controls"></a>
## Conversation Controls

Items can be added to the conversation controls dropdown menu by handling the `conversationIndexDefault` event.

**Adding A Conversation Control**

	public function handler_conversationController_conversationIndexDefault($sender, $conversation, $controls, $replyForm, $replyControls)
	{
		$controls->add("bookmark", "<a href='#'><i class='icon-bookmark'></i> <span>Bookmark</span></a>");
	}

<a name="posts"></a>
## Post Manipulation

Each post in a conversation is rendered using the `core/views/conversation/post.php` template. This template uses a specially-constructed array of post template data with the following key-value pairs:

| Key | Type | Description |
| --- | --- | --- |
| `id` | String | The value of the `id` attribute on the outer post `<div>`. |
| `title` | String (HTML) | The title of the post (the username of the post author.) |
| `avatar` | String (HTML) | The post author's avatar. |
| `class` | Array | An array of CSS classes to apply to the outer post `<div>`. |
| `info` | Array | An array of HTML strings to output next to the post title. |
| `controls` | Array | An array of HTML strings to output in the post controls area (top-right). | 
| `body` | String (HTML) | The post content. |
| `footer` | Array | An array of HTML strings to output in the post footer, below the content. |
| `data` | Array | A key => value array of HTML data attributes on the outer post `<div>`. |

A post template array can be manipulated by handling the conversation controller's `formatPostForTemplate` event. This event passes three parameters: the referenced post template array, an array of raw post data retrieved from the query (see *Loading Data* below,) and an array of the conversation's details.

**Adding A Post Control**

	public function handler_conversationController_formatPostForTemplate($controller, &$template, $post, $conversation)
	{
		// Only add this control to posts that aren't deleted.
		if ($post["deleteMemberId"]) return;
		
		addToArray($template["controls"], "<a href='#' title='Example'><i class='icon-cog'></i></a>");
	}

### Loading Data

A common requirement for a plugin is to load additional data alongside posts in a conversation — for example: likes, attachments, or user profile information. See Schema Manipulation for information on how to store this data.

The obvious way to do this might be to perform an SQL query within a `formatPostForTemplate` event handler so the data can be added straight to the post template array. However, this is a big no-no! It will result in a query being run for every post that is displayed on the page — that's twenty-plus unnecessary queries on every page load. 

Instead, the necessary data can be loaded for all of the posts at once. This can be achieved using two events on the post model: `getPostsBefore` and `getPostsAfter`. 

If the data is simply an additional column on the posts or members table, then all that you need to do is specify that you want that column loaded in the [SQL query](/docs/database#queries). It will then become available in the raw post data array in the `formatPostsForTemplate` event. 

**Loading An Additional Column**

	public function handler_postModel_getPostsBefore($controller, $sql)
	{
		$sql->select("p.country", "location");
		
		// If the column lives in another table, you can join that here using the `from` method on the SQL query.
		// See ETPostModel::getWithSQL for a list of tables that are already joined.
	}
	
	public function handler_conversationController_formatPostForTemplate($controller, &$template, $post, $conversation)
	{
		// Only add this information to posts that aren't deleted.
		if ($post["deleteMemberId"]) return;

		addToArray($template["info"], "<span>".$post["location"]."</span>");
	}

In more complex cases, you might have multiple rows of data that need to be fetched for each post (e.g. multiple attachments per post.) This can be achieved in the `getPostsAfter` event by selecting all of the data for all of the posts in one query, and then sorting it manually. 

**Loading "Has Many" Data**

	public function handler_postModel_getPostsAfter($controller, &$posts)
	{
		if (!count($posts)) return;

		// Create an array of all the posts using post IDs for the keys.
		// Also give each post an empty "attachments" array.
		$postsById = array();
		foreach ($posts as &$post) {
			$postsById[$post["postId"]] = &$post;
			$post["attachments"] = array();
		}

		// Run a query to get all of the attachments for all of these posts at once.
		$result = ET::SQL()
			->select("postId")
			->select("a.*")
			->from("attachment a")
			->where("postId IN (:ids)")
			->bind(":ids", array_keys($postsById))
			->exec();

		// Loop through the results and add each attachment to the appropriate post.
		while ($attachment = $result->nextRow()) {
			$postsById[$attachment["postId"]]["attachments"][] = $attachment;
		}
	}

### Formatting

The `ETFormat` class is responsible for converting plain-text input to HTML output, as explained in [Formatting](/docs/formatting). This class triggers various events which can be used to perform additional formatting tasks on post content. These are `beforeFormat`, `format`, and `afterFormat`.

Generally speaking, most formatting should be done using the `format` event. The `beforeFormat` and `afterFormat` events are useful for performing pre- and post-formatting tasks, such as removing code blocks from the string and then inserting them back in so that they are unaffected by formatting.

**Performing Custom Formatting**

	public function handler_format_format($formatter)
	{
		$formatter->content = str_replace("foo", "bar", $formatter->content);
	}
