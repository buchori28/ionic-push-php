# ionic-push-php [![Release](https://img.shields.io/github/release/tomloprod/ionic-push-php.svg)](https://github.com/tomloprod/ionic-push-php) [![License](https://img.shields.io/github/license/tomloprod/ionic-push-php.svg)](http://www.opensource.org/licenses/mit-license.php) 

ionic-push-php is a library that allows you to consume the *Ionic Cloud API* for **sending push notifications** (*normal and scheduled*), get a paginated **list of sending push notifications**,  get **information of registered devices**, **remove registered devices by token**, ...

Ionic official documentation: [Ionic HTTP API - Push](https://docs.ionic.io/api/endpoints/push.html).

## Requirements:

- PHP 5.1+
- cURL

## Installation:

    composer require tomloprod/ionic-push-php

## Configuration:


First, make sure you have your `$ionicAPIToken` and your `$ionicProfile`:

- (string) **$ionicAPIToken:** The API token that you must create in *Settings › API Keys* in the [Dashboard](https://apps.ionic.io).
- (string) **$ionicProfile:** The Security Profile tag found in *Settings › Certificates* in the [Dashboard](https://apps.ionic.io)

> More information [here](https://github.com/tomloprod/ionic-push-php/issues/1).

If you don't know how to configure your ionic app, you can take a look here: [Setup Ionic Push](http://docs.ionic.io/services/push/#setup)


## How to use:

First, instance an object as follow:

```php
use Tomloprod\IonicApi\Push;

$ionicPushApi = new Push($ionicProfile, $ionicAPIToken);
```

Then you can interact (*list, remove, create, ...*) with `device tokens`, `messages` and `notifications`:

### [Device Tokens]

 **1) List tokens:**
 
```php
// [OPTIONAL] Indicates whether the JSON response will be converted to a PHP variable before return. 
// Default => false
$decodeJson = true;

$response = $ionicPushApi->deviceTokens->paginatedList([
    // Determines whether to include invalidated tokens (boolean)
    'show_invalid' => 1,
    // Only display tokens associated with the User ID (string)
    'user_id' => $desiredUserId,
    // Sets the number of items to return per page (integer)
    'page_size' => 4,
    // Sets the page number (integer)
    'page' => 1
], $decodeJson);

// If response is an object with data, we loop through each device token:
if(is_object($response) && property_exists($response, "data")) {
    foreach($response->data as $deviceToken){
        // Show the type (ios, android, ...) of each device token.
        echo $deviceToken->type;
    }
}
```

 **2) List users associated with a device token:**
 
```php
// [OPTIONAL] Indicates whether the JSON response will be converted to a PHP variable before return. 
// Default => false
$decodeJson = true;

$associatedUsers = $ionicPushApi->deviceTokens->listAssociatedUsers($desiredDeviceToken, [
    // Sets the number of items to return per page (integer)
    'page_size' => 1,
    // Sets the page number (integer)
    'page' => 1,
], $decodeJson);
```

**3) Create device token that was previously generated by a device platform:**

```php
$createdDeviceToken = $ionicPushApi->deviceTokens->create([
    // Device token (string)
    'token' => $newToken,
    // User ID. Associate the token with the User (string)
    'user_id' => $uuid 
]);
```

**4) Retrieve device information related to the device token:**

```php
$deviceInformation = $ionicPushApi->deviceTokens->retrieve($desiredDeviceToken);
```

**5) Update an specific token:**

```php
$updatedDeviceInformation = $ionicPushApi->deviceTokens->update($desiredDeviceToken, [
    // Determines whether the device token is valid (boolean)
    'valid' => 1
]);
```

**6) Delete a device related to the device token:**

```php
$deleteResult = $ionicPushApi->deviceTokens->delete($desiredDeviceToken);
```

### [Messages]

**1) Retrieve specific message:**

```php
$message = $ionicPushApi->messages->retrieve($desiredMessageId);
```

**2) Delete a message:**

```php
$deleteResult = $ionicPushApi->messages->delete($desiredMessageId);
```

### [Notifications]
 
**1) List notifications:**
```php
// [OPTIONAL] Indicates whether the JSON response will be converted to a PHP variable before return. 
// Default => false
$decodeJson = true;

$response = $ionicPushApi->notifications->paginatedList([
    // Sets the number of items to return per page (integer)
    'page_size' => 1,
    // Sets the page number (integer)
    'page' => 1,
    // You can also pass other fields like "message_total" or "overview" (string[])
    'fields' => [
        // Total number of messages tied to each notification.
        'message_total',
        // Get an overview of messages delivered and failed for each notification.
        'overview'
    ]
], $decodeJson);

// If response is an object with data, we loop through each notification:
if(is_object($response) && property_exists($response, "data")) {
    foreach($response->data as $notification){
        // Show the message of each notification.
        echo $notification->config->notification->message;
    }
}
```

**2) Retrieve specific notification:**

```php
$notification = $ionicPushApi->notifications->retrieve($desiredNotificationId);
```
 
**3) Delete a notification:**

```php
$deleteResult = $ionicPushApi->notifications->delete($desiredNotificationId);
```

**4) Delete all notifications:**

```php
// Return true if all notifications have been deleted.
$areAllDeleted = $ionicPushApi->notifications->deleteAll();
```

**5) List messages of a notification:**

```php
$messages = $ionicPushApi->notifications->listMessages($desiredNotificationId, [
    // Sets the number of items to return per page (integer)
    'page_size' => 1,
    // Sets the page number (integer)
    'page' => 1
])
 ```
 
**6) Send notifications:**

```php
// Configuration of the notification
$notificationConfig = [
    'title' => 'Your notification title',
    'message' => 'Your notification message. Bla, bla, bla, bla.',
    'android' => [
        'tag' => 'YourTagIfYouNeedIt'
    ],
    'ios' => [
        'priority' => 10,
        'badge' => 1
    ]
];

// [OPTIONAL] You can also pass custom data to the notification. Default => []
$payload = [ 
    'myCustomField' => 'This is the content of my customField',
    'anotherCustomField' => 'More custom content'
];

// [OPTIONAL] And define, if you need it, a silent notification. Default => false
$silent = true;

// [OPTIONAL] Or/and even a scheduled notification for an indicated datetime. Default => ''
$scheduled = '2016-12-10 10:30:10';


// Configure notification:
$ionicPushApi->notifications->setConfig($notificationConfig, $payload, $silent, $scheduled);

// Send notification...
$ionicPushApi->notifications->sendNotificationToAll(); // ...to all registered devices
$ionicPushApi->notifications->sendNotification([$desiredToken1, $desiredToken2, $desiredToken3]); // ...to some devices
```
    
##### *NOTE: You will need to parse the returned array of the methods that return information.*

<br>

## TODO:

1. Methods replace() ~~and listMessages()~~ of **Notifications**.
1. Methods ~~listAssociatedUsers(),~~ associateUser() and dissociateUser() of **DeviceTokens** : both implemented, but pending testing
