## Facebook nodejs
A simple module for querying Facebook graph api and fql

## Usage example

	var express = require('express');
	var fbgraph = require('fbgraphapi');
	var app = express();
	var http = require('http');
	var server = http.createServer(app);
	
		
	app.use(express.bodyParser());
	app.use(express.cookieParser());
	app.use(express.session({secret : "SECRET"}));
	app.use(fbgraph.auth( {
			appId : "...",
			appSecret : "...",
			redirectUri : "http://0.0.0.0:3000/"
		}));
	
	
	app.get('/login', function(req, res) {
		console.log('Start login');
		fbgraph.redirectLoginForm(req, res);	
	});
	
	app.get('/', function(req, res) {
		if (!req.hasOwnProperty('facebook')) {
			console.log('You are not logged in');
			return res.redirect('/login');
		}
		/* See http://developers.facebook.com/docs/reference/api/ for more */
		req.facebook.graph('/me', function(err, me) {
		    console.log(me);
		});
		
		req.facebook.graph('/me?fields=id,name', function(err, me) {
		    console.log(me);
		});
		
		req.facebook.me(function(err, me) {
		    console.log(me);
		});
		
		req.facebook.me(function(err, me) {
		    console.log(me);
		}, 'id,name');
		
		// /me/likes
		req.facebook.my.likes(function(err, likes) {
		    console.log(likes);
		});
		
		/* Single fql query */
		req.facebook.fql('SELECT uid FROM user WHERE uid IN (SELECT uid2 FROM friend WHERE uid1=me())  AND is_app_user = 1', function(err, result) {
		    console.log(result);
		});
		
		/* Multiple fql queries */
		req.facebook.fql({
		    uids : 'SELECT uid FROM user WHERE uid IN (SELECT uid2 FROM friend WHERE uid1=me()) AND is_app_user = 1',
		    myapp : 'SELECT application_id, role FROM developer WHERE developer_id = me()'
		}, function(err, result) {
		    console.log(result);
		});
		res.end("Check console output");
	});
	
	server.listen(3000);

## Methods
### auth(config)
config is an object with those properties
* `appId` Facebook application Id
* `appSecret` Secret hash key generated by Facebook
* `redirectUri` The url to redirect to when user logged in.
* `scope` Permissions/scope that your application asks for, optional and default empty.

### authenticate(req, res, next)
This method is returned when calling auth() above. When loggin is successfull it will assign a Facebook instance to req (see example above).

### redirectLoginForm(req, res)
This method will redirect user to Facebook login form.

### destroySession(req, res, clearCookie)
This method is for logging out user or for any reason want to clear user's logged-in info

## Classes
### Facebook(accessToken)
* `accessToken` Valid access token from Facebook (oauth_token).
* `graph(path, callback)` Main method for making call to Facebook API. Path can contain only /me/likes or with params like /me/likes?limit=3
* `fql(query, callback)` Specific method for making fql query to Facebook API
* `me(callback, fields)` Specific method to get info of the logged user, same as /me when using graph-method. Fields is comma-separated for instance fields=id,name
* `getAppFriends(callback, fields)` Return friends that use your Facebook application. Fields is comma-separated.
* `my` Instance of My class
	
	var fb = new Facebook(accessToken); // if have a valid accessToken for instance from js login
	fb.me(function(err, me) {
		console.log(me);
	});
	

### My(facebook)
This class uses internally to create object my (property in Facebook). This object wrap the me-object. Each connection type is a method. This means that you can make a call like
	
	req.facebook.my.friends(function(err, friends) {
		console.log(friends)
	});
	
	// OR for checkins
	req.facebook.my.checkins(function(err, checkins) {
		console.log(checkins)
	});

Supported connection types are:

* `friends`
* `feed`
* `likes`
* `movies`
* `music`
* `books`
* `albums`
* `notes`
* `permissions`
* `photos`
* `videos`
* `events`
* `groups`
* `checkins`
* `locations`

