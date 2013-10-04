# URLs

- [Linking To Pages](#pages)
- [Linking To Resources](#resources)

<a name="pages"></a>
## Linking To Pages

To accommodate for different URL structures (i.e. [friendly URLs](/docs/configuration#friendly-urls) and mod_rewrite), all hyperlinks should be created using the `URL` function. Given a request path, this function will construct a relative or absolute URL which can be used to link to a page in esoTalk.

**Constructing A URL**

	$url = URL("conversation/delete/123"); // /forum/conversation/delete/123
	
**Constructing An Absolute URL**

	$url = URL("conversation/delete/123", true); // http://esotalk.org/forum/conversation/delete/123
	
You may construct request paths to common entities within esoTalk using the following functions. These functions ensure entity links are formatted consistently and are future-proof. However, the resulting request path still needs to be run through the `URL` function to properly link to a page.

**Constructing A Request Path To A Conversation**

	$url = conversationURL(123, "Conversation Title"); // 123-conversation-title

**Constructing A Request Path To A Member**

	$url = memberURL(123, "Toby"); // member/123-toby

**Constructing A Request Path To A Post**

	$url = postURL(123); // conversation/post/123

**Constructing A Request Path To A Search**

	$url = searchURL("search query", "general"); // conversations/general/?search=search+query
	
<a name="resources"></a>
## Linking To Resources

The `URL` function should only be used to link to pages. To link to resources (images or other files), use the `getResource` function.

**Constructing A URL To An Image**

	$url = getResource("core/skin/avatar.png");
	
> This function also takes into account the value of C("esoTalk.resourceURL") so that static resources may be served on a CDN.