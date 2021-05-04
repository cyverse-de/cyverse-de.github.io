---
layout: page
title: Discovery Environment (DE) Release Notes
---

# Discovery Environment (DE) Release Notes

## May 2021

* The Discovery Environment 2.0 was released and is now available at [https://de.cyverse.org](https://de.cyverse.org)
* The legacy Discovery Environment is still available for users until July 5th 2021 at [https://legacy-de.cyverse.org](https://legacy-de.cyverse.org)
* Features ported to DE 2.0 from the Legacy DE during this release
  * Apps: integrating new apps, editing, and publishing
  * Webhooks Notification
  * Moving files and folders
  * Requesting access to Visual Interactive Computing Environment (VICE) apps
  * Teams: request to join a team and notifications related to your request
  * HPC (TAPIS authentication)
  * VICE analysis authentication using Keycloak
* New Feature
  * Instant Launch - Instanly launch a VICE app from the dashboard with 1 click or from the Data View
  * Access the Discovery Environment using github credentials
  * Copy link for an app, file or folder from corresponding view or from the search results
  * In-app VICE loading page


## February 2021

* The Discovery Environment 2.0 (beta) is now live at [https://de2.cyverse.org](https://de2.cyverse.org)
* Includes performance improvements and new features such as a Dashboard with recently used apps and analyses, global search bar, and listings for public apps and data with no login required.
* Some features in the original DE (still available at [https://de.cyverse.org](https://de.cyverse.org) until deprecated in summer 2021) that are not yet implemented in DE 2.0:
  * Apps: integrating new apps, editing, copying, and publishing
  * Communities: administrating, managing, and browsing
  * Teams: request to join a team and notifications related to your request
  * Notifications: some notifications, some webhooks, and some keyboard shortcuts are not yet available
  * Requesting access to Visual Interactive Computing Environment (VICE) apps
  * Moving files and folders

## December 2020

* Allowed anonymous user access to public apps (includes public apps listing, searching, and viewing app details and 
  documentation)

## November 2020

* Assorted performance improvements in data, app, and analysis listings, as well as other data endpoints.
* Allowed anonymous user access to public community data (includes directory listing, searching, and
 viewing files)

## September 2020

* Fixed a bug in the DE where dialogs with paths in them would incorrectly mark the path as invalid and fail validation
* Added announcers in the DE to indicate to users that data moves are now asynchronous

## August 2020

* Users will now receive notifications when asynchronous file and folder actions (rename, move, delete, etc.) are completed.
* Fixed a bug in the DOI Request service to allow the data folder to move asynchronously.

## July 2020

* VICE apps are now restricted and require authorization.
  Users will be notified to request access, or if their concurrent VICE job limit is reached.

## March 2020

* Bug where moves into or out of collections with inheritance turned on is presumed fixed.
* Wording in the Introduction has been updated.
* Bug where trying to select a collaborator via Choose Collaborators in the sharing dialog is now fixed.
* Relaunching more than one analysis at a time is now supported:
    * Relaunching more than one analysis at once will relaunch all selected analyses, reusing their original parameters and analysis names.
    * If the selected analyses are sub-jobs of an HT analysis, then those selected analyses will still be nested under that parent HT analysis, and their output folders will also be grouped under that parent HT analysis' output folder, but the relaunched sub-jobs will be renamed with a `-redo-#` suffix to differentiate them from their original sub-jobs.
    * Otherwise, relaunched analyses will be treated as new analyses, even though they reuse the same name and parameters as their original analyses.


## January 2020

* Communities
    * Fixed a bug where users were able to share apps with a community via the Share Dialog in the Apps window.
      This was an unintended feature discovered by a user attempting to get help with their app from the science team.
      Users should continue to share their apps/analyses to the `siuser` account to get help.
    * Fixed a bug where users had to deselect the previously selected app in order to select a new app to tag with a community.
* Analysis Relaunch
    * Fixed a bug to allow resource requirements set in the original analysis to be used instead of the tool default resource requirements.
* React Migrations
    * Tool Details dialog
    * Edit Teams dialog


## November 2019

* When the user deletes a submitted or running analysis, the service has been updated to cancel the analysis first.
* [Analyses Resource Requirements](https://learning.cyverse.org/projects/discovery-environment-guide/en/latest/analyses_resource_reqs.html)
  * User adjustable resource requests, based on the resource requirements and limits defined by an app’s tools, are now available from the app launch dialog.
* Changes to app publication requests
  * A single-step app using a public tool. The app should be published and the service returns a 200 status.
  * A single-step app using a private tool that is in one of the trusted registries. The app and tool will be published and the service returns a 200 status.
  * A single-step app using a private tool that is not in one of the trusted registries. The app will not be published. Instead, a publication request will be created and the service returns a 202 status.
  * A pipeline using some public tools and some private tools that are in trusted registries. The pipeline will not be published, and the service returns a 400 status.
  * A pipeline using some public tools and some private tools that are not in trusted registries.  The pipeline will not be published, and the service will return a 400 status.
* React migrations
  * Manage Tools window
  * Teams window
  * App Listing
* Exporting metadata to a file (required for NCBI submissions) is now much faster for folders.


## September 2019

* Run Visual and Interactive Computing Environment (VICE) apps like JupyterLab,
R Studio etc entirely in Kubernetes Cluster avoiding scheduling contention with non-VICE analyses.
* View Logs of VICE analyses using Analyses Menu -> View Logs
* Time limit extension button now available in the Discovery Environment.
Click on “Hour Glass” icon next to the analysis status to extend the time limit for your VICE analysis.
* The VICE web app at cyverse.run is now deprecated.
All functionality is available through the Discovery Environment.
The site at cyverse.run will redirect to the Discovery Environment in a future release.
* More updates to [Swagger public API documentation](https://de.cyverse.org/terrain/docs/index.html)
* Fixed a bug that prevented users behind HTTP proxies from being able to log in to the Discovery Environment.


## Jan 2019 - Aug 2019

* Migrated DE Desktop, analyses window, details panel of Data window from GXT to Material UI library.
* Migrated Metadata Dialog to React, allowing editing of nested metadata AVUs.
* Release [DE public API documentation](https://de.cyverse.org/terrain/docs/index.html)
* Tool integrators can now set min and max resource requirements (CPU, RAM, and disk space).
* Create `Quick Launch` for an analysis.
Attach this button your website to redirect users to the DE to relaunch your analysis with preconfigured inputs.
* Initial release of VICE.
* Added support for the new TAPIS job submission API, Aloe.
* Fixed a bug that prevented jobs using input files whose paths contain spaces from being submitted to Aloe.
* Communities added to DE
* Added analysis history to the analysis info dialog.
* QA testing migrated to Kubernetes
* User and error details added to the DE’s error page
* Advanced Search in Data window migrated to React
* Edit Tools dialog migrated to React
* Add ability to do basic searching of nested AVUs in the DE
* Improved case where sometimes not all data files would be transferred back to the Data Store when “Complete and Save” is used
-- the time limit for the upload process should now be an hour, rather than 10 minutes
