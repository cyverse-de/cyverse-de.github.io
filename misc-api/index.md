---
layout: page
title: DE Miscellaneous APIs Documentation
---

## Miscellaneous/extra APIs not covered by Terrain

This is only for a small and generally unauthenticated subset of the API features. Most API features are documented in the [Main API section](../api) instead.

### Sending status updates for jobs

Status updates for jobs can be sent to an unauthenticated endpoint, to ensure that jobs running off-premise can update their own status. If you access the DE at https://de.example.com, the status update endpoint is available at `https://de.example.com/job/uuid/status`, where `uuid` should be replaced with the job's external ID. For a running job, this is available in the `job` file, under the `uuid` key.

The body of the request should be JSON describing the state of the job, the hostname the update is being sent from, and a descriptive message, for example:

```json
{
  "hostname": "foo.example.com",
  "message": "Finished downloading input files",
  "state": "running"
}
```

With curl, the whole request might look like:

```sh
curl -XPOST -d '{"hostname": "foo.example.com", "message": "Finished downloading input files", "state": "running"}' https://de.example.com/job/2c8f0b52-a3fc-4f42-bf56-791a218e9bc4/status
```

The available states are:

* submitted: this should be used whenever the job has not yet begun any work, and in general shouldn't need to be updated by the job itself
* running: this should be used for anything during the job's stage-in, execution, and stage-out
* completed: this should be used when the job has fully completed and the results are ready to view within the job's output folder
* failed: this should be used when the job has fully completed *and will do no further work*, i.e., this is a permanent failure state and not a transient one
