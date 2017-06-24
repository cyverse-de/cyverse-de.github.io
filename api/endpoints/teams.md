---
layout: page
title: DE API Documentation
---

# Table of Contents

* [Team Management Endpoints](#team-management-endpoints)
    * [Listing Teams](#listing-teams)
    * [Creating a New Team](#creating-a-new-team)
    * [Getting Team Information](#getting-team-information)
    * [Updating a Team](#updating-a-team)
    * [Deleting a Team](#deleting-a-team)
    * [Listing Team Members](#listing-team-members)
    * [Adding Team Members](#adding-team-members)
    * [Removing Team Members](#removing-team-members)
    * [Listing Team Privileges](#listing-team-privileges)
    * [Updating Team Privileges](#updating-team-privileges)


# Team Management Endpoints

## Listing Teams

Secured Endpoint: GET /teams

Query Parameters:

|| Name || Type || Description ||
| search | string | Searches for teams by name. |
| creator | string | Displays only teams created by the given user. |
| member | string | Displays only teams for which the given subject is a member. |

Listing all teams:

```
$ curl -sH "$AUTH_HEADER" "http://localhost:8888/teams" | jq .
{
  "groups": [
    {
      "name": "dennis:aqua-team",
      "type": "group",
      "description": "mah team description",
      "display_extension": "aqua-team",
      "display_name": "iplant:de:de-2:teams:dennis:aqua-team",
      "extension": "aqua-team",
      "id_index": "11104",
      "id": "230705b9ee8a40fa80c12bc2a1ba2efc"
    },
    ...
  ]
}
```

Listing teams created by a user:

```
$ curl -sH "$AUTH_HEADER" "http://localhost:8888/teams?creator=ipcdev" | jq .
{
  "groups": [
    {
      "name": "ipcdev:aqua-team",
      "type": "group",
      "description": "mah team description",
      "display_extension": "aqua-team",
      "display_name": "iplant:de:de-2:teams:ipcdev:aqua-team",
      "extension": "aqua-team",
      "id_index": "11122",
      "id": "e7bdfacc9dc3433c82763a5949385405"
    },
    ...
  ]
}
```

Listing teams that a user belongs to:

```
$ curl -sH "$AUTH_HEADER" "http://localhost:8888/teams?member=dennis" | jq .
{
  "groups": [
    {
      "name": "ipcdev:aqua-team",
      "type": "group",
      "description": "mah team description",
      "display_extension": "aqua-team",
      "display_name": "iplant:de:de-2:teams:ipcdev:aqua-team",
      "extension": "aqua-team",
      "id_index": "11122",
      "id": "e7bdfacc9dc3433c82763a5949385405"
    },
    ...
  ]
}
```

Searching for a team by name:

```
$ curl -sH "$AUTH_HEADER" "http://localhost:8888/teams?search=mahteam" | jq .
{
  "groups": [
    {
      "name": "dennis:mahteam",
      "type": "group",
      "description": "mah team description",
      "display_extension": "mahteam",
      "display_name": "iplant:de:de-2:teams:dennis:mahteam",
      "extension": "mahteam",
      "id_index": "11101",
      "id": "e6d382ba059a408fb34a4d9189966008"
    }
  ]
}
```

## Creating a New Team

Secured Endpoint: POST /teams

Request Body:

|| Field || Description ||
| name | The new team name without the username prefix. |
| description | A brief description of the team. |
| public_privileges | A list of privileges to grant to all DE users. |

A special note on the `name` field. The name of the team is automatically namespaced to the authenticated user. For
example, if the username is `foo` and the team name is `bar` then the full team name will be `foo:bar`.

The public privileges are standard Grouper privilege names. Some common values for this field are:

|| Privilege || Description ||
| view | All users can see the team, but not its members. |
| optin | All users can see the team and join the team without requiring administrator approval. Implies `view`. |
| read | All users can see the team and its members. Implies `view`. |

Example:

```
$ curl -sH "$AUTH_HEADER" -H "Content-Type: application/json" -d '{
  "name": "some-team",
  "description": "some team description",
  "public_privileges": ["view"]
}' "http://localhost:8888/teams" | jq .
{
  "description": "some team description",
  "name": "dennis:some-team",
  "type": "group",
  "extension": "some-team",
  "id": "93559b42f21a4423a4eaf322ea4dba97",
  "display_extension": "some-team",
  "display_name": "iplant:de:de-2:teams:dennis:some-team",
  "id_index": "11124"
}
```

## Getting Team Information

Secured Endpoint: GET /teams/{name}

Displays information about a specific team:

```
$ curl -sH "$AUTH_HEADER" "http://localhost:8888/teams/dennis%3Asome-team" | jq .
{
  "description": "some team description",
  "name": "dennis:some-team",
  "type": "group",
  "extension": "some-team",
  "id": "93559b42f21a4423a4eaf322ea4dba97",
  "display_extension": "some-team",
  "display_name": "iplant:de:de-2:teams:dennis:some-team",
  "id_index": "11124"
}
```

## Updating a Team

Secured Endpoint: PATCH /teams/{name}

Updates either the name or description of a team:

```
$ curl -sX PATCH -H "$AUTH_HEADER" -H 'Content-Type: application/json' -d '{
  "name": "some-other-team",
  "description": "some other team description"
}' "http://localhost:8888/teams/dennis%3Asome-team" | jq .
{
  "description": "some other team description",
  "name": "dennis:some-other-team",
  "type": "group",
  "extension": "some-other-team",
  "id": "93559b42f21a4423a4eaf322ea4dba97",
  "display_extension": "some-other-team",
  "display_name": "iplant:de:de-2:teams:dennis:some-other-team",
  "id_index": "11124"
}
```

One thing to note is the difference between the name in the request body and the names in the URI and the response
body. The former is just the last portion of the team name, `some-other-team`. The latter both contain the username
prefix, `dennis:some-other-team` (URL encoded in the URI). This helps to prevent teams from being renamed into another
user's team folder, which would be a potential source of confusion.

## Deleting a Team

Secured Endpoint: DELETE /teams/{name}

Deletes a team.

```
$ curl -sX DELETE -H "$AUTH_HEADER" "http://localhost:8888/teams/dennis%3Asome-other-team" | jq .
{
  "description": "some other team description",
  "display_extension": "some-other-team",
  "display_name": "iplant:de:de-2:teams:dennis:some-other-team",
  "extension": "some-other-team",
  "name": "dennis:some-other-team",
  "type": "group",
  "id": "93559b42f21a4423a4eaf322ea4dba97",
  "id_index": "11124"
}
```

## Listing Team Members

Secured Endpoint: GET /teams/{name}/members

Lists the members of a team:

```
$ curl -sH "$AUTH_HEADER" "http://localhost:8888/teams/dennis%3Aaqua-team/members" | jq .
{
  "members": [
    {
      "id": "ipctest",
      "name": "Ipc Test",
      "first_name": "Ipc",
      "last_name": "Test",
      "email": "ipctest@iplantcollaborative.org",
      "institution": "iplant collaborative",
      "source_id": "ldap"
    }
  ]
}
```

## Adding Team Members

Secured Endpoint: POST /teams/{name}/members

Adds members to a team:

```
$ curl -sH "$AUTH_HEADER" -d '{"members":["dennis"]}' "http://localhost:8888/teams/dennis%3Aaqua-team/members" | jq .
{
  "results": [
    {
      "success": true,
      "subject_id": "dennis",
      "subject_name": "Dennis Roberts"
    }
  ]
}
```

## Removing Team Members

Secured Endpoint: POST /teams/{name}/members/deleter

Removes members from a team:

```
$ curl -sH "$AUTH_HEADER" -d '{"members":["dennis"]}' "http://localhost:8888/teams/dennis%3Aaqua-team/members/deleter" | jq .
{
  "results": [
    {
      "success": true,
      "subject_id": "dennis",
      "subject_name": "Dennis Roberts"
    }
  ]
}
```

## Listing Team Privileges

Secured Endpoint: GET /teams/{name}/privileges

Lists team privileges:

```
$ curl -sH "$AUTH_HEADER" "http://localhost:8888/teams/dennis%3Aaqua-team/privileges" | jq .
{
  "privileges": [
    {
      "type": "access",
      "name": "view",
      "allowed": true,
      "revokable": false,
      "subject": {
        "id": "GrouperAll",
        "name": "EveryEntity",
        "source_id": "g:isa"
      }
    },
    {
      "type": "access",
      "name": "admin",
      "allowed": true,
      "revokable": true,
      "subject": {
        "id": "dennis",
        "name": "Dennis Roberts",
        "first_name": "Dennis",
        "last_name": "Roberts",
        "email": "dennis@cyverse.org",
        "institution": "The University of Arizona",
        "source_id": "ldap"
      }
    }
  ]
}
```

## Updating Team Privileges

Secured Endpoint: POST /teams/{name}/privileges

Updates team privileges for one or more subjects:

```
$ curl -sH "$AUTH_HEADER" -d '{"updates":[{"subject_id":"ipcdev","privileges":["admin", "optin"]}]}' \
    "http://localhost:8888/teams/dennis%3Aaqua-team/privileges" | jq .
{
  "privileges": [
    {
      "type": "access",
      "name": "view",
      "allowed": true,
      "revokable": false,
      "subject": {
        "id": "GrouperAll",
        "name": "EveryEntity",
        "source_id": "g:isa"
      }
    },
    {
      "type": "access",
      "name": "admin",
      "allowed": true,
      "revokable": true,
      "subject": {
        "id": "dennis",
        "name": "Dennis Roberts",
        "first_name": "Dennis",
        "last_name": "Roberts",
        "email": "dennis@cyverse.org",
        "institution": "The University of Arizona",
        "source_id": "ldap"
      }
    },
    {
      "type": "access",
      "name": "admin",
      "allowed": true,
      "revokable": true,
      "subject": {
        "id": "ipcdev",
        "name": "Ipc Dev",
        "first_name": "Ipc",
        "last_name": "Dev",
        "email": "core-sw@iplantcollaborative.org",
        "institution": "iPlant Collaborative",
        "source_id": "ldap"
      }
    },
    {
      "type": "access",
      "name": "optin",
      "allowed": true,
      "revokable": true,
      "subject": {
        "id": "ipcdev",
        "name": "Ipc Dev",
        "first_name": "Ipc",
        "last_name": "Dev",
        "email": "core-sw@iplantcollaborative.org",
        "institution": "iPlant Collaborative",
        "source_id": "ldap"
      }
    }
  ]
}
```

Specify an empty list of privileges to remove all privileges for a user:

```
$ curl -sH "$AUTH_HEADER" -d '{"updates":[{"subject_id":"ipcdev","privileges":[]}]}' \
    "http://localhost:8888/teams/dennis%3Aaqua-team/privileges" | jq .
{
  "privileges": [
    {
      "type": "access",
      "name": "view",
      "allowed": true,
      "revokable": false,
      "subject": {
        "id": "GrouperAll",
        "name": "EveryEntity",
        "source_id": "g:isa"
      }
    },
    {
      "type": "access",
      "name": "admin",
      "allowed": true,
      "revokable": true,
      "subject": {
        "id": "dennis",
        "name": "Dennis Roberts",
        "first_name": "Dennis",
        "last_name": "Roberts",
        "email": "dennis@cyverse.org",
        "institution": "The University of Arizona",
        "source_id": "ldap"
      }
    }
  ]
}
```
