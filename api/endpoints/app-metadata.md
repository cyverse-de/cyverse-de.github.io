---
layout: page
title: DE API Documentation
---

# Table of Contents

* [Application Metadata Endpoints](#application-metadata-endpoints)
    * [Listing App Elements](#listing-app-elements)
    * [Listing App Identifiers](#listing-app-identifiers)
    * [Adding Categories](#adding-categories)
    * [Deleting Categories](#deleting-categories)
    * [Deleting a Category by ID](#deleting-a-category-by-id)
    * [Updating an App Category](#updating-an-app-category)
    * [Managing App AVU Metadata](#managing-app-avu-metadata)
    * [Listing Tasks in an App](#listing-tasks-in-an-app)
    * [Categorizing Apps](#categorizing-apps)
    * [Creating an App for the Current User](#creating-an-app-for-the-current-user)
    * [Getting Analyses in the JSON Format Required by the DE](#getting-analyses-in-the-json-format-required-by-the-de)
    * [Getting App Details](#getting-app-details)
    * [App Documentation](#app-documentation)
    * [Listing App Categories](#listing-app-categories)
    * [Exporting an Analysis](#exporting-an-analysis)
    * [Permanently Deleting an App](#permanently-deleting-an-app)
    * [Logically Deleting Apps](#logically-deleting-apps)
    * [Updating a Single-Step App](#updating-a-single-step-app)
    * [Creating a Pipeline](#creating-a-pipeline)
    * [Updating a Pipeline](#updating-a-pipeline)
    * [Updating App Labels](#updating-app-labels)
    * [Rating Apps](#rating-apps)
    * [Deleting App Ratings](#deleting-app-ratings)
    * [Searching for Apps](#searching-for-apps)
    * [Previewing Command Line Arguments](#previewing-command-line-arguments)
    * [Listing Apps in an App Group](#listing-apps-in-an-app-group)
    * [Listing Tools in an App](#listing-tools-in-an-app)
    * [Adding an App Favorite](#adding-an-app-favorite)
    * [Removing an App Favorite](#removing-an-app-favorite)
    * [Making a Copy of an App Available for Editing](#making-a-copy-of-an-app-available-for-editing)
    * [Submitting an App for Public Use](#submitting-an-app-for-public-use)
    * [Determining if an App Can be Made Public](#determining-if-an-app-can-be-made-public)
    * [Obtaining an App Representation for Editing](#obtaining-an-app-representation-for-editing)
    * [Making a Pipeline Available for Editing](#making-a-pipeline-available-for-editing)
    * [Making a Copy of a Pipeline Available for Editing](#making-a-copy-of-a-pipeline-available-for-editing)
    * [Requesting Installation of a Tool](#requesting-installation-of-a-tool)
    * [Updating a Tool Installation Request](#updating-a-tool-installation-request)
    * [Listing Tool Installation Requests](#listing-tool-installation-requests)
    * [Listing Tool Installation Request Details](#listing-tool-installation-request-details)
    * [Listing Tool Request Status Codes](#listing-tool-request-status-codes)
    * [Listing Permissions for a Set of Apps](#listing-permissions-for-a-set-of-apps)
    * [Granting Access to a Set of Apps](#granting-access-to-a-set-of-apps)
    * [Revoking Access to a Set of Apps](#revoking-access-to-a-set-of-apps)

# Application Metadata Endpoints

Note that secured endpoints in Terrain and apps are a little different from each other.
Please see [Terrain Vs. Apps](terrain-v-apps.html) for more information.

## Listing App Elements

Secured Endpoint: GET /apps/elements

Delegates to apps: GET /apps/elements

Secured Endpoint: GET /apps/elements/{element-type}

Delegates to apps: GET /apps/elements/{element-type}

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Listing App Identifiers

Secured Endpoint: GET /apps/ids

Delegates to apps: GET /apps/ids

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Adding Categories

Secured Endpoint: POST /admin/apps/categories

Delegates to apps: POST /admin/apps/categories

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Deleting Categories

Secured Endpoint: POST /admin/apps/categories/shredder

Delegates to apps: POST /admin/apps/categories/shredder

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Deleting a Category by ID

Secured Endpoint: DELETE /admin/apps/categories/{category-id}

Delegates to apps: DELETE /admin/apps/categories/{category-id}

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Updating an App Category

Secured Endpoint: PATCH /admin/apps/categories/{category-id}

Delegates to apps: PATCH /admin/apps/categories/{category-id}

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Managing App AVU Metadata

Secured Endpoint: GET /admin/apps/{app-id}/metadata

Delegates to apps: GET /admin/apps/{app-id}/metadata

Secured Endpoint: POST /admin/apps/{app-id}/metadata

Delegates to apps: POST /admin/apps/{app-id}/metadata

Secured Endpoint: PUT /admin/apps/{app-id}/metadata

Delegates to apps: PUT /admin/apps/{app-id}/metadata

Secured Endpoint: GET /apps/{app-id}/metadata

Delegates to apps: GET /apps/{app-id}/metadata

Secured Endpoint: POST /apps/{app-id}/metadata

Delegates to apps: POST /apps/{app-id}/metadata

Secured Endpoint: PUT /apps/{app-id}/metadata

Delegates to apps: PUT /apps/{app-id}/metadata

These endpoints are passthroughs to the apps endpoints using the same paths.
Please see the corresponding service documentation for more information.

## Listing Tasks in an App

Secured Endpoint: GET /apps/{app-id}/tasks

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Categorizing Apps

Secured Endpoint: POST /admin/apps

Delegates to apps: POST /admin/apps

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Creating an App for the Current User

Secured Endpoint: POST /apps

Delegates to apps: POST /apps

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Getting Analyses in the JSON Format Required by the DE

Secured Endpoint: GET /apps/{app-id}

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Getting App Details

Secured Endpoint: GET /apps/{app-id}/details

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## App Documentation

Secured Endpoint: GET /apps/{app-id}/documentation

Delegates to apps: GET /apps/{app-id}/documentation

Secured Endpoint: PATCH /apps/{app-id}/documentation

Delegates to apps: PATCH /apps/{app-id}/documentation

Secured Endpoint: POST /apps/{app-id}/documentation

Delegates to apps: POST /apps/{app-id}/documentation

Secured Endpoint: PATCH /admin/apps/{app-id}/documentation

Delegates to apps: PATCH /admin/apps/{app-id}/documentation

Secured Endpoint: POST /admin/apps/{app-id}/documentation

Delegates to apps: POST /admin/apps/{app-id}/documentation

These endpoints are passthroughs to the apps endpoints using the same path.
Please see the apps service documentation for more information.

## Listing App Categories

Secured Endpoint: GET /apps/categories

Delegates to apps: GET /apps/categories

Secured Endpoint: GET /admin/apps/categories

Delegates to apps: GET /admin/apps/categories

These endpoints are passthroughs to the apps endpoints using the same path.
Please see the apps service documentation for more information.

## Exporting an Analysis

Unsecured Endpoint: GET /export-workflow/{analysis-id}

Delegates to apps: GET /export-workflow/{analysis-id}

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Permanently Deleting an App

Secured Endpoint: POST /admin/apps/shredder

Delegates to apps: POST /admin/apps/shredder

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Logically Deleting Apps

Secured Endpoint: DELETE /admin/apps/{app-id}

Delegates to apps: DELETE /admin/apps/{app-id}

Secured Endpoint: DELETE /apps/{app-id}

Delegates to apps: DELETE /apps/{app-id}

Secured Endpoint: POST /apps/shredder

Delegates to apps: POST /apps/shredder

These endpoints are passthroughs to their corresponding apps endpoint.
Please see the apps service documentation for more information.

## Updating a Single-Step App

*Secured Endpoint:* PUT /apps/{app-id}

*Delegates to apps:* PUT /apps/{app-id}

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Creating a Pipeline

Secured Endpoint: POST /apps/pipelines

Delegates to apps: POST /apps/pipelines

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Updating a Pipeline

Secured Endpoint: PUT /apps/pipelines/{app-id}

Delegates to apps: PUT /apps/pipelines/{app-id}

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Updating App Labels

Secured Endpoint: PATCH /apps/{app-id}

Delegates to apps: PATCH /apps/{app-id}

Secured Endpoint: PATCH /admin/apps/{app-id}

Delegates to apps: PATCH /apps/{app-id}

These endpoints are passthroughs to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Rating Apps

Secured Endpoint: POST /apps/{app-id}/rating

Delegates to apps: POST /apps/{app-id}/rating

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Deleting App Ratings

Secured Endpoint: DELETE /apps/{app-id}/rating

Delegates to apps: DELETE /apps/{app-id}/rating

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Searching for Apps

Secured Endpoint: GET /apps?search={term}

Delegates to apps: GET /apps?search={term}

Secured Endpoint: GET /admin/apps

Delegates to apps: GET /admin/apps

These endpoints are passthroughs to the apps endpoints using the same path.
Please see the apps service documentation for more information.

## Previewing Command Line Arguments

Unsecured Endpoint: POST /apps/arg-preview

Delegates to apps: POST /apps/arg-preview

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Listing Apps in an App Group

Secured Endpoint: GET /apps/categories/{group-id}

Delegates to apps: GET /apps/categories/{group-id}

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Listing Tools in an App

Secured Endpoint: GET /apps/{app-id}/tools

Delegates to apps: GET /apps/{app-id}/tools

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Adding an App Favorite

Secured Endpoint: PUT /apps/{app-id}/favorite

Delegates to apps: PUT /apps/{app-id}/favorite

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Removing an App Favorite

Secured Endpoint: DELETE /apps/{app-id}/favorite

Delegates to apps: DELETE /apps/{app-id}/favorite

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Making a Copy of an App Available for Editing

Secured Endpoint: POST /apps/{app-id}/copy

Delegates to apps: POST /apps/{app-id}/copy

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Submitting an App for Public Use

Secured Endpoint: POST /apps/:app-id/publish

Delegates to apps: POST /apps/:app-id/publish

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Determining if an App Can be Made Public

Secured Endpoint: GET /apps/{app-id}/is-publishable

Delegates to apps: GET /apps/{app-id}/is-publishable

This endpoint is a passthrough to the apps endpoint using the path above.
Please see the apps service documentation for more information.

## Obtaining an App Representation for Editing

Secured Endpoint: GET /apps/{app-id}/ui

Delegates to apps: GET /apps/{app-id}/ui

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Making a Pipeline Available for Editing

Secured Endpoint: GET /apps/pipelines/{app-id}/ui

Delegates to apps: GET /apps/pipelines/{app-id}/ui

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Making a Copy of a Pipeline Available for Editing

Secured Endpoint: POST /apps/pipelines/{app-id}/copy

Delegates to apps: POST /apps/pipelines/{app-id}/copy

This endpoint is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more information.

## Requesting Installation of a Tool

Secured Endpoint: POST /tool-requests

Delegates to apps: POST /tool-requests

This service is primarily a passthrough to the apps endpoint using the same path.
The only difference is that this endpoint also sends a message to the tool request email address
and generates a notification for the new tool request indicating that the tool request was successfully submitted.
Please see the apps documentation for more details.

## Updating a Tool Installation Request

Secured Endpoint: POST /admin/tool-requests/{tool-request-id}/status

Delegates to apps: POST /admin/tool-requests/{tool-request-id}/status

This service is primarily a passthrough to the apps endpoint using the same path.
The only difference is that this endpoint also generates a notification for the tool request status update.
Please see the apps service documentation for more details.

## Listing Tool Installation Requests

Secured Endpoint: GET /tool-requests

Delegates to apps: GET /tool-requests

Secured Endpoint: GET /admin/tool-requests

Delegates to apps: GET /admin/tool-requests

These services are passthroughs to the apps endpoints using the same path.
Please see the apps service documentation for more details.

## Listing Tool Installation Request Details

Secured Endpoint: GET /admin/tool-requests/{tool-request-id}

Delegates to apps: GET /admin/tool-requests/{tool-request-id}

This service is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more details.

## Listing Tool Request Status Codes

Secured Endpoint: GET /tool-requests/status-codes

This service is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more details.

## Listing Permissions for a Set of Apps

Secured Endpoint: POST /apps/permission-lister

This service is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more details.

## Granting Access to a Set of Apps

Secured Endpoint: POST /apps/share

This service is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more details.

## Revoking Access to a Set of Apps

Secured Endpoint: POST /apps/unshare

This service is a passthrough to the apps endpoint using the same path.
Please see the apps service documentation for more details.
