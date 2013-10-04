# Localization

Whenever you output language to the page in esoTalk code, you should run it through the localization function, `T()`. This allows the application to be consistently translated into different languages. Default language definitions are stored in `addons/languages/English/definitions.php`.

The `T` function accepts two arguments: (1) the string to translate, and (2) a default value if the translation has not been defined in the current language. If (2) is omitted, the value of (1) is used as the default.

**Translating A String**

	$translation = T("Click on a member's name to remove them.");

**Translating A String Or Returning A Default Translation**

	$message = T("message.conversationNotFound", "The conversation was not found.");

> The `T()` function is an alias for `ET::translate()`.

<!-- -->
> Localization can also be performed in JavaScript using an identical `T()` function. For more information on how this works, see [JavaScript](/docs/javascript#data).

Default values can also be defined using the `ET::define` method. They will only be used as a fallback if a definition does not exist in the current language, unless a third parameter is set to `true`.

**Defining A Default Translation**

	ET::define("message.conversationNotFound", "The conversation was not found.");