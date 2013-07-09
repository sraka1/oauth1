BasicOAuth1
------------

PHP client library for working with OAuth1 endpoints. Forked from abraham/twitteroauth.

Flow Overview
=============

1. Build BasicOAuth object using client credentials.
2. Request temporary credentials from endpoint.
3. Build authorize/authenticate URL.
4. Redirect user to authorize/authenticate URL.
5. User authorizes access and returns from endpoint.
6. Rebuild BasicOAuth object with client credentials and temporary credentials.
7. Get token credentials from endpoint.
8. Rebuild BasicOAuth object with client credentials and token credentials.
9. Query endpoint API.
10. Profit!

Parameters
==========

There are a number of parameters you can modify after creating a BasicOAuth object. Some are required:

Required:

    $connection->accessTokenURL

    $connection->authenticateURL

    $connection->authorizeURL

    $connection->requestTokenURL

Optional:
Use this as an API endpoint, if you want to make API requests using this class as well (not just for auth). Should include trailing slash.

    $connection->host = "https://api.twitter.com/1.1/";

Custom useragent.

    $connection->useragent = 'Custom useragent string';

Verify endpoints SSL certificate.

    $connection->ssl_verifypeer = TRUE;

There are several more you can find in BasicOAuth.php.

Extended flow using example code
================================

To use BasicOAuth with the an OAuth endpoint you need *BasicOAuth.php*, *OAuth.php* and
client credentials. You can get client credentials by registering your application at the 
provider you will be connecting to. The simplest way to attain the files is via Composer.


1) First, build the basic object.
	use \OAuth1\BasicOauth;

    $connection = new BasicOAuth('abc890', '123xyz');

2) Using the built $connection object you will ask the endpoint for temporary credentials. The `oauth_callback` value is required.

    $tempCredentials = $connection->getRequestToken('http://asdf.com/oauthcallback');

3) Now that we have temporary credentials the user has to go to endpoint and authorize the app
to access and updates their data. You can also pass a second parameter of FALSE to not use request login (re-authenticate, however not all OAuth1 endpoints support this).

    $redirect_url = $connection->getAuthorizeURL($tempCredentials); // Must login
    $redirect_url = $connection->getAuthorizeURL($tempCredentials, FALSE); //Doesn't have to login

4) You will now have an endpoint URL that you must send the user to.

    $authUrl = https://api.twitter.com/oauth/authenticate?oauth_token=xyz123
    i.e.: header('Location: ' . $authUrl);

5) The user is now on the endpoint site's server and may have to login. Once authenticated with the server they will
will either have to click on allow/deny, or will be automatically redirected back to the callback.

6) Now that the user has returned to callback.php and allowed access we need to build a new
BasicOAuth object using the temporary credentials.

    $connection = new BasicOAuth('abc890', '123xyz', $_SESSION['oauth_token'],
    $_SESSION['oauth_token_secret']);

7) Now we ask the endpoint for long lasting token credentials. These are specific to the application
and user and will act like a password for making future requests. Normally the token credentials would
get saved in your database but for this example we are just using sessions.

    $tokenCredentials = $connection->getAccessToken($_REQUEST['oauth_verifier']);
    //save to DB

8) With the token credentials we can build a new BasicOAuth object.

    $connection = new BasicOAuth(CONSUMER_KEY, CONSUMER_SECRET, $tokenCredentials['oauth_token'],
    $tokenCredentials['oauth_token_secret']);

9) And finally we can make requests authenticated as the user. You can GET, POST, and DELETE API
methods. Directly copy the path from the API documentation. This consists of $connection->host with the provided url.

    $account = $connection->get('account/verify_credentials');
    $status = $connection->post('statuses/update', array('status' => 'Text of status here', 'in_reply_to_status_id' => 123456));
    $status = $connection->delete('statuses/destroy/12345');

10) Profit!

Tests
================================

The library has been confirmed to work with Twitter, BitBucket and Dropbox so far. Unit tests are on the way.