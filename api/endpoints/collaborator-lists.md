---
layout: page
title: DE API Documentation
---

# Table of Contents

* [Collaborator List Management Endpoints](#collaborator-list-management-endpoints)
    * [Listing Collaborator Lists](#listing-collaborator-lists)
    * [Adding a Collaborator List](#adding-a-collaborator-list)
    * [Getting Information About a Collaborator List](#getting-information-about-a-collaborator-list)
    * [Updating a Collaborator List](#updating-a-collaborator-list)
    * [Deleting a Collaborator List](#deleting-a-collaborator-list)
    * [Listing Collaborator List Members](#listing-collaborator-list-members)
    * [Adding Collaborator List Members](#adding-collaborator-list-members)
    * [Removing Collaborator List Members](#removing-collaborator-list-members)

# Collaborator List Management Endpoints

## Listing Collaborator Lists

Secured Endpoint: GET /collaborator-lists

This endpoint returns collaborator lists that belong to the authenticated user.

```
$ curl -sH "$AUTH_HEADER" "http://example.org/collaborator-lists" | jq .
{
  "groups": [
    {
      "name": "bar",
      "type": "group",
      "description": "bar",
      "display_extension": "bar",
      "display_name": "iplant:de:de-2:users:dennis:collaborator-lists:bar",
      "extension": "bar",
      "id_index": "10293",
      "id": "f449ea254cdf480a80a7ad96d785389d"
    },
    {
      "name": "foo",
      "type": "group",
      "description": "foo",
      "display_extension": "foo",
      "display_name": "iplant:de:de-2:users:dennis:collaborator-lists:foo",
      "extension": "foo",
      "id_index": "10277",
      "id": "ac0868a8835d4e73b7079754777def59"
    }
  ]
}
```

The result can also be filtered by adding the `search` query parameter.

```
$ curl -sH "$AUTH_HEADER" "http://example.org/collaborator-lists?search=foo" | jq .
{
  "groups": [
    {
      "name": "foo",
      "type": "group",
      "description": "foo",
      "display_extension": "foo",
      "display_name": "iplant:de:de-2:users:dennis:collaborator-lists:foo",
      "extension": "foo",
      "id_index": "10277",
      "id": "ac0868a8835d4e73b7079754777def59"
    }
  ]
}
```

## Adding a Collaborator List

Secured Endpoint: POST /collaborator-lists

```
$ curl -sH "$AUTH_HEADER" -H "Content-Type: application/json" -d '{"name": "bar", "description": "bar"}' \
    "http://example.org/collaborator-lists" | jq .
{
  "description": "bar",
  "name": "bar",
  "type": "group",
  "extension": "bar",
  "id": "f449ea254cdf480a80a7ad96d785389d",
  "display_extension": "bar",
  "display_name": "iplant:de:de-2:users:dennis:collaborator-lists:bar",
  "id_index": "10293"
}
```

## Getting Information About a Collaborator List

Secured Endpoint: GET /collaborator-lists/{name}

```
$ curl -sH "$AUTH_HEADER" "http://example.org/collaborator-lists/foo" | jq .
{
  "description": "foo",
  "name": "foo",
  "type": "group",
  "extension": "foo",
  "id": "ac0868a8835d4e73b7079754777def59",
  "display_extension": "foo",
  "display_name": "iplant:de:de-2:users:dennis:collaborator-lists:foo",
  "id_index": "10277"
}
```

## Updating a Collaborator List

Secured Endpoint: PATCH /collaborator-lists/{name}

Two fields can currently be updated: `name` and `description`. Fields that aren't provided will remain unchanged.

```
$ curl -sH "$AUTH_HEADER" -H "Content-Type: application/json" -X PATCH -d '{"name": "baz", "description":"baz"}' \
    "http://example.org/collaborator-lists/foo" | jq .
{
  "description": "baz",
  "name": "baz",
  "type": "group",
  "extension": "baz",
  "id": "ac0868a8835d4e73b7079754777def59",
  "display_extension": "baz",
  "display_name": "iplant:de:de-2:users:dennis:collaborator-lists:baz",
  "id_index": "10277"
}
```

## Deleting a Collaborator List

Secured Endpoint: DELETE /collaborator-lists/{name}

```
$ curl -sX DELETE -H "$AUTH_HEADER" "http://example.org/collaborator-lists/bar" | jq .
{
  "description": "bar",
  "display_extension": "bar",
  "display_name": "iplant:de:de-2:users:dennis:collaborator-lists:bar",
  "extension": "bar",
  "name": "bar",
  "type": "group",
  "id": "f449ea254cdf480a80a7ad96d785389d",
  "id_index": "10293"
}
```

## Listing Collaborator List Members

Secured Endpoint: GET /collaborator-lists/{name}/members

```
$ curl -sH "$AUTH_HEADER" http://example.org/collaborator-lists/foo/members | jq .
{
  "members": [
    {
      "id": "ipcdev",
      "name": "Ipc Dev",
      "first_name": "Ipc",
      "last_name": "Dev",
      "email": "ipcdev@example.org",
      "institution": "Iplant collaborative",
      "source_id": "ldap"
    }
  ]
}
```

## Adding Collaborator List Members

Secured Endpoint: POST /collaborator-lists/{name}/members

```
$ curl -sH "$AUTH_HEADER" -d '{"members":["ipctest"]}' http://example.org/collaborator-lists/foo/members | jq .
{
  "results": [
    {
      "success": true,
      "subject_id": "ipctest",
      "subject_name": "IPC Test"
    }
  ]
}
```

## Removing Collaborator List Members

Secured Endpoint: POST /collaborator-lists/{name}/members/deleter

```
$ curl -sH "$AUTH_HEADER" -d '{"members":["ipctest"]}' http://exmaple.org/collaborator-lists/foo/members/deleter | jq .
{
  "results": [
    {
      "success": true,
      "subject_id": "ipctest",
      "subject_name": "IPC Test"
    }
  ]
}
```
