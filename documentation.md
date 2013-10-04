# README.md

esoTalk is a free, open-source forum software package built with PHP and MySQL. It is designed to be fast, simple, and highly extensible.

## Installation

## Upgrading

## Development

- Community
- Documentation

## Donate

# Contents

- Preface
	- Introduction
		- esoTalk Philosophy
		- Donate
	- Contributing
		- GitHub
		- Pull Requests
		- Code Standards
	- Roadmap
- Getting Started
	- Installation
	- Upgrading
	- Configuration
- Framework
	- Overview
		- ET
		- Configuration
		- Translation
		- Functions: R, C, T, URL functions
		- Factory
		- Request Lifecycle
	- Controllers & Views
		- Response Types
		- Rendering Views
		- Asset Management
	- Database
		- Queries
		- Results
		- Structure
	- Models
	- Session
	- Forms
	- Menus
	- Messages
	- Security
	- Useful Functions
	- JavaScript
		- jQuery
		- AJAX
		- Sheets
		- Popups
		- Messages
		- Tooltips
- Plugins
	- Packaging
	- Development
		- Events
		- Controller Methods
		- Factory
		- Setup
		- Settings & Configuration
	- Advanced
		- Activity & Notifications
		- Search
		- Conversation
		- Member & Settings
	- List of Events
- Skins
	- Packaging
	- Development
		- Styles, base skin
		- Settings & Configuration
		- Views
- Languages
	- Packaging
	- Translation

Assets
Menus
Sheets

JavaScript
- jQuery
- Sheets
- Messages
- Popups



# Plugins

A note about the future of esoTalk: 



Plugins

Introduction

Writing plugins for esoTalk couldn't be easier. There are numerous ways that esoTalk's code can be extended or overridden by a plugin, allowing you to add or change virtually any functionality you desire.

### Structure

A plugin is contained within a single folder, stored in the addons/plugins folder of an esoTalk installation. Each plugin should contain, at bare minimum, two important files:

#### package.json

This is a JSON document that contains basic information about your plugin package. It should look like this:

...

If your plugin relies on another plugin being enabled, or requires a minimum version of esoTalk to be installed, you can define dependencies in the package.json file. Simply add a 'dependencies' key containing a hash of dependency names for keys and minimum versions as the values. For example:

...

esoTalk will not allow a plugin to be enabled in the administration interface unless all dependencies are met.

#### plugin.php

This file is where all of your plugin's code will go. The file must consist of a class which extends the esoTalk\Plugin class. This class must be named Plugin_x, where x is the name of the folder in which the plugin is contained (and the 'id' attribute in package.json.)

For example, if I named my plugin 'HelloWorld', my plugin.php file might look like:

...

#### Icon

If your plugin folder contains a file named icon.png, esoTalk will automatically use it as your plugin's icon. It should be no larger than 16x16 pixels. Easy!

#### Resources

Your plugin may contain resources (CSS, JavaScript, or image files.) Resources must be placed in a folder called 'resources'. This will enable you to easily fetch the path to a resource for inclusion in the page using the ..... method.

#### Views



#### Other files

Your plugin may contain other files, such as views, controllers, or other libraries. You can put these wherever you want! Feel free to create additional folders within your plugin to keep your files nice and organized. You will ultimately be responsible for the inclusion of these files.

### Events

The first of the three main ways to extend esoTalk is with 'events'. Throughout esoTalk's core code, there are lots of event 'calls'--in other words, opportunities for you to run your own code. 

To use events, simply:

1. Find the event where you would like your code to run. For a full list of events, see the Event List. Or, you can just look through the source code. 

2. Attach your event handler to this event using the Event::attach method.

For example, to attach a listener to the 'save_conversation' event, you would put the following in your plugin's __construct method:

...

#### common events

There are a few events that might come in handy quite often. 

The compose_layout event is triggered when the page's layout is being set up. This is the best place to add any assets (CSS or JavaScript files) to the page:

...

> esoTalk intelligently aggregates JavaScript and CSS files before serving them to the client. You can read more about how to take advantage of this at Assets. 

### 

Assets
Menus
Sheets

JavaScript
- jQuery
- Sheets
- Messages
- Popups



Routes, Controllers, & Views