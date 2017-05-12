---
layout: page
title: DE API Documentation
---

# Table of Contents

* [Subject Endpoints](#subject-endpoints)
    * [Search for Subjects](#search-for-subjects)

# Subject Endpoints

## Search for Subjects

Secured Endpoint: GET /subjects

This endpoint returns a list of subjects matching a search term:

```
$ curl -sH "$AUTH_HEADER" "http://example.org/subjects?search=foo" | jq
{
  "subjects": [
    {
      "id": "ac0868a8835d4e73b7079754777def59",
      "name": "iplant:de:de-2:users:dennis:collaborator-lists:foo",
      "source_id": "g:gsa"
    },
    {
      "id": "someone1234",
      "name": "Someone Foo",
      "first_name": "Someone",
      "last_name": "Foo",
      "email": "somefoo@example.org",
      "source_id": "ldap"
    },
    ...
  ]
}
```
