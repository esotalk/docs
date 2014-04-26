# Forms

- [Basic Usage](#usage)
- [Generating Elements](#elements)
- [Processing Input](#input)
- [Form Construction](#structure) 

<a name="usage"></a>
## Basic Usage

The basic purpose of the **[ETForm](/api/class-ETForm.html)** class is to provide convenient methods for processing form data and outputting form HTML. An ETForm instance represents a complete form; properties can be set on it in the *controller*, and then methods can be used to output the form elements in the *view*.

**Creating A New Form Object**

	$form = ETFactory::make("form");
	$form->action = URL("conversation/save");
	
**Opening & Closing A Form**

	echo $form->open(); // <form action='conversation/save' method='post'>
	echo $form->close(); // </form>
	
Hidden inputs may be added to the form before it is opened. They will be outputted following the opening `<form>` tag.

**Adding A Hidden Input**

	$form->addHidden("memberId", 1);

<a name="elements"></a>
## Generating Elements

The ETForm class has an [abundance of methods](/api/class-ETForm.html) to output all different types of form elements.

**Generating Basic Input Elements**

	echo $form->input("name", "text");
	echo $form->input("password", "password");
	echo $form->input("description", "textarea", array("cols" => 20, "rows" => 5));
	
**Generating A Drop-Down List**

	echo $form->select("size", array(
		"S" => "Small",
		"M" => "Medium",
		"L" => "Large"
	));
	
**Generating Checkboxes & Radio Buttons**

	echo $form->checkbox("agree");
	echo $form->radio("gender", "Male");
	echo $form->radio("gender", "Female");

**Generating A Submit Button**

	echo $form->button("save", "Submit Form");
	
**Generating A Standard "Save Changes" Button**

	echo $form->saveButton();
	
**Generating A Standard "Cancel" Button**

	echo $form->cancelButton();

Note that none of these methods allow you to specify the **default value** of the form element. Instead, this is done on the ETForm object itself, and then values are automatically inserted when generating the form HTML.

**Setting A Default Form Value**

	$form->setValue("name", "Simon");

<a name="input"></a>
## Processing Input

The ETForm class provides a method to check if a form has been submitted. There is automatic CSRF protection so you don't have to worry about it.

**Checking If A Form Has Been Submitted**

	if ($form->validPostBack()) {
		// The form was submitted
	}
	
**Checking If A Certain Button Was Submitted**

	if ($form->validPostBack("save")) {
		// The form was submitted by clicking the "save" button
	}
	
Once you know that the form was submitted safely, you will probably want to retireve the submitted values and process them.

**Getting All Submitted Values**

	$values = $form->getValues();
	
**Getting The Value Of A Single Field**

	$name = $form->getValue("name", "Frank McDefault");

The following example demonstrates how a Form object may be used across a controller and view.

**Basic Form Example**

Controller:

	$form = ETFactory::make("form");
	$form->setValue("name", "Simon");
	
	if ($form->validPostBack("save")) {
		$name = $form->getValue("name");
		// Save the name somewhere...
	}
	
	$this->data("form", $form);
	
View:

	$form = $data["form"];
	echo $form->open(),
		$form->input("name", "text"),
		$form->button("save", "Save name"),
		$form->close();
	
<a name="structure"></a>
## Form Construction

The ETForm class can also be used to dynamically construct forms containing multiple sections and fields. This is particularly useful in allowing plugins to easily extend forms.

A form can contain multiple **sections**, and a section can contain multiple **fields**. A final argument can be specified to insert a section or a field at a certain position, in the same way as discussed in the [Menu](/docs/menus) documentation.

**Adding A Section**

	$form->addSection("account", "Account Details");
	$form->addSection("notifications", "Notification Preferences", array("after" => "account"));
	
**Removing A Section**

	$form->removeSection("account");

When adding a field, the following must be provided:

- The **ID of the section** to put this field in
- The **ID of the field** to add
- A callback function that will **render the field**
- A callback function that will **process the field** (optional)
- A **position** at which to insert the field (optional)

**Adding A Field To A Section**
	
	$form->addField("account", "name", "renderNameField", "processNameField");
	
	function renderNameField($form)
	{
		return $form->input("name", "text");
	}
	
	function processNameField($form, $key)
	{
		$name = $form->getValue($key);
	}
	
Once the form's sections and fields have been constructed, the form object may be passed to the view and this structure can then be outputted. This is achieved by retrieving and looping through the sections using the `getSections` method, and then retrieving and outputting each section's fields using the `getFieldsInSection` method.

**Outputting A Structured Form**

	<?php echo $form->open(); ?>
	<?php foreach ($form->getSections() as $section => $title): ?>

		<h1><?php echo $title; ?></h1>
		<?php foreach ($form->getFieldsInSection($section) as $field): ?>
			<?php echo $field; ?>
		<?php endforeach; ?>

	<?php endforeach; ?>

	<?php echo $form->saveButton(); ?>
	<?php echo $form->close(); ?>
	
Finally, when it is submitted, the form's fields must be processed in the controller. Each field should have been defined with a processing callback function; these callbacks can be run using the `runFieldCallbacks` method.

**Running Field Processing Callbacks**

	if ($form->validPostBack("save")) {
		$form->runFieldCallbacks();
	}

Often, rather than saving data (i.e. writing to the database) *each* time a processing callback function is called, it may be more efficient to collect data to then write to the database after all fields have been processed. Simply pass a *collector* variable to the `runFieldCallbacks` method and then manipulate it inside of the processing callback functions.

**Collecting Data From Field Processing Callbacks**

	$form->addField("account", "name", "renderNameField", "processNameField");

	function processNameField($form, $key, &$data)
	{
		$data["name"] = $form->getValue($key);
	}

 	 if ($form->validPostBack("save")) {
 	 	$data = array();
		$form->runFieldCallbacks($data);
		
		// $data["name"] will be set now. Write $data to the database...
	}
