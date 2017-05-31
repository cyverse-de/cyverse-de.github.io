---
layout: page
title: DE API Documentation
---

# Table of Contents

* [Collaborator List Management Endpoints](#collaborator-list-management-endpoints)
    * [Listing Collaborators](#listing-collaborators)
    * [Adding Collaborators](#adding-collaborators)
    * [Removing Collaborators](#removing-collaborators)
    * [Searching for Users](#searching-for-users)
    * [Obtaining User Info](#obtaining-user-info)

# Collaborator List Management Endpoints

Note that secured endpoints in Terrain and apps are a little different from each other. Please see [Terrain Vs. Apps](terrain-v-apps.html) for more information.

## Listing Collaborators

Secured Endpoint: GET /secured/collaborators

This service delegates all of its calls to apps's GET /collaborators endpoint. Please refer to apps's documentation for more information.

## Adding Collaborators

Secured Endpoint: POST /secured/collaborators

This service delegates all of its calls to apps's POST /collaborators endpoint. Please refer to apps's documentation for more information.

## Removing Collaborators

Secured Endpoint: POST /secured/remove-collaborators

This service delegates all of its calls to apps's POST /collaborators/shredder endpoint. Please refer to apps's documentation for more information.

## Searching for Users

Secured Endpoint: GET /secured/user-search?search={search-string}

This endpoint allows the caller to search for user information by username, email address and actual name. The search search string provided in the URL should be URL encoded before being sent to the service. The response body is in the following format:

```json
{
    "users": [
        {
            "id": "id-1",
            "name": "name-1",
            "first_name": "first-name-1",
            "last_name": "last-name-1",
            "email": "email-address-1",
            "institution": "institution-1",
            "source_id": "source-id-1"
        },
        {
            "id": "id-n",
            "name": "name-n",
            "first_name": "first-name-n",
            "last_name": "last-name-n",
            "email": "email-address-n",
            "institution": "institution-n",
            "source_id": "source-id-n"
        }
    ],
    "truncated": false
}
```

Assuming an error doesn't occur, the status code will be 200 and the response body will contain up to the first fifty users whose username matched the search string, up to the first fifty users whose actual name matched the search string, and up to the first fifty users whose email address matched the search string. Here's an example:

```
$ curl -sH "$AUTH_HEADER" "http://by-tor:8888/secured/user-search?search=nobody" | python -mjson.tool
{
    "truncated": false,
    "users": [
        {
            "id": "nobody",
            "name": "Nobody Inparticular",
            "first_name": "Nobody",
            "last_name": "Inparticular",
            "email": "nobody@iplantcollaborative.org",
            "institution": "iplant collaborative",
            "source_id": "ldap"
        }
    ]
}
```

This endpoint delegates to iplant-groups' GET /subjects endpoint.

## Obtaining User Info

Secured Endpoint: GET /secured/user-info

This endpoint allows the caller to search for information about users with specific usernames. Each username is specified using the `username` query string parameter, which can be specified multiple times to search for information about more than one user. The response body is in the following format:

```json
{
    "username-1": {
        "id": "id-1",
        "name": "name-1",
        "first_name": "first-name-1",
        "last_name": "last-name-1",
        "email": "email-address-1",
        "institution": "institution-1",
        "source_id": "source-id-1"
    },
    "username-n": {
        "id": "id-n",
        "name": "name-n",
        "first_name": "first-name-n",
        "last_name": "last-name-n",
        "email": "email-address-n",
        "institution": "institution-n",
        "source_id": "source-id-n"
    }
}
```

Assuming the service doesn't encounter an error, the status code will be 200 and the response body will contain the information for all of the users who were found. If none of the users were found then the response body will consist of an empty JSON object.

Here's an example with a match:

```
$ curl -sH "$AUTH_HEADER" "http://by-tor:8888/secured/user-info?username=nobody" | python -mjson.tool
{
    "nobody": {
        "id": "nobody",
        "name": "Nobody Inparticular",
        "first_name": "Nobody",
        "last_name": "Inparticular",
        "email": "nobody@iplantcollaborative.org",
        "institution": "iplant collaborative",
        "source_id": "ldap"
    }
}
```

Here's an example with no matches:

```
$ curl -sH "$AUTH_HEADER" "http://by-tor:8888/secured/user-info?username=foo" | python -mjson.tool
{}
```

This endpoint delegates to iplant-groups' GET /subjects/:subject-id endpoint.
