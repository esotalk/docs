# Session

The ETSession object provides functions to handle all operations relating to the current user session. For starters, the session object provides a wrapper to store and retrieve values from the session.

**Storing A Value In The Session**

	ET::$session->store("key", "value");
	
**Retrieving A Value From The Session**

	$value = ET::$session->get("key", "default");

The session object is also the point of access for getting information about the currently logged-in user, and performing operations and checks on that user.

**Determining If A User Is Logged In**

	if (ET::$session->userId) {
		// Logged in. User accessible via ET::$session->user
	} else {
		// Not logged in
	}

**Getting A User's Preference**

	$value = ET::$session->preference("key", "default");
	
**Setting The User's Preferences**

	ET::$session->setPreferences(array("key" => "value"));
	
**Determining If The User Is An Administrator**

	$admin = ET::$session->isAdmin();