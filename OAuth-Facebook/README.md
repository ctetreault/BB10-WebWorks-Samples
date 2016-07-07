# Facebook OAuth 2.0 Sample

This sample demonstrates how to integrate Facebook and the OAuth 2.0 protocol into a BlackBerry WebWorks application for BlackBerry 10.

Facebook OAuth Sample is the name of a proof-of-concept WebWorks application that allows a user to authenticate with Facebook, post a new status to, and view their News Feed.

**Applies To**

* [BlackBerry 10 WebWorks SDK](https://developer.blackberry.com/html5/download/sdk) 
* [Apache Cordova for BlackBerry 10](https://github.com/blackberry/cordova-blackberry/tree/master/blackberry10) 

**Author(s)** 

* [Chad Tetreault](http://bit.ly/chadli123)

**Dependencies**

1. [bbUI.js](https://github.com/blackberry/bbUI.js) is [licensed](https://github.com/blackberry/bbUI.js/blob/master/LICENSE) under the Apache 2.0 license.
2. [jQuery](http://code.jquery.com/jquery-1.7.2.js) is [dual licensed](http://jquery.org/license/) under the MIT or GPL Version 2 licenses.

**Icons**

* The [Liz Myers](http://www.myersdesign.com) Icon set and are [licensed](http://creativecommons.org/licenses/by/3.0/) under the CC-BY-3.0 license.

**Contributing**

* To contribute code to this repository you must be [signed up as an official contributor](http://blackberry.github.com/howToContribute.html).

## Screenshots ##

![image](https://raw.github.com/blackberry/BB10-WebWorks-Samples/WebWorks-2.0/OAuth-Facebook/www/_screenshots/oauth-facebook.png)

## Required Plugins ##

####The following Cordova Plugins are required for this sample:####

	com.blackberry.app
	com.blackberry.ui.toast

## Initial Facebook Setup

1. Create an application on Facebook (http://developers.facebook.com/) 
2. Under "App Domains" enter a domain where you'll be redirecting the user after they authenticate
3. Under "Select how your app integrates with Facebook", choose "Website with Facebook Login"
4. Enter the domain again, where the users will be redirect to after authentication
3. When your app is created, copy down your App ID, and App Secret

**For STEP 4. Make sure you enter a trailing slash after the domain name, for example: http://mycallbackurl.com/**

## WebWorks App Setup

Open app.js and edit the following object

	facebookOptions = {
    	clientId: '',
	    clientSecret: '',
	    redirectUri: ''
	};

## Config.xml 
As of BlackBerry WebWorks 1.0.2.9 SDK, all domains you plan on making Ajax/XHR requests to must be whitelisted in your app's config.xml.

**Note: While we need to disable web security in order to read the location of our childWindow object, it's recommended that you don't do this in your apps unless absolutely necessary.**

	<access uri="*" subdomains="true" />
    <preference name="websecurity" value="disable" />

## Security Considerations

Your App Secret key, is intended to stay SECRET.  For demonstration purposes we coded the App ID and App Secret right in the JavaScript source.  This is not best practice, and is not recommended.  You don’t want anybody to get access to your keys.

One way to securely pass your App Secret to your application is to host it on a server, then use SSL and do a POST to obtain your key. It can then be used to obtain an Access Token from the service (in this case, Facebook).

## End Points

This sample app shows how to connect your application with a few different Facebook end-points. The source code is fairly well commented, and will show the entire flow. The syntax is as follows:

### User is prompted to allow your application access to their info.  Facebook returns an access "code".

Note the ***scope*** parameter in the url below.  This is how your application requests all the permissions it will need to interact with the user’s Facebook profile.  In this sample we are requesting permission to ***publish*** and ***read*** the users stream.

See Facebook’s [Permissions Reference]( https://developers.facebook.com/docs/authentication/permissions/) for a list of additional permissions your app may require.

	var url = 'https://www.facebook.com/dialog/oauth?client_id=' + facebookOptions.clientId + '&redirect_uri=' + facebookOptions.redirectUri + '&scope=publish_stream,read_stream';
	
	childWindow = window.open(url, '_blank');

### Exchange the access code, for an access token (we use this token for authenticated requests)
		

	var url = 'https://graph.facebook.com/oauth/access_token?client_id=' + facebookOptions.clientId + '&redirect_uri=' + facebookOptions.redirectUri + '&client_secret=' + facebookOptions.clientSecret + '&code=' + authCode;

	$.ajax({
	   type: 'GET',
	   url: url,
	
	   success: function(data) {
	      var response = data;

    	  // parse 'access_token' from the response
	      var response = response.split('&');
	      var theAccessToken = response[0].split('=');
	      window.accessToken = theAccessToken[1];

    	  // get authenticated users' info/name
	      getUserInfo();
	   },

	   error: function(data) {
	      alert('Error getting access_token: ' + data.responseText);
	      return false;
	   }
	});

### Get authenticated user's info (we use this to display their real name)

var url = 'https://graph.facebook.com/me?access_token=' + accessToken;

	$.ajax({
	   type: 'GET',
	   url: url,
	   dataType: 'json',
	
	   success: function(data) {
     	  bb.pushScreen('connected.html', 'connected');
	      window.userName = data.name;
	   },

	   error: function(data) {
	   	   alert('Error getting users info: ' + data.responseText);
		   return false;
	   }
	});

### Posting an update to the news feed


	var randomNum = Math.round(Math.random() * 999 + 1);
	var status = 'Test (' + randomNum + ') of the Facebook OAuth sample for BlackBerry 10 by @chadtatro! (http://twitter.com/chadtatro) http://bit.ly/106Blwv';
	var url = 'https://graph.facebook.com/me/feed?message=' + status + '&access_token=' + accessToken;

	$.ajax({
	   type: 'POST',
	   url: url,
	   dataType: 'json',
	
	   success: function(data) {
     	  getFeed();
	   },

	   error: function(data) {
     	  alert('Error updating status: ' + data.responseText);	      return false;
	   }
	});


### Fetching the user's news feed

	$('#content p').remove();
	var url = 'https://graph.facebook.com/me/feed?access_token=' + accessToken;

	$.ajax({
	   type: 'GET',
	   url: url,
	   dataType: 'json',
	   
	   success: function(data) {
     	  var feed = data.data;

	      // show the last 10 items from the users news feed
	      // note: there are several objects that could be posted in a news feed. for simplicity
	      // we're only showing objects with a 'story' attribute
	
	      for(var i = 0; $('#content p').size() < 10; i++) {
	         if(typeof feed[i].message !== 'undefined') {
             $('#content').append('<p>' + feed[i].message + '</p>');
          }
       },
	
		error: function(data) {
    		alert('Error loading news feed: ' + data.responseText);
	        return false;
	    }
	});

## How to Build

1. Clone this repo to your local machine.

2. Ensure the [BlackBerry 10 WebWorks SDK 2.0](https://developer.blackberry.com/html5/download/sdk) is correctly installed.

3. Open a command prompt (windows) or terminal (mac) and run the following command:

	```
	webworks create <your source folder>\OAuth-Facebook
	```

4. **Replace** the default OAuth-Facebook\www folder with the \www folder from **this** project

5. **Replace** the default OAuth-Facebook\config.xml with the config.xml from **this** project

6. From the command prompt (Windows) or terminal (mac), navigate to the OAuth-Facebook folder

	```
	cd <your source folder>\OAuth-Facebook
	```

7. Run the following commands to configure plugins used by **this app**

	```
	webworks plugin add com.blackberry.app
	webworks plugin add com.blackberry.ui.toast
	```

8. Run the following command to build and deploy the app to a device connected via USB

	```
	webworks run
	```
OAuth-Facebook


## More Info

* [BlackBerry HTML5 WebWorks](https://bdsc.webapps.blackberry.com/html5/) - Downloads, Getting Started guides, samples, code signing keys.
* [BlackBerry WebWorks Development Guides](https://bdsc.webapps.blackberry.com/html5/documentation)
* [BlackBerry WebWorks Community Forums](http://supportforums.blackberry.com/t5/Web-and-WebWorks-Development/bd-p/browser_dev)
* [BlackBerry Open Source WebWorks Contributions Forums](http://supportforums.blackberry.com/t5/BlackBerry-WebWorks/bd-p/ww_con)

## Contributing Changes

Please see the [README](https://github.com/blackberry/BB10-WebWorks-Samples) of the BB10-WebWorks-Samples repository for instructions on how to add new Samples or make modifications to existing Samples.

## Bug Reporting and Feature Requests

If you find a bug in a Sample, or have an enhancement request, simply file an [Issue](https://github.com/blackberry/BB10-WebWorks-Samples/issues) for the Sample.

## Disclaimer

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
