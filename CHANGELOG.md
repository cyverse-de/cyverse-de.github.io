---
layout: page
title: Discovery Environment (DE) Release Notes
---

# Discovery Environment (DE) Release Notes

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
