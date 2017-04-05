---
layout: page
title: DE API Documentation
---

# File IO

Provides a REST-like API for uploading and downloading files to and from iRODS.


## Error handling

If you try to hit an endpoint that doesn't exist, you'll get a 404.

For all other errors, you should receive a 500 HTTP status code and a JSON body in the following format:

```json
    {
        "error_code" : "<Scruffian error code>"
    }
```

Most errors will return other contextual fields, but they will vary from error to error. For programmatic usage, only depend on the three fields listed above.

Each section listed below lists the error codes that you may encounter. In addition to these, you may run into the ERR_UNCHECKED_EXCEPTION, which means that an uncaught exception was encountered.


## Downloading

__URL Path__: /secured/fileio/download

__URL Path__: /secured/filesystem/display-download

__HTTP Method__: GET

__Request Query Parameters__:

* path - The path to the file in iRODS to be downloaded.

__Error Codes__:

+ ERR_INVALID_JSON (wrong content-type or JSON syntax errors)
+ ERR_BAD_OR_MISSING_FIELD (JSON field is missing or has an invalid value)
+ ERR_MISSING_QUERY_PARAMETER (Query parameter is missing)
+ ERR_NOT_A_USER (invalid user specified)
+ ERR_DOES_NOT_EXIST (File request doesn't exist)
+ ERR_NOT_READABLE (File requested isn't readable by the specified user)

__Curl Command__:

    curl -H "$AUTH_HEADER" 'http://127.0.0.1:31370/secured/fileio/download?path=/iplant/home/testuser/myfile.txt'

This will result is the file contents being printed out to stdout. Redirect to a file to actually get the file.

When using fileio/download, it's always downloaded as an attachment and the content-type is always application/octet-stream. When using display-download, attachment is a parameter that can be passed along.

These endpoints delegate to data-info's GET /data/path/:zone/:path endpoint, with some processing.

## Uploading

__URL Path__: /secured/fileio/upload

__HTTP Method__: POST

__Request Query Parameters__:

* dest - The path to the iRODS folder where the file(s) will be uploaded.

__Request Form Fields__:

* file - The contents of the file to be uploaded.

__Error Codes__:

+ ERR_MISSING_FORM_FIELD (One of the form data fields is missing)
+ ERR_MISSING_QUERY_PARAMETER (Query parameter is missing)
+ ERR_NOT_A_USER (Invalid user specified)
+ ERR_DOES_NOT_EXIST (Destination directory doesn't exist)
+ ERR_NOT_WRITEABLE (Destination directory isn't writeable)

__Response Body__:

A success will return JSON like this:

```json
{
    "file": {
        "id": "<path to the file>",
        "path": "<path to the file>",
        "label": "<basename of the file path>",
        "permission": "own",
        "date-created": <seconds since the epoch>,
        "date-modified": <seconds since the epoch>,
        "file-size": <size in bytes>
    }
}
```

__Curl Command__:

Uploading is handled through multipart requests:

    curl -H "$AUTH_HEADER" -F file=@testfile.txt "localhost:31325/secured/fileio/upload?dest=/iplant/home/testuser"

Notice that the `dest` value points to a directory and not a file.

This endpoint delegates to data-info's POST /data endpoint.

## URL Uploads

__URL Path__: /secured/fileio/urlupload

__HTTP Method__: POST

__Error codes__:

+ ERR_INVALID_JSON (Missing content-type or JSON syntax error)
+ ERR_BAD_OR_MISSING_FIELD (Missing JSON field or invalid JSON field value)
+ ERR_MISSING_QUERY_PARAMETER (One of the query parameters is missing)
+ ERR_NOT_A_USER (Invalid user specified)
+ ERR_NOT_WRITEABLE (Destination directory isn't writeable by the specified user)
+ ERR_EXISTS (Destination file already exists)
+ ERR_REQUEST_FAILED (General failure to spawn upload thread)
+ ERR_INVALID_URL (URL that was passed in couldn't be used)

__Request Query Parameters__:

__Request Body__:

    {
        "dest" : "/iplant/home/testuser/",
        "address" : "http://www.google.com/index.html"
    }

__Response Body__:

On success you should get JSON that looks like this:

    {
        "msg" : "Upload scheduled.",
        "url" : "<URL>",
        "label" : "<URL base filename>",
        "dest" : "<destination in irods>"
    }

On on error, you'll either get a stacktrace or JSON that looks like this:

    {
        "msg" : "<JSON passed in through the request>",
        "error_code" : "ERR_REQUEST_FAILED"
    }

If the URL passed in is incorrect, then the error message will look like this:

    {
        "error_code" : "ERR_INVALID_URL",
        "url" : "<URL Passed in>"
    }

__Curl Command__:

    curl -H "$AUTH_HEADER" -d '{"dest" : "/iplant/home/testuser/", "address" : "http://www.google.com/index.html"}' http://127.0.0.1:31370/secured/fileio/urlupload

The 'dest' value in the JSON refers to the path to the directory in iRODS that the file will be saved off to. The filename of the file will be extracted from the path portion of the URL.


## Save

__URL Path__: /secured/fileio/save

__HTTP Method__: POST

__Error Codes__:

+ ERR_INVALID_JSON (Missing content-type or JSON syntax error)
+ ERR_BAD_OR_MISSING_FIELD (Missing JSON field or invalid JSON field value)
+ ERR_NOT_A_USER (Invalid user specified)
+ ERR_DOES_NOT_EXIST (The destination directory does not exist)
+ ERR_NOT_WRITEABLE (The destination directory is not writable by the user)
+ ERR_FILE_SIZE_TOO_LARGE (The size of the "content" field is larger than the terrain.fileio.max-edit-file-size config setting)

__Request Body__:

```json
    {
        "content" : "This is the content for the file.",
        "dest" : "/iplant/home/testuser/savedfile.txt"
    }
```

__Response Body__:

```json
{
    "file": {
        "id": "<path to the file>",
        "path": "<path to the file>",
        "label": "<basename of the file path>",
        "permission": "own",
        "date-created": <seconds since the epoch>,
        "date-modified": <seconds since the epoch>,
        "file-size": <size in bytes>
    }
}
```

__Curl Command__:

    curl -H "$AUTH_HEADER" -d '{"dest" : "/iplant/home/testuser/savedfile.txt", "content" : "This is the content for the file."}' 'http://127.0.0.1:31370/secured/fileio/save'


## Save As

__URL Path__: /secured/fileio/saveas

__HTTP Method__: POST

__Error Codes__:

+ ERR_INVALID_JSON (Missing content-type or JSON syntax error)
+ ERR_BAD_OR_MISSING_FIELD (Missing JSON field or invalid JSON field value)
+ ERR_NOT_A_USER (Invalid user specified)
+ ERR_DOES_NOT_EXIST (The destination directory does not exist)
+ ERR_NOT_WRITEABLE (The destination directory is not writable by the user)
+ ERR_EXISTS (The destination file already exists)

__Request Body__:

```json
    {
        "content" : "This is the content for the file.",
        "dest" : "/iplant/home/testuser/savedfile.txt"
    }
```

__Response Body__:

```json
{
    "file": {
        "id": "<path to the file>",
        "path": "<path to the file>",
        "label": "<basename of the file path>",
        "permission": "own",
        "date-created": <seconds since the epoch>,
        "date-modified": <seconds since the epoch>,
        "file-size": <size in bytes>
    }
}
```

__Curl Command__:

    curl -H "$AUTH_HEADER" -d '{"content" : "This is the content for the file.", "dest" : "/iplant/home/testuser/savedfile.txt"}' 'http://127.0.0.1:31370/secured/fileio/saveas'

This endpoint delegates to data-info's POST /data endpoint, similarly to the upload endpoint, with some processing.
