# Activity

- [Introduction](#introduction)
- [Activity Types](#types)
- [Creating Activity](#creation)

<a name="introduction"></a>
## Introduction

In esoTalk, "activity" is something that happens on the forum that is associated with a particular member. For example:

- A member joining the forum
- A member being mentioned in a post
- A member being added to a private conversation
- A member's group being changed

Each of these pieces of activity has its own activity **type**. Each activity type is associated with one or more **projections**, or places where it will be projected to. There are three possible projections:

- **activity** — activity of this type should be shown on the member's profile
- **notification** — activity of this type should be shown as a notification to the member
- **email** — activity of this type should be sent out as an email to the member

As an example, every time a member is mentioned in a new reply, a new piece of activity of type `mention` is created. The `mention` activity type projects to the notification and email projections. So, the member who was mentioned sees the activity in their notifications, and receives an informative email in their inbox.

<a name="types"></a>
## Activity Types

Plugins can add their own activity types, and thus custom types of user profile activity, notifications, and emails. Activity types can be registered using the `ETActivityModel::addType` method. The first argument is the name of the activity type; the second is an array with projections for keys and callback functions for values.

**Registering An Activity Type**

	public function init()
	{
		ET::activityModel(); // Load the activity model so we can use the static class
		ETActivityModel::addType("like", array(
			"notification" => array($this, "likeNotification"),
			"email"        => array($this, "likeEmail")
		));
	}

A projection's callback function is called every time an activity item needs to be rendered in that projection. It receives two arguments:

1. An array of activity details. This contains information about the member who originally performed the activity (`fromMemberId`, `fromMemberName`), as well as an array of additional activity `data`. (See [Creating Activity](#creation).)
2. An array of details about the member who is receiving the activity (e.g. the member who joined the forum, the member who was mentioned.)

The callback function for each projection should return HTML to be rendered. The exact format of the return value depends on the projection:

| Projection | Callback Return Value |
| --- | --- |
| activity | array( title, body ) | 
| notification | array( body, URL ) | 
| email | array( subject, body ) |

**Defining An Activity Notification Callback**

	public function likeNotification($activity, $member)
	{
		return array(
			sprintf(T("%s liked your post in %s."), name($activity["fromMemberName"]), "<strong>".sanitizeHTML($item["data"]["title"])."</strong>"), // Notification body
			URL(postURL($item["data"]["postId"])) // Notification link
		);
	}

<a name="creation"></a>
## Creating Activity

New activity items can be created using the `create` methods on the activity model. This method accepts various parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `$type` | String | The name of the type of activity to create. |
| `$member` | Array | An array of details for the member to create the activity for. |
| `$fromMember` | Array | An array of details for the member that the activity is from. This can be null if the activity is from no one in particular. |
| `$data` | Array | An array of custom data that can be used by the projection callback functions. |
| `$emailData` | Array | An array of custom data that can be used only by the EMAIL projection callback function. (i.e. it is not stored in the database.) |

**Creating An Activity Item**

	ET::activityModel()->create("like", $member, ET::$session->user, array(
		"postId" => $postId,
		"title"  => $title
	));
