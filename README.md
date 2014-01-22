UltimateOAuth
=============
*A __highly advanced__ Twitter library in PHP.*

[日本語](https://github.com/Certainist/UltimateOAuth/blob/master/README-Japanese.md)

@Version: 5.3.4  
@Author : CertaiN  
@License: BSD 2-Clause  
@GitHub : http://github.com/certainist  

## \[Features\]

Very similar to [twitteroauth](https://github.com/abraham/twitteroauth).  
On the other hand, there are some original functions.

| Item | twitteroauth | UltimateOAuth |
| :----: | :-----------: | :------------: |
| Supported PHP Version | 5.2.0 or Newer | 5.2.0 or Newer |
| Dependence | cURL, OAuth.php | None(It works only on this file.) |
| Automatically Fix Weird Responses | No | Yes |
| Uploading Images | No | Yes |
| Synchronize Requests | Yes | Yes |
| Asynchronize Requests | No | Yes |
| Para-xAuth Authorization | No | Yes |
| Generating Accounts | No | Yes |
| Avoid API Limits | No | Yes |

## \[Supported Classes and Methods\]

### UltimateOAuth

```php
$uo = new UltimateOAuth(
    $consumer_key = "", $consumer_secret = "", $access_token = "", $access_token_secret = ""
);

$uo->get                   ($endpoint,                  $params = array()                       );
$uo->post                  ($endpoint,                  $params = array(), $wait_response = true);
$uo->postMultipart         ($endpoint,                  $params = array(), $wait_response = true);
$uo->OAuthRequest          ($endpoint, $method = "GET", $params = array(), $wait_response = true);
$uo->OAuthRequestMultipart ($endpoint,                  $params = array(), $wait_response = true);

$uo->directGetToken ($username, $password);

$uo->getAuthorizeURL    ($force_login = false);
$uo->getAuthenticateURL ($force_login = false);
```

### UltimateOAuthMulti

```php
$uom = new UltimateOAuthMulti;

$uom->enqueue ($uo, $method, $arg1, $arg2, $arg3, ...);
$uom->execute ($wait_processes = true, $use_cwd = false);
```

### UltimateOAuthRotate

**UltimateOAuthRotate** supports  
`get()`, `post()`, `postMultipart()`, `OAuthRequest()`, `OAuthRequestMultipart()`  
of **UltimateOAuth**, by using **__call()**.

```php
$uor = new UltimateOAuthRotate;

$uor->__call       ($name, $arguments);

$uor->register     ($name, $consumer_key, $consumer_secret);
$uor->login        ($username, $password, $return_array = false, $successively = false);
$uor->setCurrent   ($name);
$uor->getInstance  ($name);
$uor->getInstances ($type);
```

------------------------------------------------------------------

0. Multiple Request Settings
-----------------------

Edit constants of **UltimateOAuthConfig**.

- For the environment that proc_open() is disabled for security reasons
  
  > - `USE_PROC_OPEN`: **FALSE**
  > - `FULL_URL_TO_THIS_FILE`: Not an absolute path, but an **absolute URL**.
  
- Otherwise
  
  > - `USE_PROC_OPEN`: **TRUE** (default)
  
  
------------------------------------------------------------------
  
1. OAuth Authentication
-----------------------

To authenticate or authorize, take the following steps.

**prepare.php**

```php
<?php

// Load this library
require_once('UltimateOAuth.php');

// Start session
session_start();

// Create a new UltimateOAuth instance
$uo = new UltimateOAuth('YOUR_CONSUMER_KEY', 'YOUR_CONSUMER_SECRET');

// Store as a session var
$_SESSION['uo'] = $uo;

// Get request_token
$res = $uo->post('oauth/request_token');
if (isset($res->errors)) {
    die(sprintf('Error[%d]: %s',
        $res->errors[0]->code,
        $res->errors[0]->message
    ));
}

// If you want to AUTHENTICATE,
$url = $uo->getAuthenticateURL();
// If you want to AUTHORIZE,
// $url = $uo->getAuthorizeURL();

// Jump to Twitter
header('Location: ' . $url);
exit();
```

After user has logined, the page will be jumped back to **Callback URL** with **oauth_verifier**.  
You have to configure this parameter in [Twitter Developers](https://dev.twitter.com/apps).

**callback.php**

```php
<?php

// Load this library
require_once('UltimateOAuth.php');

// Start session
session_start();

// Check session timeout
if (!isset($_SESSION['uo'])) {
    die('Error[-1]: Session timeout.');
}

// Restore from session
$uo = $_SESSION['uo'];

// Check oauth_verifier
if (!isset($_GET['oauth_verifier'])) {
    die('Error[-1]: No oauth_verifier');
}

// Get access_token
$res = $uo->post('oauth/access_token', array(
    'oauth_verifier' => $_GET['oauth_verifier']
));
if (isset($res->errors)) {
    die(sprintf('Error[%d]: %s',
        $res->errors[0]->code,
        $res->errors[0]->message
    ));
}

// Jump to your main page
header('Location: main.php');
exit();
```


**main.php**

```php
<?php

// Load this library
require_once('UltimateOAuth.php');

// Start session
session_start();

// Check session timeout
if (!isset($_SESSION['uo'])) {
    die('Error[-1]: Session timeout.');
}

// Restore from session
$uo = $_SESSION['uo'];

// Let's tweet!
$uo->post('statuses/update', 'status=TWEEEEEEEEEEEEEEETING!!!!');
```

**Note1:**  
If you already have `access_token` and `access_token_secret`,  
you can build up by just doing like this:

```php
<?php

require_once('UltimateOAuth.php');

$uo = new UltimateOAuth($consumer_key, $consumer_secret, $access_token, $access_token_secreet);
```

**Note2:**  
For saving authenticated UltimateOAuth object as string, use `serialize()` and `unserialize()`.

------------------------------------------------------------------

2-1. Class Detail - UltimateOAuth
----------------------------------

### UltimateOAuth::__construct()

Create a new UltimateOAuth instance.

```php
$uo = new UltimateOAuth;
$uo = new UltimateOAuth($consumer_key, $consumer_secret);
$uo = new UltimateOAuth($consumer_key, $consumer_secret, $access_token, $access_token_secret);
```

#### Arguments

- *(string)* *__\[$consumer\_key\]__*  
  A random official one is used when omitted.
- *(string)* *__\[$consumer\_secret\]__*  
  A random official one is used when omitted.
- *(string)* *__\[$access\_token\]__*  
  Necessary if you don't authenticate/authorize later.
- *(string)* *__\[$access\_token\]__*  
  Necessary if you don't authenticate/authorize later.
  
=========================================

### UltimateOAuth::OAuthRequest()

Used for requests mainly.

```php
$uo->OAuthRequest($endpoint, $method = "GET", $params = array(), $wait_response = true);
```

#### Arguments

- *(string)* *__$endpoint__*  
  See [API Documentation](https://dev.twitter.com/docs/api/1.1).  
  Examples:  
  `statuses/update`, `1.1/statuses/update`, `https://api.twitter.com/1.1/statuses/update.json`
  
- *(string)* *__\[$method\]__*  
  `POST` or `GET`. Case-insensitive. `GET` as default.
  
- *(mixed)* *__\[$params\]__*  
  **Query String** or **Associative Array**.  
  Examples:
  ```php
  $params = 'status=TestTweet';
  $params = array('status' => 'TestTweet');
  ```
  For uploading files:
  ```php
  $params = '@image=' . $filename;
  $params = array('@image' => $filename);
  $params = array('image' => base64_encode(file_get_contents($filename)));
  ```
  
  **Note:**  
  You can't use this with `statuses/update_with_media`.  
  Use **UltimateOAuth::postMultipart()** instead.
  
- *(boolean)* *__\[$wait\_resposne\]__*  
  `TRUE` as default.
  See below.

#### Return Value

- If successfully, return decoded JSON.  
  Basically it is returned as **stdClass**.  
  Some endpoints return **Array**.  
  Example: `statuses/home_timeline`, `users/lookup`  
  Some endpoints get **Query String**, but it is parsed and returned as **stdClass**.  
  Example: `oauth/request_token`, `oauth/access_token`
  
- If it failed, return an **Error Object**.  
  It has the following structure.
  
  > - *(integer)* `$response->errors[0]->code`  
  >   **HTTP STATUS CODE**. Not an error code.  
  >   All error codes are overwritten with HTTP status codes.  
  >   If a local error occurred, this will be `-1`.
  >   
  > - *(string)* `$response->errors[0]->message`  
  >   An error message.
  
- If `$wait_response` is set to `FALSE`, quickly return `NULL`.


=========================================


### UltimateOAuth::get()<br />UltimateOAuth::post()

A wrapper for **UltimateOAuth::OAuthRequest()**.

```php
$uo->get($endpoint, $params);
$uo->post($endpoint, $params, $wait_response);
```

=========================================

### UltimateOAuth::postMultipart()<br />UltimateOAuth::OAuthRequestMultipart()

Mainly used for the endpoint `statuses/update_with_media`.  
**postMultipart()** and **OAuthRequestMultipart()** are completely equal.

```php
$uo->postMultipart($endpoint, $params, $wait_response);
$uo->OAuthRequestMultipart($endpoint, $params, $wait_response);
```

#### Arguments

- *(string)* *__$endpoint__*  
  Example: `statuses/update_with_media`
  
- *(string)* *__\[$params\]__*  
  Examples:
  ```php 
  $params = '@media[]=' . $filename;
  $params = array('@media[]' => $filename);
  $params = array('media[]' => file_get_contents($filename));
  ```
  
- *(boolean)* *__\[$wait\_response\]__*  
  Same as  **UltimateOAuth::OAuthRequest()**.  
  `TRUE` as default.
  
#### Return Value
- Same as **UltimateOAuth::OAuthRequest()**.


=========================================

### UltimateOAuth::directGetToken()

This method enables you to use OAuth like xAuth.  
This was named as **para-xAuth** authentication.

```php
$uo->directGetToken($username, $password);
```

#### Arguments
 
- *(string)* *__$username__*  
  *screen_name* or E-mail Address.
  
- *(string)* *__$password__*  
  password.
  
#### Return Value
- Same as **UltimateOAuth::OAuthRequest()** when requesting `oauth/access_token`.


=========================================

### UltimateOAuth::getAuthenticateURL()<br />UltimateOAuth::getAuthorizeURL()

Get URL for authorization/authentication.

```php
$uo->getAuthenticateURL($force_login);
$uo->getAuthorizeURL($force_login);
```

#### Arguments

- *(boolean)* *__$force\_login__*  
  Whether force logined user to login again.
  `FALSE` as default.
  
#### Return Value

- A URL String.

#### Note: What is the difference between *Authenticate* and *Authorize* ?

|                | Authenticate  |  Authorize   |
| -------------: |:---------------:| :-----------:|
| New User       | Jump to Twitter | Jump to Twitter |
| Logined User   | Jump to Twitter, but if you set your application<br /> **__Allow this application to be used to Sign in with Twitter__**, <br />quickly jump back to your callback URL.  |  Jump to Twitter  |

------------------------------------------------------------------

2-2. Class Detail - UltimateOAuthMulti
--------------------------------------

This class enables you to execute multiple request **parallelly**.


=========================================


### UltimateOAuthMulti::__construct()

Create a new UltimateOAuthMulti instance.

```php
$uom = new UltimateOAuthMulti;
```


=========================================


### UltimateOAuthMulti::enqueue()

Enqueue a new job.

```php
$uom->enqueue($uo, $method, $arg1, $arg2, ...);
```

=========================================

#### Arguments

- *(UltimateOAuth)* *__$uo__*  
  An **UltimateOAuth** object.
  
- *(string)* *__$method__*  
  A method name. This meanas **CLASS METHOD**. Not a HTTP method.  
  Example: `post`
  
- *(mixed)* *__\[$arg1\]__*, *__\[$arg2\]__*, *__\[...\]__*  
  Example: `'statuses/update', 'status=TestTweet'`
  
#### Note

Arguments containing binary data cannot be enqueued.  
You have to enqueue them with `@` prefix.  
Example: `'status=test&@media[]=test.jpg'`

=========================================
  
  
### UltimateOAuthMulti::execute()

Execute All jobs.  
After executing, all queues are dequeued.

```php
$uom->execute($wait_processes, $use_cwd);
```

#### Arguments

- *(boolean)* *__\[$wait\_processes\]__*  
  Whether synchronous or not. `TRUE` as default.
  
- *(boolean)* *__\[$use\_cwd\]__*  
  Whether use current working directory, or use the directory this library exists in.  
  `FALSE` **(UltimateOAuth.php directory)** as default.  
  This cannot be `TRUE` when `USE_PROC_OPEN == False`.
  
#### Return Value

- Return an **Array**, collection of the results.


------------------------------------------------------------------


2-3. Class Detail - UltimateOAuthRotate
---------------------------------------

This class enables you to **avoid API limits** easily.  
Also you can use very useful **secret endpoints**, like:

- `GET activity/about_me`  
  Get activities about me.
- `GET activity/by_friends`  
  Get activities by friends.
- `GET statuses/:id/activity/summary`  
  Get activities about a specified status.
- `GET conversation/show/:id`  
  Get statuses related to a specified status.
- `POST friendships/accept`  
  Accept a specified follower request.
- `POST friendships/deny`  
  Deny a specified follower request.
- `POST friendships/accept_all`  
  Accept all follower requests.
- `POST account/generate`  
  Create a new account.

=========================================
  
### UltimateOAuthRotate::__construct()

Create a new UltimateOAuthRotate instance.

```php
$uor = new UltimateOAuthRotate;
```

=========================================

### UltimateOAuthRotate::register()

Register your own application.

```php
$uor->register($name, $consumer_key, $consumer_secret);
```

#### Arguments

- *(string)* *__$name__*  
  Any name is okay as long as not duplicate with official applications already registered.  
  Just used for identification.  
  Example: `my_app_01`
  
- *(string)* *__$consumer\_key__*

- *(string)* *__$consumer\_secret__*

#### Return Value

- Return result as `TRUE` or `FALSE`.


=========================================


### UltimateOAuthRotate::login()

Login with all registered applications.  
This method depends on **UltimateOAuthMulti** class.

```php
$uor->login($username, $password, $return_array, $successively);
```

#### Arguments and Return Value
 
- *(string)* *__$username__*  
  *screen_name* or E-mail Address.
  
- *(string)* *__$password__*  
  password.
  
- *(string)* *__\[$return\_array\]__*  
Whether return responses as **array**, or if all successful as **boolean**.  
`FALSE` **(Return Boolean)** as default.
  
- *(boolean)* *__\[$successively\]__*  
Whether successively do all jobs, or parallelly do by UltimateOAuthMulti class.  
`FALSE` **(By UltimateOAuthMulti)** as default.

=========================================

### UltimateOAuthRotate::setCurrent()

Select an application for **POST** requesting.  
GET requests have nothing to do with this.

```php
$uor->setCurrent($name);
```

#### Arguments

- *(string)* *__$name__*  
  Example: `my_app_01`

#### Return Value

- Return result as `TRUE` or `FALSE`.


=========================================


### UltimateOAuthRotate::getInstance($name)

Get a **clone** of specified UltimateOAuth Instance.

```php
$uor->getInstance($name);
```

#### Arguments

- *(string)* *__$name__*  
  Example: `my_app_01`
  
#### Return Value

- Return **UltimateOAuth** instance or `FALSE`.

=========================================


### UltimateOAuthRotate::getInstances()

Get **clones** of all UltimateOAuth Instance.

```php
$uor->getInstances($type);
```

#### Arguments

- *(integer)* *__$type__*  
  __0__ - Return all instances **(Default)**  
  __1__ - Return official instances  
  __2__ - Return original instances  
  __3__ - Return sign-up instances
  
#### Return Value

- Return an **Array**, collection of the UltimateOAuth instances.


=========================================


### UltimateOAuthRotate::__call()

You can call an **UltimateOAuth** method.  
A magic method `__call()` enables you to do like this.

Example:
```php
$uor->get('statuses/home_timeline');
```

------------------------------------------------------------------

3. Other Examples
------------------

### Generate an account

#### Important

This library apply the header contents of  
`x-twitter-new-account-oauth-access-token`, `x-twitter-new-account-oauth-secret`  
as response properties.  
You can use `$response->access_token`, `$response->access_token_secret`.

#### Sample code

```php
<?php

// Load this library
require_once('UltimateOAuth.php');

// Get a sign-up instance
$uor = new UltimateOAuthRotate;
$base = $uor->getInstance('Twitter for Android Sign-Up');

// Authorize yourself
$base->directGetToken('Your screen_name', 'Your password');

// Create a new account
$random_string = substr(md5(mt_rand()), 0, 15);
$res = $base->post('account/generate', array(
    'name'        => 'HAHAHAHA',
    'screen_name' => $random_string,
    'password'    => 'test1234',
    'email'       => $random_string . '@examples.com',
));

// Check errors
if (isset($res->errors)) {
    die(sprintf('Error[%d]: %s',
        $res->errors[0]->code,
        $res->errors[0]->message
    ));
}

// Test tweet
$tmp = $uor->getInstance('Twitter for Android');
$uo = new UltimateOAuth($tmp->consumer_key, $tmp->consumer_secret, $res->access_token, $res->access_token_secret);
$uo->post('statuses/update', 'status=HAHAHAHA!!!!');
```

================================

### Walk cursors

Some endpoints require walking cursors for getting all results.

#### Sample code

Fetch random 250 screen_name among all followers.

```php
<?php

// Load this library
require_once('UltimateOAuth.php');

// Create a new instance
$uo = new UltimateOAuth('CK', 'CS', 'AT', 'AS');

// Initialization
$cursor = '-1';
$result = array();
$screen_names = array();

// Walk cursors
do {
    $res = $uo->get('followers/ids', 'stringify_ids=1&cursor=' . $cursor);
    if (isset($res->errors)) {
        die(sprintf('Error[%d]: %s',
            $res->errors[0]->code,
            $res->errors[0]->message
        ));
    }
    $result += array_flip($res->ids);
    $cursor = $res->next_cursor_str;
} while ($cursor);

// Get 250 of them
$count = count($result);
$result = $count <= 250 ? array_keys($result) : array_rand($result, 250);

// Get all screen_names of 250
for ($i = 0; $i < 250; $i += 100) {
    $res = $uo->get('users/lookup', 'user_id=' . implode(',', array_slice($result, $i, 100)));
    if (isset($res->errors)) {
        die(sprintf('Error[%d]: %s',
            $res->errors[0]->code,
            $res->errors[0]->message
        ));
    }
    foreach ($res as $user) {
        $screen_names[] = $user->screen_name;
    }
}

// Output
print_r($screen_names);
```

===============================

### Linkify urls, user_mentions and so on by entities

Use this library.  
[TwitterText](https://github.com/Certainist/TwitterText)



