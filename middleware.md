# Middleware

App.net middleware provides a mechanism to insert your app into a user's status post (before and after). It primarily uses HTTP callbacks as a mechanism to change and augment behavior for App.net users.

Scenarios that App.net middleware makes possible. As a developer you can build services that:

* **Auto post to other networks**

   Immediately post to associated networks (Twitter, Google+, and Facebook.)

* **Post later (bufferapp style)**

   Store a post on your server, and post it for your user later. Possibly by pulling that information from the post itself. e.g. "Another amazing Breaking Bad episode #post_on_sun11pm"

* **Spelling corrections**
	
   Automatically fix common spelling mistakes. If unsure about something, buffer the post and and ask the user to clarify before continuing.

* **Easily turn an @reply into a DM**

   Watch for certain tags to change the post functionality entirely. Make this post actually send a DM: "@railsjedi grab drinks? #dm"


* **Talk to an ELIZA bot**

   Power a private eliza bot from your input text. e.g. "#eliza I'm having a bad day"

* **Tons more... sky's the limit**
	
   Allow users to power their entire life through conversational interfaces.

## Registering your App.net middleware

When register your application on App.net, fill in the "Middleware Spec" field with the url that points to your json spec. This spec describes how we display the middleware to new users.


### Writing a middleware spec

A middleware spec is a JSON api that describes your middleware. It must be hosted on your application's domain: http://myapp.com/appdotnet/middleware.json.

This spec will list all your available middlewares, and the URL callback for each one. All urls are relative to the base domain of the middleware spec json file.

Spec JSON Format:

	
	{
		"middleware": [
			{
				name: "Automatically Post to Twitter",
				description: "Automatically post to twitter. On first post, you will receive a DM reply that provides an authorization link so we can tie your App.net to your Twitter login",
				icon: "http://s3.amazonaws.com/myiconbucket/myicon.png",
				url: "/api/auto_post?service=twitter"
			},

			{
				name: "Buffer Posts for Later",
				description: "Will automatically store posts for posting at a later date by reading the #post_at hashtag.",
				icon: "http://s3.amazonaws.com/myiconbucket/myicon.png",
				url: "/api/buffer"
			}
		]
	}


## Install Middleware

To install middleware, users will just authenticate with your service via OAuth. The confirmation screen will include a list of your available middleware and a description of each one. They can then choose which ones to include.


### Middleware Opt in

When a user oauths with your App, they can opt out of the associated middleware. At any time, they can enable/disable your app's middleware via their Applications page on user settings.

[TODO] screenshot of OAuth page with middleware opt-in


### Middleware enabled/disable

After successfully authenticating your app, a user can 

[TODO] screenshot of App.net settings page with enabled/disable middleware



## Writing your Middleware

Once a user installs your middleware, every post he makes will generate an HTTP POST request to your endpoint.


### Request Body

Passed along in the request body will be a JSON string that contains the [Post object](https://github.com/appdotnet/api-spec/blob/master/objects.md#post) and the [User object](https://github.com/appdotnet/api-spec/blob/master/objects.md#user).


	{
		"user": {
			...			
		},
		"post": {
			...
		}
	}


You can use this information to handle the action.


### Response Body

App.net expects a JSON response that contains a `"post"` field with the same parameters as [Create a Post](https://github.com/railsjedi/api-spec/blob/master/resources/posts.md#create-a-post). 


e.g. Cross posting to several networks

	{
		"post": {	
		    "text": "App.net is the shiznik",
		    "annotations": {},
		    "links": {},
		}
		"message": "Your post has been posted to Google+, Twitter, and Facebook"
	}


Most of the time you just want to pass along the post object and then provide a result.

You can also return a short string `"message"` which will be displayed by the App.net client (in addition to your middleware's name). This alerts them to what has happened. You can also use it to authenticate users with additional services.

e.g. Cross posting to several networks

	{
		"post": {}
		"message": "To post to crosspost to Twitter, login here: http://myapp/auth/twitter?ref=123837"
	}


If no post exists in the response, then App.net will not create the status update. This is useful for utility middlewares (be sure that it's only in certain cases, not a catch all). It's also useful for cases where you want to save the post for a later time.


e.g. Storing a post in buffer for later.

	{
		"post": {},
		"message": "Your post has been saved and will be posted later at Sunday 11:00pm PST."
	}
	

### Handling Response Status Codes

App.net interprets any 2xx callback code as a successful post.

For any nonsuccessful HTTP status, App will display a message that your middleware is not functioning and give them a quick way to disable it and send the post anyways.

### Debugging Middleware

To test and debug your App.net middleware, just log into your developer site and you'll be able to see a log of the most recent middleware posts.

[TODO: screenshot here of App.net developer tools]



