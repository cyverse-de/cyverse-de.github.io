---
layout: page
title: DE Permanent ID Request API Documentation
---

# Table of Contents

* [Permanent ID Requests](#permanent-id-requests)
    * [Workflow Overview](#workflow-overview)

# Permanent ID Requests

These endpoints create and manage user requests for persistent identifiers
(DataCite DOIs) for data the user wishes to make public.

The persistent identifiers will be created with the
[DataCite API](https://support.datacite.org/docs/api-create-dois)
and managed using [DataCite Fabrica](https://doi.datacite.org/).

## Workflow Overview

1. User reviews [DOI Request Quickstart page](https://learning.cyverse.org/projects/cyverse-doi-request-quickstart/en/latest/)
   and determines that a CyVerse DOI is appropriate for their data.
2. User organizes their data in a single folder per Permanent ID, according to very general CyVerse guidelines.
3. Based on what they learned from tutorial, user will complete DataCite metadata template on that folder.
4. User [creates a Permanent ID Request](https://de.cyverse.org/terrain/docs/index.html#!/permanent45id45requests/post_terrain_permanent_id_requests)
   for this folder.
    * Request must be for one of the [available Permanent ID Request types](https://de.cyverse.org/terrain/docs/index.html#!/permanent45id45requests/get_terrain_permanent_id_requests_types),
    however the [DataCite API](https://support.datacite.org/docs/api-create-dois)
    only supports creating new DOIs.
    * Request triggers the validation check within the DE of the metadata and folder name
      (must not conflict with any folder names in the data commons repo `staging` or `curated` folders).
    * If pass:
        * Results of request are emailed to curation team.
        * Folder automatically moved to data commons repo `staging` folder.
            * Curators automatically given `own` permission.
            * User automatically given `write` permission.
        * User may [view all Permanent ID Requests](https://de.cyverse.org/terrain/docs/index.html#!/permanent45id45requests/get_terrain_permanent_id_requests) they have submitted.
        * User may [view details for any of their Permanent ID Requests](https://de.cyverse.org/terrain/docs/index.html#!/permanent45id45requests/get_terrain_permanent_id_requests_request_id).
    * If fail, user returns to Step 3.
5. Curator finds request with [Permanent ID Request listing](https://de.cyverse.org/terrain/docs/index.html#!/admin45permanent45id45requests/get_terrain_admin_permanent_id_requests).
    * Curator may [view the Permanent ID Request details](https://de.cyverse.org/terrain/docs/index.html#!/admin45permanent45id45requests/get_terrain_admin_permanent_id_requests_request_id),
      which includes the request's status history (initially only `Submitted`).
    * Curator checks metadata and data structure
    (see the [DOI Creation SOP](https://cyverse.atlassian.net/wiki/spaces/DC/pages/241867479/DOI+Creation+SOP+for+Curators) page for more details).
    * Curator may [update the status of the Permanent ID Request](https://de.cyverse.org/terrain/docs/index.html#!/admin45permanent45id45requests/post_terrain_admin_permanent_id_requests_request_id_status)
      in this or any subsequent step.
        * Curator may use any previously created
          [Permanent ID Request Status Codes](https://de.cyverse.org/terrain/docs/index.html#!/permanent45id45requests/get_terrain_permanent_id_requests_status_codes),
          or add a new status (which is saved for future reuse).
    * If pass or minor changes that can be made by curator: go to Step 6
    * If changes needed by user: Curator emails user and asks them to make changes
    * Once corrections are made, go to Step 6.
6. Curator [creates Permanent ID](https://de.cyverse.org/terrain/docs/index.html#!/admin45permanent45id45requests/post_terrain_admin_permanent_id_requests_request_id_doi) for this request.
    * Folder name must not conflict with any folder names in the data commons repo `curated` folder.
    * Metadata from folder is submitted to the [DataCite API](https://support.datacite.org/docs/api-create-dois)
      in order to create the requested Permanent ID
      (which currently only supports DOIs).
    * Fails:
        * Permanent ID not generated and error reported to curator.
        * [Request status automatically updated](https://de.cyverse.org/terrain/docs/index.html#!/admin45permanent45id45requests/post_terrain_admin_permanent_id_requests_request_id_status) to `Failed`.
        * Curator corrects any errors.
        * Repeat Step 6 until pass.
    * Passes:
        * Permanent ID is generated and notification sent to curator and user.
        * [Request status automatically updated](https://de.cyverse.org/terrain/docs/index.html#!/admin45permanent45id45requests/post_terrain_admin_permanent_id_requests_request_id_status) to `Completion`.
        * Folder metadata automatically updated to include new Permanent ID and current date.
        * Folder automatically moved to data commons repo `curated` folder.
            * Curators automatically given `own` permission.
            * Public automatically given `read` permission.
        * Curator checks that data is public and visible on mirrors,
          that metadata appears correct on the [DataCite Fabrica](https://doi.datacite.org/) landing page,
          and that the Permanent ID redirect works.
