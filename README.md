Instagram PHP (v.2.0)

The [Instagram Platform](http://instagram.com/developer/) is
a set of APIs that make your app more social.

Requirements
-----

- PHP 5.2.x or higher
- cURL
- Registered Instagram App

Get started
-----

[Register your application](http://instagram.com/developer/register/) with Instagram, and receive your OAuth `client_id` and `client_secret`.  
Take a look at the [uri guidlines](#samples-for-redirect-urls) before registering a redirect URI.

Usage
-----

The [examples][examples] are a good place to start. The minimal you'll need to
have is:

```php
<?php
    require_once 'instagram-php/instagram.php';

    $instagram = new Instagram(array(
      'client_id'  => 'YOUR_CLIENT_ID',
      'client_secret' => 'YOUR_CLIENT_SECRET',
      'redirect_url' => 'YOUR_REDIRECT_URL',
    ));

    echo "<a href='{$instagram->getLoginUrl()}'>Login with Instagram</a>";
?>
```
### Authenticate user (OAuth2)

```php
<?php
    // Grab OAuth callback code
    $code = $_GET['code'];
    $data = $instagram->getOAuthToken($code);
    
    echo 'Your username is: ' . $data->user->username;
?>
```

### Get user likes

```php
<?php
    // Store user access token
    $instagram->setAccessToken($data);
    
    // Get all user likes
    $likes = $instagram->getUserLikes();
    
    // Take a look at the API response
    echo '<pre>';
    print_r($likes);
    echo '<pre>';
?>
```

**All methods return the API data `json_decode()` - so you can directly access the data.**

## Available methods

### Setup Instagram

`new Instagram(<array>/<string>);`

`array` if you want to authenticate a user and access its data:

```php
new Instagram(array(
      'client_id'  => 'YOUR_CLIENT_ID',
      'client_secret' => 'YOUR_CLIENT_SECRET',
      'redirect_url' => 'YOUR_REDIRECT_URL',
));
```

`string` if you *only* want to access public data:

```php
new Instagram('YOUR_APP_KEY');
```

### Get login URL

`getLoginUrl(<array>)`

```php
getLoginUrl(array(
  'basic',
  'likes'
));
```

**Optional scope parameters:**

<table>
  <tr>
    <th>Scope</th>
    <th>Legend</th>
    <th>Methods</th>
  </tr>
  <tr>
    <td><code>basic</code></td>
    <td>to use all user related methods [default]</td>
    <td><code>getUser()</code>, <code>getUserFeed()</code>, <code>getUserFollower()</code> etc.</td>
  </tr>
  <tr>
    <td><code>relationships</code></td>
    <td>to follow and unfollow users</td>
    <td><code>modifyRelationship()</code></td>
  </tr>
  <tr>
    <td><code>likes</code></td>
    <td>to like and unlike items</td>
    <td><code>getMediaLikes()</code>, <code>likeMedia()</code>, <code>deleteLikedMedia()</code></td>
  </tr>
  <tr>
    <td><code>comments</code></td>
    <td>to create or delete comments</td>
    <td><code>getMediaComments()</code>, <code>addMediaComment()</code>, <code>deleteMediaComment()</code></td>
  </tr>
</table>

### Get OAuth token

`getOAuthToken($code, <true>/<false>)`

`true` : Returns only the OAuth token  
`false` *[default]* : Returns OAuth token and profile data of the authenticated user

### Set / Get access token

Stores access token, for further method calls:  
`setAccessToken($token)`

Returns access token, if you want to store it for later usage:  
`getAccessToken()`

### User methods

**Public methods**

- `getUser($id)`
- `searchUser($name, <$limit>)`

**Authenticated methods**

- `getUser()`
- `getUserLikes(<$limit>)`
- `getUserFeed(<$limit>)`
- `getUserMedia(<$id>, <$limit>)`
    - if an `$id` isn't defined, it returns the media of the logged in user

> [Sample responses of the User Endpoints.](http://instagram.com/developer/endpoints/users/)

### Relationship methods

**Authenticated methods**

- `getUserFollows(<$id>, <$limit>)`
- `getUserFollower(<$id>, <$limit>)`
- `getUserRelationship($id)`
- `modifyRelationship($action, $user)`
    - `$action` : Action command (follow / unfollow / block / unblock / approve / deny)
    - `$user` : Target user id

```php
<?php
    // Follow the user with the ID 1574083
    $instagram->modifyRelationship('follow', 1574083);
?>
```

---

Please note that the `modifyRelationship()` method requires the `relationships` [scope](#get-login-url).

---

> [Sample responses of the Relationship Endpoints.](http://instagram.com/developer/endpoints/relationships/)

### Media methods

**Public methods**

- `getMedia($id)`
- `getPopularMedia()`
- `searchMedia($lat, $lng, <$distance>)`
    - `$lat` and `$lng` are coordinates and have to be floats like: `48.145441892290336`,`11.568603515625`
    - `$distance` Radial distance in meter (max. distance: 5km = 5000)

All `<$limit>` parameters are optional. If the limit is undefined, all available results will be returned.

> [Sample responses of the Media Endpoints.](http://instagram.com/developer/endpoints/media/)

### Comment methods

**Public methods**

- `getMediaComments($id)`

**Authenticated methods**

- `addMediaComment($id, $text)`
    - **restricted access:** please email `apidevelopers[at]instagram.com` for access
- `deleteMediaComment($id, $commentID)`
    - the comment must be authored by the authenticated user

---

Please note that the authenticated methods require the `comments` [scope](#get-login-url).

---

> [Sample responses of the Comment Endpoints.](http://instagram.com/developer/endpoints/comments/)

### Tag methods

**Public methods**

- `getTag($name)`
- `getTagMedia($name)`
- `searchTags($name)`

> [Sample responses of the Tag Endpoints.](http://instagram.com/developer/endpoints/tags/)

### Likes methods

**Authenticated methods**

- `getMediaLikes($id)`
- `likeMedia($id)`
- `deleteLikedMedia($id)`

> How to like a Media: [Example usage](https://gist.github.com/3287237)  
> [Sample responses of the Likes Endpoints.](http://instagram.com/developer/endpoints/likes/)

## Instagram videos

Instagram entries are marked with a `type` attribute (`image` or `video`), that allows you to identify videos.

An example of how to embed Instagram videos by using [Video.js](http://www.videojs.com), can be found in the `/example` folder.  

---

**Please note:** Instagram currently doesn't allow to filter videos.

---

## Pagination

Each endpoint has a maximum range of results, so increasing the `limit` parameter above the limit won't help (e.g. `getUserMedia()` has a limit of 90).

That's the point where the "pagination" feature comes into play.  
Simply pass an object into the `pagination()` method and receive your next dataset:

```php
<?php
    $photos = $instagram->getTagMedia('kitten');

    $result = $instagram->pagination($photos);
?>
```

Samples for redirect URLs
-----

<table>
  <tr>
    <th>Registered Redirect URI</th>
    <th>Redirect URI sent to /authorize</th>
    <th>Valid?</th>
  </tr>
  <tr>
    <td>http://example.com/</td>
    <td>http://example.com/</td>
    <td>yes</td>
  </tr>
  <tr>
    <td>http://example.com/</td>
    <td>http://example.com/?this=that</td>
    <td>yes</td>
  </tr>
  <tr>
    <td>http://example.com/?this=that</td>
    <td>http://example.com/</td>
    <td>no</td>
  </tr>
  <tr>
    <td>http://example.com/?this=that</td>
    <td>http://example.com/?this=that&another=true</td>
    <td>yes</td>
  </tr>
  <tr>
    <td>http://example.com/?this=that</td>
    <td>http://example.com/?another=true&this=that</td>
    <td>no</td>
  </tr>
  <tr>
    <td>http://example.com/callback</td>
    <td>http://example.com/</td>
    <td>no</td>
  </tr>
  <tr>
    <td>http://example.com/callback</td>
    <td>http://example.com/callback/?type=mobile</td>
    <td>yes</td>
  </tr>
</table>

> If you need additional informations, take a look at [Instagrams API docs](http://instagram.com/developer/authentication/).


[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/meownosaurus/instagram-php-sdk/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

