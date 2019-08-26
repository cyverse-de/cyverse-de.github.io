---
layout: page
title: DE Quick Launches API Documentation
---

# Quick Launches

Quick Launches provide a way to set default parameter values for an analysis,
which can make it much easier to launch a large number of similar jobs
without having to select the parameter values that the jobs have in common for every new analysis.

Quick Launches are created on a per-user basis,
but they can be made public when they’re created and shared with other users.

The Swagger docs are available here: https://de.cyverse.org/terrain/docs/index.html#/analyses-quicklaunches

Creating a new Quick Launch is a lot like launching an app.
The primary difference is that instead of calling the
[POST /terrain/analyses](https://de.cyverse.org/terrain/docs/index.html#!/analyses/post_terrain_analyses)
endpoint, call the
[POST /terrain/quicklaunches](https://de.cyverse.org/terrain/docs/index.html#!/analyses-quicklaunches/post_terrain_quicklaunches)
endpoint.
The request body contains an analysis submission wrapped in an outer object.
The fields at the top level describe the Quick Launch itself.
The object in the `submission` field contains the request body that's normally sent to the
[POST /terrain/analyses](https://de.cyverse.org/terrain/docs/index.html#!/analyses/post_terrain_analyses)
endpoint when launching a job.
Note that not all required analysis parameters of the `AnalysisSubmissionConfig`
need to be specified in the Quick Launch.

Once a Quick Launch is defined, the
[GET /terrain/quicklaunches](https://de.cyverse.org/terrain/docs/index.html#!/analyses-quicklaunches/get_terrain_quicklaunches)
endpoint can be used to list Quick Launches that are available to the user.
The description currently says that it lists Quick Launches that were created by the user.
This isn’t quite correct.
Any Quick Launch that the user has permission to view will be listed by this endpoint.

All available Quick Launches for an app can be listed using the
[GET /terrain/quicklaunches/apps/{app-id}](https://de.cyverse.org/terrain/docs/index.html#!/analyses-quicklaunches/get_terrain_quicklaunches_apps_app_id)
endpoint.
This endpoint will only list Quick Launches that the user has permission to view.

Quick Launch defaults allow the user to specify which Quick Launch to use for a specific app.
Both global defaults and user-specific defaults can be defined.

The app launch info saved in a Quick Launch can be retrieved with the
[GET /terrain/quicklaunches/{ql-id}/app-info](https://de.cyverse.org/terrain/docs/index.html#!/analyses-quicklaunches/get_terrain_quicklaunches_ql_id_app_info)
endpoint.
This is analogous to calling the
[GET /terrain/apps/{system-id}/{app-id}](https://de.cyverse.org/terrain/docs/index.html#!/apps/get_terrain_apps_system_id_app_id)
endpoint.
