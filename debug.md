# Troubleshooting

By default, esoTalk's "fatal error" messages aren't very helpful — and often, not even hitting your computer seems to help. Thankfully, you can flick on esoTalk's "debug" switch to find out what exactly is going wrong and get some direction on what to do next.

To enable these more verbose error messages, add the following line to `config/config.php`:

	$config["esoTalk.debug"] = true;
	
The [esoTalk Support Forum](http://esotalk.org/forum) is a good place to start to see whether anyone else has encountered the same error as you — and if not, you can start a conversation with the error details to get some help.

Some common problems and their solutions are listed below.

## *Page Not Found* Errors

If you keep getting *Page Not Found* errors whenever you try to navigate somewhere on your forum, double-check that [friendly URLs](/docs/configuration#friendly-urls) are set up correctly.

## "Could not instantiate mail function"

Make sure that your server is configured correctly to be able to send mail. If you don't know how to do that, try enabling and configuring the SMTP plugin included with esoTalk.