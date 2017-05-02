---
layout: page
title: DE API Documentation
---

# Table of Contents

    * [Listing and Searching for Tools](#listing-and-searching-for-tools)
    * [Get a Tool by ID](#get-a-tool-by-id)
    * [Importing Tools](#importing-tools)
    * [Deleting Tools](#deleting-tools)
    * [Updating Tools](#updating-tools)

## Listing and Searching for Tools

Secured Endpoint: GET /tools

Delegates to apps: GET /tools

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Get a Tool by ID

Secured Endpoint: GET /tools/{tool-id}

Delegates to apps: GET /tools/{tool-id}

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Importing Tools

Secured Endpoint: POST /admin/tools

Delegates to apps: POST /admin/tools

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Deleting Tools

Secured Endpoint: DELETE /admin/tools/{tool-id}

Delegates to apps: DELETE /admin/tools/{tool-id}

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Updating Tools

Secured Endpoint: PATCH /admin/tools/{tool-id}

Delegates to apps: PATCH /admin/tools/{tool-id}

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.
