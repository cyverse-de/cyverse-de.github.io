---
layout: page
title: DE API Documentation
---

# Table of Contents

* [Tool Endpoints](#tool-endpoints)
    * [Listing and Searching for Tools](#listing-and-searching-for-tools)
    * [Add a Private Tool](#add-a-private-tool)
    * [Listing Permissions for a Set of Tools](#listing-permissions-for-a-set-of-tools)
    * [Granting Access to a Set of Tools](#granting-access-to-a-set-of-tools)
    * [Revoking Access to a Set of Tools](#revoking-access-to-a-set-of-tools)
    * [Get a Tool by ID](#get-a-tool-by-id)
    * [Importing Tools](#importing-tools)
    * [Deleting Tools](#deleting-tools)
    * [Updating Tools](#updating-tools)
    * [Listing Apps by Tool ID](#listing-apps-by-tool-id)
    * [Making a Tool Public](#making-a-tool-public)

# Tool Endpoints

## Listing and Searching for Tools

Secured Endpoint: GET /tools

Delegates to apps: GET /tools

Secured Endpoint: GET /admin/tools

Delegates to apps: GET /admin/tools

These endpoints are passthroughs to the apps endpoints using the same path.
Please see the apps service documentation for more information.

## Add a Private Tool

Secured Endpoint: POST /tools

Delegates to apps: POST /tools

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Listing Permissions for a Set of Tools

Secured Endpoint: POST /tools/permission-lister

Delegates to apps: POST /tools/permission-lister

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Granting Access to a Set of Tools

Secured Endpoint: POST /tools/sharing

Delegates to apps: POST /tools/sharing

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Revoking Access to a Set of Tools

Secured Endpoint: POST /tools/unsharing

Delegates to apps: POST /tools/unsharing

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Get a Tool by ID

Secured Endpoint: GET /tools/{tool-id}

Delegates to apps: GET /tools/{tool-id}

Secured Endpoint: GET /admin/tools/{tool-id}

Delegates to apps: GET /admin/tools/{tool-id}

These endpoints are passthroughs to the apps endpoints using the same path.
Please see the apps service documentation for more information.

## Importing Tools

Secured Endpoint: POST /admin/tools

Delegates to apps: POST /admin/tools

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Deleting Tools

Secured Endpoint: DELETE /tools/{tool-id}

Delegates to apps: DELETE /tools/{tool-id}

Secured Endpoint: DELETE /admin/tools/{tool-id}

Delegates to apps: DELETE /admin/tools/{tool-id}

These endpoints are passthroughs to the apps endpoints using the same path.
Please see the apps service documentation for more information.

## Updating Tools

Secured Endpoint: PATCH /tools/{tool-id}

Delegates to apps: PATCH /tools/{tool-id}

Secured Endpoint: PATCH /admin/tools/{tool-id}

Delegates to apps: PATCH /admin/tools/{tool-id}

These endpoints are passthroughs to the apps endpoints using the same path.
Please see the apps service documentation for more information.

## Listing Apps by Tool ID

Secured Endpoint: GET /tools/{tool-id}/apps

Delegates to apps: GET /tools/{tool-id}/apps

Secured Endpoint: GET /admin/tools/{tool-id}/apps

Delegates to apps: GET /admin/tools/{tool-id}/apps

These endpoints are passthroughs to the apps endpoints using the same path.
Please see the apps service documentation for more information.

## Making a Tool Public

Secured Endpoint: POST /admin/tools/{tool-id}/publish

Delegates to apps: POST /admin/tools/{tool-id}/publish

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.
