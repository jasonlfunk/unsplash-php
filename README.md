# PHP Unsplash Wrapper

[![Build Status](https://travis-ci.org/unsplash/unsplash-php.svg?branch=master)](https://travis-ci.org/unsplash/unsplash-php)

A PHP client for the [Unsplash API](https://unsplash.com/documentation).

- [Official documentation](https://unsplash.com/documentation)
- [Changelog](https://github.com/unsplash/unsplash-pHP/blob/master/CHANGELOG.md)

Quick links to methods you're likely to care about:

- [Get a list of new photos](#photo-all) 🎉
- [Get a random photo](#photo-random) 🎑
- [Trigger a photo download](#photo-download) 📡
- [Search for a photo by keyword](#search-photos) 🕵️‍♂️

**Note:** Every application must abide by the [API Guidelines](https://help.unsplash.com/api-guidelines/unsplash-api-guidelines). Specifically, remember to [hotlink images](https://help.unsplash.com/api-guidelines/more-on-each-guideline/guideline-hotlinking-images) and [trigger a download when appropriate](https://help.unsplash.com/api-guidelines/more-on-each-guideline/guideline-triggering-a-download).

## Installation

`unsplash-php` uses [Composer](https://getcomposer.org/). To use it, require the library

```
composer require unsplash/unsplash
```

## Usage

### Configuration

Before using, configure the client with your access key and secret. If you don't have an access key and secret, follow the steps from the [Unsplash API](https://unsplash.com/documentation#creating-a-developer-account) to register your application.

Note: if you're just using actions that require the [public permission scope](#permission-scopes), only the access key is required. Access key is entered as `applicationId` due to legacy reasons.

Note: if utmSource is omitted from $credentials a notice will be raised.

```php
Unsplash\HttpClient::init([
	'applicationId'	=> 'YOUR ACCESS KEY',
	'secret'	=> 'YOUR APPLICATION SECRET',
	'callbackUrl'	=> 'https://your-application.com/oauth/callback',
	'utmSource' => 'NAME OF YOUR APPLICATION'
]);
```

### User Authorization workflow

If you need to access actions that are non-public on behalf of the user (i.e. uploading a photo to a specific account), you'll need to follow the [user authentication workflow](https://unsplash.com/documentation/user-authentication-workflow) to access their data.

An example of this flow can be found in /examples/oauth-flow.php

Direct them to an authorization URL (configuring any scopes before generating the authorization URL):

```php
$scopes = ['public', 'write_user'];
Unsplash\HttpClient::$connection->getConnectionUrl($scopes);
```

Upon authorization, Unsplash will return to you an authentication code via your OAuth
callback handler. Use it to generate an access token:

```php
Unsplash\HttpClient::$connection->generateToken($code);
```

With the token you can now access any additional non-public actions available for the authorized user.


#### Permission Scopes

The current permission scopes defined by the [Unsplash API](https://unsplash.com/documentation/user-authentication-workflow#permission-scopes) are:

- `public` (Access a user's public data)
- `read_user` (Access a user's private data)
- `write_user` (Edit and create user data)
- `read_photos` (Access private information from a user's photos)
- `write_photos` (Post and edit photos for a user)
- `write_likes` (Like a photo for a user)
- `read_collections` (View a user’s private collections)
- `write_collections` (Create and update a user’s collections)

----

### API methods

For more information about the the responses for each call, refer to the [official documentation](https://unsplash.com/documentation).

Some parameters are identical across all methods:

  param     | Description
------------|-----------------------------------------------------
`$per_page` | Defines the number of objects per page. *Default 10*
`$page`     | Defines the offset page. *Default 1*

*Note: The methods that return multiple objects return an `ArrayObject`, which acts like a normal stdClass.*

----

### Search

<div id="search-photos" />

#### Photos

Retrieve a single page of photo results depending on search results.

**Arguments**

  Argument     | Type   | Opt/Required
---------------|--------|--------------
`$search`      | string | Required
`$page`        | int    | Opt *(Default: 1)*
`$per_page`    | int    | Opt *(Default: 10 / Maximum: 30)*
`$orientation` | string | Opt *(Default: null / Available: "landscape", "portrait", "squarish")*
`$collections` | string | Opt *(Default: null / If multiple, comma-separated)*

**Example**


```php
$search = 'forest';
$page = 3;
$per_page = 15;
$orientation = 'landscape';

Unsplash\Search::photos($search, $page, $per_page, $orientation);
```

----

#### Collections

Retrieve a single page of collection results depending on search results.

**Arguments**

  Argument     | Type   | Opt/Required
---------------|--------|--------------
`$search`      | string | Required
`$per_page`    | int    | Opt *(Default: 10 / Maximum: 30)*
`$page`        | int    | Opt *(Default: 1)*

**Example**


```php
Unsplash\Search::collections($search, $page, $per_page);
```

----

#### Users

Retrieve a single page of user results depending on search results.

**Arguments**

  Argument     | Type   | Opt/Required
---------------|--------|--------------
`$search`      | string | Required
`$per_page`    | int    | Opt *(Default: 10 / Maximum: 30)*
`$page`        | int    | Opt *(Default: 1)*

**Example**


```php
Unsplash\Search::users($search, $page, $per_page);
```

----

### Collections

####
Retrieve the list of collections.

**Arguments**

  Argument     | Type | Opt/Required
---------------|------|--------------
`$per_page`    | int  | Opt *(Default: 10 / Maximum: 30)*
`$page`        | int  | Opt *(Default: 1)*

**Example**


```php
Unsplash\Collection::all($page, $per_page);
```

----

#### Unsplash\Collection::photos($page, $per_page)
Retrieve photos from a collection.

*Note:* You need to instantiate a collection object first.

**Arguments**

  Argument     | Type | Opt/Required
---------------|------|--------------
`$per_page`    | int  | Opt *(Default: 10 / Maximum: 30)*
`$page`        | int  | Opt *(Default: 1)*

**Example**

```php
$collection = Unsplash\Collection::find(integer $id);
$photos = $collection->photos($page, $per_page);
```

----

#### Unsplash\Collection::related($page, $per_page)
Retrieve list of featured collections.

*Note* You must instantiate a collection first

**Arguments**

  Argument     | Type | Opt/Required
---------------|------|--------------


**Example**


```php
$collection = Unsplash\Collection::find($id);
$collection->related();
```

----

#### Unsplash\Collection::create($title, $description, $private)
Create a collection on the user's behalf.

*Note:* You need the `write_collections` permission scope

**Arguments**

  Argument     | Type    | Opt/Required
---------------|---------|--------------
`$title`       | string  | Required
`$description` | string  | Opt *(Default: '')*
`$private`     | boolean | Opt *(Default: false)*

**Example**

```php
$collection = Unsplash\Collection::create($title);
```

----

#### Unsplash\Collection::update($parameters)
Update a collection on the user's behalf.

*Note:* You need to instantiate a collection object first

*Note:* You need the `write_collections` permission scope

**Arguments**

  Argument     | Type    | Opt/Required | Note
---------------|---------|----------------------
`$parameters`  | array   | Required     | The following keys can be set in the array : `title`, `description`, `private`

**Example**

```php
$collection = Unsplash\Collection::find(int $id);
$collection->update(['private' => true])
```

----

#### Unsplash\Collection::destroy()
Delete a collection on the user's behalf.

*Note:* You need to instantiate a collection object first

*Note:* You need the `write_collections` permission scope

**Example**

```php
$collection = Unsplash\Collection::find(int $id);
$collection->destroy()
```

----

#### Unsplash\Collection::add($photo_id)
Add a photo in the collection on the user's behalf.

*Note:* You need to instantiate a collection object first

*Note:* You need the `write_collections` permission scope

**Arguments**

  Argument     | Type    | Opt/Required |
---------------|---------|---------------
`$photo_id`    | integer | Required     |

**Example**

```php
$collection = Unsplash\Collection::find(int $id);
$collection->add(int $photo_id)
```

----

#### Unsplash\Collection::remove($photo_id)
Remove a photo from the collection on the user's behalf.

*Note:* You need to instantiate a collection object first

*Note:* You need the `write_collections` permission scope

**Arguments**

  Argument     | Type    | Opt/Required |
---------------|---------|---------------
`$photo_id`    | integer | Required     |

**Example**

```php
$collection = Unsplash\Collection::find(int $id);
$collection->remove(int $photo_id)
```

----


### Photo

<div id="photo-all" />

#### Unsplash\Photo::all($page, $per_page, $order_by)
Retrieve a list of photos.

**Arguments**

  Argument     | Type | Opt/Required
---------------|------|--------------
`$per_page`    | int  | Opt *(Default: 10 / Maximum: 30)*
`$page`        | int  | Opt *(Default: 1)*
`$order_by` | string | Opt *(Default: latest / Available: oldest, popular)*

**Example**

```php
Unsplash\Photo::all($page, $per_page, $order_by);
```

----

#### Unsplash\Photo::find($id)
Retrieve a specific photo.

**Arguments**

  Argument     | Type | Opt/Required
---------------|------|--------------
`$id`          | int  | Required

**Example**

```php
Unsplash\Photo::find($id);
```

----

#### Unsplash\Photo::update($parameters = [])
Post a photo on the user's behalf.

*Note:* You need the `write_photos` permission scope
You need to instantiate the Photo object first

**Arguments**

  Argument     | Type   | Opt/Required
---------------|--------|--------------
`$parameters`   | array | Required

**Example**

```php
$photo = Unsplash\Photo::find(string $id)
$photo->update(array $parameters);
```

----

#### Unsplash\Photo::photographer()
Retrieve the photo's photographer.

*Note:* You need to instantiate a photo object first

**Arguments**

*N/A*

**Example**


```php
$photo = Unsplash\Photo::find(string $id);
$photo->photographer();
```

----

<div id="photo-random" />

#### Unsplash\Photo::random([featured => $value, username => $value, query => $value, w => $value, h => $value])
Retrieve a random photo from specified filters. For more information regarding filtering, [refer to the Offical documentation](https://unsplash.com/documentation#get-a-random-photo).

*Note:* An array needs to be passed as a parameter.

**Arguments**


  Argument     | Type | Opt/Required
---------------|------|--------------
featured | boolean | Opt *(Limit selection to featured photos)*
username | string | Opt *(Limit selection to a single user)*
query | string | Opt *(Limit selection to photos matching a search term)*
w | int | Opt *(Image width in pixels)*
h | int | Opt *(Image height in pixels)*


**Example**


```php

// Or apply some optional filters by passing a key value array of filters
$filters = [
    'username' => 'andy_brunner',
    'query'    => 'coffee',
    'w'        => 100,
    'h'        => 100
];
Unsplash\Photo::random($filters);
```

----

#### Unsplash\Photo::like()
Like a photo on the user's behalf.

*Note:* You need to instantiate a photo object first

*Note:* You need the `like_photos` permission scope

**Arguments**

*N/A*

**Example**


```php
$photo = Unsplash\Photo::find(string $id);
$photo->like();
```

----

#### Unsplash\Photo::unlike()
Unlike a photo on the user's behalf.

*Note:* You need to instantiate a photo object first

*Note:* You need the `like_photos` permission scope

**Arguments**

*N/A*

**Example**


```php
$photo = Unsplash\Photo::find(string $id);
$photo->unlike();
```

----

#### Unsplash\Photo::statistics(string $resolution, int $quantity)
Retrieve total number of downloads, views and likes of a single photo, as well as the historical breakdown of these stats in a specific timeframe (default is 30 days).

*Note:* You must instantiate a Photo object first

**Arguments**


  Argument     | Type | Opt/Required
---------------|------|--------------
resolution | string | Opt *(Accepts only days currently)*
quantity | int | Opt *(Defaults to 30, can be between 1 and 30)*


**Example**


```php


$photo = Unsplash\Photo::find($id);
$photo->statistics('days', 7);
```

----

<div id="photo-download" />

#### Unsplash\Photo::download()
Trigger a download for a photo. This is needed to follow the ['trigger a download' API Guideline](https://help.unsplash.com/api-guidelines/more-on-each-guideline/guideline-triggering-a-download).

*Note:* You must instantiate a Photo object first

**Arguments**


  Argument     | Type | Opt/Required
---------------|------|--------------


**Example**


```php
$photo = Unsplash\Photo::find();
$photo->download();
```

----

### User

#### Unsplash\User::find($username)
Retrieve a user's information.

**Arguments**

  Argument     | Type   | Opt/Required
---------------|--------|--------------
`$username`    | string | Required

**Example**

```php
Unsplash\User::find($username)
```

----

#### Unsplash\User::portfolio($username)
Retrieve a link to the user's portfolio page.

**Arguments**

  Argument     | Type   | Opt/Required
---------------|--------|--------------
`$username`    | string | Required

**Example**

```php
Unsplash\User::portfolio($username)
```

----

#### Unsplash\User::current()
Retrieve the user's private information.

*Note:* You need the *read_user* permission scope

**Arguments**

*N/A*

**Example**

```php
$user = Unsplash\User::current();
```

----

#### Unsplash\User::photos($page, $per_page, $order_by)
Retrieve user's photos.

*Note:* You need to instantiate a user object first

**Arguments**

  Argument     | Type | Opt/Required
---------------|------|--------------
`$per_page`    | int  | Opt *(Default: 10 / Maximum: 30)*
`$page`        | int  | Opt *(Default: 1)*
`$order_by` | string | Opt *(Default: latest / Available: oldest, popular)*

**Example**

```php
$user = Unsplash\User::find($username);
$user->photos($page, $per_page);
```

----


#### Unsplash\User::collections($page, $per_page)
Retrieve user's collections.

*Note:* You need to instantiate a user object first
*Note:* You need the *read_collections* permission scope to retrieve user's private collections

**Arguments**

  Argument     | Type | Opt/Required
---------------|------|--------------
`$per_page`    | int  | Opt *(Default: 10 / Maximum: 30)*
`$page`        | int  | Opt *(Default: 1)*

**Example**

```php
$user = Unsplash\User::find($username);
$user->collections($page, $per_page);
```

----

#### Unsplash\User::likes($page, $per_page, $order_by)
Retrieve user's collections.

*Note:* You need to instantiate a user object first

**Arguments**

  Argument     | Type | Opt/Required
---------------|------|--------------
`$per_page`    | int  | Opt *(Default: 10 / Maximum: 30)*
`$page`        | int  | Opt *(Default: 1)*
`$order_by` | string | Opt *(Default: latest / Available: oldest, popular)*


**Example**

```php
$user = Unsplash\User::find($username);
$user->likes($page, $per_page, $order_by);
```

----


#### Unsplash\User::update([$key => value])
Update current user's fields. Multiple fields can be passed in the array.

*Note:* You need to instantiate a user object first

*Note:* You need the *write_user* permission scope.

**Arguments**

  Argument     | Type   | Opt/Required | Note  |
---------------|--------|--------------|-------|
`$key`         | string | Required     | The following keys are accepted: `username`, `first_name`, `last_name`, `email`, `url`, `location`, `bio`, `instagram_username`
`$value`       | mixed  | required

```php
$user = Unsplash\User::current();
$user->update(['first_name' => 'Elliot', 'last_name' => 'Alderson']);
```

#### Unsplash\User::statistics(string $resolution, int $quantity)
Retrieve total number of downloads, views and likes for a user, as well as the historical breakdown of these stats in a specific timeframe (default is 30 days).

*Note:* You must instantiate the User object first

**Arguments**


  Argument     | Type | Opt/Required
---------------|------|--------------
resolution | string | Opt *(Accepts only days currently)*
quantity | int | Opt *(Defaults to 30, can be between 1 and 30)*


**Example**


```php
$user = Unsplash\User::find($id);
$user->statistics('days', 7);
```

----

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/unsplash/unsplash-php. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org/) code of conduct.
