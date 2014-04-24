# Members

- [Preferences](#preferences)
- [Settings](#settings)
- [Profiles](#profiles)
- [Last Action Types](#last-action-types)

<a name="preferences"></a>
## Preferences

User preferences, such as language and notification flags, are stored in a serialized array in the `preferences` column of the `members` table. Keys should be namespaced with the name of your plugin followed by a period (`.`).

When a member is fetched by the `ETMemberModel`, their serialized preferences are automatically expanded into an array, accessible via the "preferences" key on the member array. For convenience, there are [methods on the `ETSession` object](/docs/session) which allow quick access to the current user's preferences.

The `ETSettingsController` handles the user settings interface and all of its default panes. Of particular interest is the *General* pane; this pane contains a [form](/docs/forms) that allows the user to update their preferences, and it can be extended to add more.

You can add your own fields and sections to the form (as described in [Forms](/docs/forms)) by handling the `initGeneral` event on the settings controller. The `savePreference` and `saveBoolPreference` methods on the settings controller can be used as processing callbacks; the name of the field will correspond to the preference key that is saved. You can also use your own processing callback, which should accept three arguments:

* The ETForm object
* The key of the field that is being processed
* A referenced array of preferences to save to the database

**Adding A Section & Field To The Settings Form**

	public function handler_settingsController_initGeneral($controller, $form)
	{
		$form->addSection("about", T("About"));

		$form->setValue("bio", ET::$session->preference("Profiles.bio"));
		$form->addField("about", "bio", array($this, "fieldBio"), array($this, "saveBio"));
	}
	
	public function fieldBio($form)
	{
		return $form->input("bio", "textarea");
	}
	
	public function saveBio($form, $key, &$preferences)
	{
		$preferences["Profiles.bio"] = $form->getValue($key);
	}

<a name="settings"></a>
## Settings

Each settings pane is its own action on the settings controller. In order to render a view inside of the profile "wrapper" view (i.e. showing the profile header and tabs), use the `renderProfile` method. 

**Adding A Settings Pane Action**

	public function action_settingsController_example($controller)
	{
		// The profile method sets up the settings page and returns the member's details.
		// The argument is the name of the currently-active pane.
		if (!($member = $controller->profile("activity"))) return;
		
		// ...
		
		$controller->renderProfile($this->view("example"));
	}

You'll probably also want to add a [tab](/docs/menus) for your settings pane. This can be achieved by listening for the `initProfile` event.

**Adding A Settings Tab**

	public function handler_settingsController_initProfile($controller, $panes)
	{
		$panes->add("example", "<a href='".URL("settings/example")."'>Example</a>");
	}

<a name="profiles"></a>
## Profiles

Similar to the settings page, each pane on a member profile page has its own action on the member controller. In order to render a view inside of the profile "wrapper" view (i.e. showing the profile header and tabs), use the `renderProfile` method. 

**Adding A Profile Pane Action**

	public function action_memberController_example($controller, $member = "")
	{
		// The profile method sets up the member profile page and returns the member's details. 
		// The first argument is the ID of the member being requested.
		// The second is the name of the currently-active pane.
		if (!($member = $controller->profile($member, "example"))) return;
		
		// ...
		
		$controller->renderProfile($this->view("example"));
	}

You'll probably also want to add a [tab](/docs/menus) for your profile pane. This can be achieved by listening for the `initProfile` event.

**Adding A Profile Tab**

	public function handler_memberController_initProfile($controller, &$member, $panes)
	{
		$panes->add("example", "<a href='".URL(memberURL($member["memberId"], $member["username"], "example"))."'>Example</a>");
	}
	
You can also add items to the "controls" dropdown menu as well as additional "action" links using this event. Controls are loosely defined as things that can be changed on the member (like their name or permissions,) while actions are things that can be done in relation to the member (like starting a private conversation.)

**Adding Profile Controls**

	public function handler_memberController_initProfile($controller, &$member, $panes, $controls, $actions)
	{
		$controls->add("exampleControl", "<a href='#'>Control</a>");
		$actions->add("exampleAction", "<a href='#'>Action</a>");
	}

<a name="last-action-types"></a>
## Last Action Types

