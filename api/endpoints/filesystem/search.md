---
layout: page
title: DE API Documentation
---

This document describes the endpoints used to performing searches of user data.

# Table of Contents

* [Indexed Information](#indexed-information)
* [Basic Usage](#basic-usage)
    * [Search Requests](#search-requests)

# Indexed Information

For each file and folder stored in the iPlant data store, its ACL, system metadata, and user metadata are indexed as a JSON document. For files, [file records](../../schema.html#file-record) are indexed, and for folders, [folder records](../../schema.html#folder-record) are indexed.

In addition, CyVerse metadata (non-iRODS metadata) is indexed in a child document, and tags are indexed separately in their own way as well.

# Basic Usage

For the client without administrative privileges, Terrain provides endpoints for performing searches and for checking the status of the indexer.

## Search Requests

Terrain provides search endpoints that allow callers to search the data by name and various pieces of system and user metadata. It supports the all the queries in the [ElasticSearch query DSL] (http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl-queries.html) for searching.

Each field in an indexed document may be explicitly used in a search query. If the field is an object, i.e. an aggregate of fields, the object's fields may be explicitly referenced as well using dot notation, e.g. `acl.access`.

### Endpoints

* `GET /secured/filesystem/index`

This endpoint searches the data index and retrieves a set of files and/or folders matching the terms provided in the query string.

### Request Parameters

The following additional URI parameters are recognized.

| Parameter    | Required? | Default    | Description |
| ------------ | --------- | ---------- | ----------- |
| q            | yes*      |            | This parameter holds a JSON encoded search query. See [query syntax](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl-queries.html) for a description of the syntax. |
| type         | no        | any        | This parameter restricts the search to either files or folders. It can take the values `any`, meaning files and folders, `file`, only files, and `folders`, only folders. |
| tags         | yes*      |            | This is a comma-separated list of tag UUIDs. This parameter restricts the search to only entities that have at least of the provided tags. |
| includeTrash | no        | `false`    | This parameter allows the result set to contain matching files and folders from the user's DE trash. A value `true` allows trash, and a value of `false` excludes it. |
| offset       | no        | 0          | This parameter indicates the number of matches to skip before including any in the result set. When combined with `limit`, it allows for paging results. |
| limit        | no        | 200        | This parameter limits the number of matches in the result set to be a most a certain amount. When combined with `offset`, it allows for paging results. |
| sort         | no        | score:desc | See [sorting](#sorting) |

\* At least `q` or `tags` is required.

#### Sorting

The result set is sorted. By default the sort is performed on the `score` field in descending order. The `sort` request parameter can be used to change the sort order. Its value has the form `field:direction` where `field` is one of the fields of a **match record** and `direction` is either `asc` for ascending or `desc` for descending. Use dot notation to sort by one of the nested fields of the match records, e.g., `entity.label` will sort by the `label` field of the matched entities. If no `:direction` is provided, the search direction will default to `desc`.

### Response

When the search succeeds the response document has these additional fields.

| Field          | Type   | Description |
| -------------- | ------ | ----------- |
| total          | number | This is the total number of matches found, not the number of elements in the `matches` array. |
| offset         | number | This is the value of the `offset` parameter in the query string. |
| matches        | array  | This is the set or partial set of matches found, each entry being a **match record**. It contains at most `limit` entries and is sorted by descending score. |
| execution-time | number | This is the number of milliseconds that it took to perform the query and get a response from elasticsearch. |

**Match Record**

| Field  | Type   | Description |
| ------ | ------ | ----------- |
| score  | number | an indication of how well this entity matches the query compared to other matches |
| type   | string | the entity is this type of filesystem entry, either `"file"` or `"folder"` |
| entity | object | the [file record](../../schema.html#file-record) or [folder record](../../schema.html#folder-record) matched |

### Example

```
$ curl -H "$AUTH_HEADER" \
> "http://localhost:8888/secured/filesystem/index?q=\\{\"wildcard\":\\{\"label\":\"?e*\"\\}\\}&type=file&tags=64581dbe-3aa1-11e4-a54e-3c4a92e4a804,6504ca14-3aa1-11e4-8d83-3c4a92e4a804&offset=1&limit=2&sort:desc" \
> | python -mjson.tool
{
    "matches": [
        {
            "entity": {
                "creator": "rods#iplant",
                "dateCreated": 1381325224090,
                "dateModified": 1384998366001,
                "fileSize": 13225,
                "id": "6278f8e0-121f-4ce6-a15f-1083cfad6de5",
                "label": "read1_10k.fq",
                "fileType": null,
                "metadata": [],
                "path": "/iplant/home/rods/analyses/fc_01300857-2013-10-09-13-27-04.090/read1_10k.fq",
                "permission" : "own",
                "userPermissions": [
                    {
                        "permission": "read",
                        "user": "rods#iplant"
                    },
                    {
                        "permission": "own",
                        "user": "rodsadmin#iplant"
                    }
                ]
            },
            "score": 1.0,
            "type": "file"
        },
        {
            "entity": {
                "creator": "rods#iplant",
                "dateCreated": 1381325285602,
                "dateModified": 1384998366001,
                "fileSize": 14016,
                "fileType": null,
                "id": "acaeb63d-8e84-412d-89a2-716a6a4dda7e",
                "label": "read1_10k.fq",
                "metadata": [
                    {
                        "attribute": "color",
                        "unit": null,
                        "value": "red"
                    }
                ],
                "path": "/iplant/home/rods/analyses/ft_01251621-2013-10-09-13-28-05.602/read1_10k.fq",
                "permission": "write",
                "userPermissions": [
                    {
                        "permission": "read",
                        "user": "rods#iplant"
                    },
                    {
                        "permission": "write",
                        "user": "rodsadmin#iplant"
                    }
                ]
            },
            "score": 1.0,
            "type": "file"
        }
    ],
    "execution-time" : 300,
    "offset": 1,
    "total": 7
}
```
