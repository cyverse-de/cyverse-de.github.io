---
layout: page
title: DE API Documentation
---

# Table of Contents

* [Miscellaneous Terrain Endpoints](#miscellaneous-terrain-endpoints)
    * [Verifying that Terrain is Running](#verifying-that-terrain-is-running)
    * [Initializing a User's Workspace](#initializing-a-users-workspace)
    * [Saving User Session Data](#saving-user-session-data)
    * [Retrieving User Session Data](#retrieving-user-session-data)
    * [Removing User Session Data](#removing-user-seession-data)
    * [Saving User Preferences](#saving-user-preferences)
    * [Retrieving User Preferences](#retrieving-user-preferences)
    * [Removing User Preferences](#removing-user-preferences)
    * [Obtaining Identifiers](#obtaining-identifiers)
    * [Submitting User Feedback](#submitting-user-feedback)
    * [Getting a user's saved searches](#getting-saved-searches)
    * [Setting a user's saved searches](#setting-saved-searches)
    * [Deleting a user's saved searches](#deleting-saved-searches)

# Miscellaneous Terrain Endpoints

Note that secured endpoints in Terrain and apps are a little different from each other. Please see [Terrain Vs. Apps](terrain-v-apps.html) for more information.

## Verifying that Terrain is Running

Unsecured Endpoint: GET /

The root path in Terrain can be used to verify that Terrain is actually running and is responding. Currently, the response to this URL contains only a welcome message. Here's an example:

```
$ curl -s http://by-tor:8888/
The infinite is attainable with Terrain!
```

## Initializing a User's Workspace and Preferences

Secured Endpoint: GET /secured/bootstrap

Delegates to apps: `POST /users/login`

Delegates to apps: `GET /workspaces`

Delegates to data-info: `GET /navigation/base-paths`

Delegates to preferences: `GET /{username}`


The DE calls this service as soon as the user logs in to initialize the user's workspace
if the user has never logged in before, and returns user information,
including the user's preferences, the user's home path, the user's trash path, and the base trash.
This service always records the fact that the user logged in.

Note that the required `ip-address` query parameter cannot be obtained automatically in most cases.
Because of this, the `ip-address` parameter must be passed to this service.
Here's an example:

```json
$ curl -H "$AUTH_HEADER" "http://by-tor:8888/secured/bootstrap?ip-address=127.0.0.1" | python -mjson.tool
{
  "user_info": {
    "username": "snow-dog",
    "full_username": "snow-dog@iplantcollaborative.org",
    "email": "sd@example.org",
    "first_name": "Snow",
    "last_name": "Dog"
  },
  "session": {
    "login_time": 1374190755304,
    "auth_redirect": {
      "example": "https://example.org"
    }
  },
  "workspace": {
    "new_workspace": false,
    "workspace_id": "4"
  },
  "data_info": {
    "user_home_path": "/iplant/home/snow-dog",
    "user_trash_path": "/iplant/trash/snow-dog",
    "base_trash_path": "/iplant/trash"
  },
  "preferences": {
    "system_default_output_dir": {
      "id": "/iplant/home/snow-dog/analyses",
      "path": "/iplant/home/snow-dog/analyses"
    },
    "default_output_folder": {
      "id": "/iplant/home/snow-dog/analyses",
      "path": "/iplant/home/snow-dog/analyses"
    }
  }
}
```

This endpoint calls different services to obtain the different pieces of its response.
If any of these services (besides terrain) are down,
the bootstrap endpoint can still return success to the client,
but will provide an error message letting the client know which services are down.

For example:

```json
{
  "user_info": {
    "username": "snow-dog",
    "full_username": "snow-dog@iplantcollaborative.org",
    "email": "sd@example.org",
    "first_name": "Snow",
    "last_name": "Dog"
  },
  "session": {
    "error": "java.net.ConnectException: Connection refused"
  },
  "workspace": {
    "status": 400,
    "error": {
      "error_code": "ERR_ILLEGAL_ARGUMENT",
      "reason": {
        "foo": "disallowed-key"
      }
    }
  },
  "data_info": {
    "status": 404,
    "error": {
      "success": false,
      "reason": "unrecognized service path"
    }
  },
  "preferences": {
    "status": 503,
    "error": "<html>\r\n<head><title>503 Service Unavailable</title></head>\r\n<body bgcolor=\"white\">\r\n<center><h1>503 Service Unavailable</h1></center>\r\n<hr><center>nginx/1.11.4</center>\r\n</body>\r\n</html>\r\n"
  }
}
```

These are examples of the different kinds of errors that can show up from any of our services,
so any of these errors can show up in the `session`, `workspace`, `data_info`, or `preferences` error fields.

## Recording when a User Logs Out

Secured Endpoint: GET /secured/logout

The DE calls this service when the user explicitly logs out. This service simply records the time that the user logged out in the login record created by the `/secured/bootstrap` service. Note that this service requires these query-string parameters, which cannot be obtained automatically in most cases:

* ip-address - the source IP address of the logout request
* login-time - the login timestamp that was returned by the bootstrap service

Here's an example:

```
$ curl -sH "$AUTH_HEADER" "http://by-tor:8888/secured/logout?ip-address=127.0.0.1&login-time=1374190755304" | python -mjson.tool
```

Check the HTTP status of the response to tell if it succeeded. It should return a status in the 200 range.

## Saving User Session Data

Secured Endpoint: POST /secured/sessions

This service can be used to save arbitrary JSON user session information. The post body is stored as-is and can be retrieved by sending an HTTP GET request to the same URL.

Here's an example:

```
$ curl -sH "$AUTH_HEADER" -d '{"foo":"bar"}' "http://by-tor:8888/secured/sessions"
```

## Retrieving User Session Data

Secured Endpoint: GET /secured/sessions

This service can be used to retrieve user session information that was previously saved by sending a POST request to the same service.

Here's an example:

```
$ curl -H "$AUTH_HEADER" "http://by-tor:8888/secured/sessions"
{"foo":"bar"}
```

## Removing User Session Data

Secured Endpoint: DELETE /secured/sessions

This service can be used to remove saved user session information. This is helpful in cases where the user's session is in an unusable state and saving the session information keeps all of the user's future sessions in an unusable state.

Here's an example:

```
$ curl -XDELETE -H "$AUTH_HEADER" "http://by-tor:8888/secured/sessions"
```

Check the HTTP status of the response to tell if it succeeded. It should return a status in the 200 range.

An attempt to remove session data that doesn't already exist will be silently ignored and return a 200 range HTTP status code.

## Saving User Preferences

Secured Endpoint: POST /secured/preferences

This service can be used to save arbitrary user preferences. The body must contain all of the preferences for the user; any key-value pairs that are missing will be removed from the preferences. Please note that the "defaultOutputDir" and the "systemDefaultOutputDir" will always be present, even if not included in the JSON passed in.

Example:

```
$ curl -sH "$AUTH_HEADER" -d '{"appsKBShortcut":"A","rememberLastPath":true,"closeKBShortcut":"Q","defaultOutputFolder":{"id":"/iplant/home/wregglej/analyses","path":"/iplant/home/wregglej/analyses"},"dataKBShortcut":"D","systemDefaultOutputDir":{"id":"/iplant/home/wregglej/analyses","path":"/iplant/home/wregglej/analyses"},"saveSession":true,"enableEmailNotification":true,"lastPathId":"/iplant/home/wregglej","notificationKBShortcut":"N","defaultFileSelectorPath":"/iplant/home/wregglej","analysisKBShortcut":"Y"}' "http://by-tor:8888/secured/preferences" | squiggles
{
    "preferences": {
        "analysisKBShortcut": "Y",
        "appsKBShortcut": "A",
        "closeKBShortcut": "Q",
        "dataKBShortcut": "D",
        "defaultFileSelectorPath": "/iplant/home/wregglej",
        "defaultOutputFolder": {
            "id": "/iplant/home/wregglej/analyses",
            "path": "/iplant/home/wregglej/analyses"
        },
        "enableEmailNotification": true,
        "lastPathId": "/iplant/home/wregglej",
        "notificationKBShortcut": "N",
        "rememberLastPath": true,
        "saveSession": true,
        "systemDefaultOutputDir": {
            "id": "/iplant/home/wregglej/analyses",
            "path": "/iplant/home/wregglej/analyses"
        }
    }
}
```

## Retrieving User Preferences

Secured Endpoint: GET /secured/preferences

This service can be used to retrieve a user's preferences.

Example:

```
$ curl -sH "$AUTH_HEADER" "http://by-tor:8888/secured/preferences" | squiggles
{
    "analysisKBShortcut": "Y",
    "appsKBShortcut": "A",
    "closeKBShortcut": "Q",
    "dataKBShortcut": "D",
    "defaultFileSelectorPath": "/iplant/home/test",
    "defaultOutputFolder": {
        "id": "/iplant/home/test/analyses",
        "path": "/iplant/home/test/analyses"
    },
    "enableEmailNotification": true,
    "lastPathId": "/iplant/home/test",
    "notificationKBShortcut": "N",
    "rememberLastPath": true,
    "saveSession": true,
    "systemDefaultOutputDir": {
        "id": "/iplant/home/test/analyses",
        "path": "/iplant/home/test/analyses"
    }
}
```

## Removing User Preferences

Secured Endpoint: DELETE /secured/preferences

This service can be used to remove a user's preferences.

Please note that the "defaultOutputDir" and the "systemDefaultOutputDir" will still be present in the preferences after a deletion.

Example:

```
$ curl -X DELETE -H "$AUTH_HEADER" "http://by-tor:8888/secured/preferences"
```

Check the HTTP status code of the response to determine success. It should be in the 200 range.

An attempt to remove preference data that doesn't already exist will be silently ignored.

## Obtaining Identifiers

Unsecured Endpoint: GET /uuid

In some cases, it's difficult for the UI client code to generate UUIDs for objects that require them. This service returns a single UUID in the response body. The UUID is returned as a plain text string.

## Submitting User Feedback

Secured Endpoint: PUT /secured/feedback

This endpoint submits feedback from the user to a configurable iPlant email address. The destination email address is stored in the configuration settting, `terrain.email.feedback-dest`. The request body is a simple JSON object with the question text in the keys and the answer or answers in the values. The answers can either be strings or lists of strings:

```json
{
    "question 1": "question 1 answer 1",
    "question 2": [
        "question 2 answer 1",
        "question 2 answer 2"
    ]
}
```

Here's an example:

```
$ curl -XPUT -sH "$AUTH_HEADER" "http://by-tor:8888/secured/feedback" -d '
{
    "What is the circumference of the Earth?": "Roughly 25000 miles.",
    "What are your favorite programming languages?": [ "Clojure", "Scala", "Perl" ]
}
'
```

## Saved Searches

The saved-search endpoint proxies requests to the saved-searches service. This endpoint is used to store, retrieve, and delete a user's saved searches.


### Getting saved searches

Secured Endpoint: GET /secured/saved-searches

Curl example:

     curl -H "$AUTH_HEADER" http://localhost:31325/secured/saved-searches

The response body will be JSON. The service endpoint doesn't have a particular JSON structure it looks for, it simply stores whatever JSON is passed to it.

Possible error codes: ERR_BAD_REQUEST, ERR_NOT_A_USER, ERR_UNCHECKED_EXCEPTION

### Setting saved searches

Secured Endpoint: POST /secured/saved-searches

Curl example:

     curl -H "$AUTH_HEADER" -d '{"foo":"bar"}' http://localhost:31325/secured/saved-searches

Response body:

```json
{
        "saved_searches" : {"foo":"bar"}
}
```

Possible error codes: ERR_BAD_REQUEST, ERR_NOT_A_USER, ERR_UNCHECKED_EXCEPTION

If you pass up invalid JSON, you'll get an error like the following:

   {"reason":"Cannot JSON encode object of class: class org.eclipse.jetty.server.HttpInput: org.eclipse.jetty.server.HttpInput@1cbeb264"}

### Deleting saved searches

Secured endpoint: DELETE /secured/saved-searches

Curl example:

     curl -H "$AUTH_HEADER" -X DELETE http://localhost:31325/secured/saved-searches
