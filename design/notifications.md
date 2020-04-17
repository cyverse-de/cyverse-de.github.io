# Notifications Redesign

## Motivation

The current notification system is really more intrusive than a notification system should be. For example, there are
several operations in the DE that will fail if the notification service is down. In an ideal implementation, the
notification system should be mostly transparent. Making the notification system completely transparent probably isn’t
feasible at this time because that would require us to eliminate virtually all point-to-point inter-service
communication, which isn’t going to happen anytime soon. We can make some improvements, however, by altering the way
that events are delivered to the notification system.

## Architecture Changes

The following high-level changes will be made to the notification system:

- Events will now be delivered to the notification system via AMQP.
- Support for system messages will be removed, since they’re no longer used.
- The current notification agent will be split into three pieces: one dedicated to recording events, one dedicated to
  sending email messages for notifications that have been recorded, and one that handles HTTP requests.
- The service dedicated to recording events will simply listen to an AMQP topic for messages, record them in the
  database and publish resulting AMQP messages to one topic for email requests and one for notifications.
- The service dedicated to processing email requests will listen for incoming messages and forward requests to
  iplant-email in order to send messages. We may eventually decide to remove this service if iplant-email is modified to
  listen to AMQP for incoming requests.
- The current API will initially be implemented in the service that handles HTTP requests for backwards compatibility,
  so that we don’t have to migrate all of our other services at once.
- A new, more RESTful API will be implemented as well, with the goal of eventually migrating all services to the new
  API.

This change is small enough to be made without an inordinate amount of work. The service that handles HTTP requests will
be a drop-in replacement for the existing notification agent. Sonora can be written to use the new Notification
API. Existing DE services can use the existing API until they can be migrated at a later time.

### Benefits of Switching to AMQP

The first benefit of switching to AMQP is that it will eventually allow us to decouple the notification system from the
various client services. Since the AMQP broker will be acting as an intermediary between the notification system and
other services, it will no longer be necessary for the notification system to be running to be able to perform various
tasks, at least not once the client services have been modified to publish events directly to AMQP. Services that are
still using the backwards-compatible API will still have to rely on the presence of the notifications service to perform
their tasks.

The second benefit is that it allows other services to listen for these same events and perform tasks when they
occur. Eventually, this should allow us to create a much more dynamic system that relies far less on synchronous HTTP
requests.

### Benefits of Splitting Event Recording From the REST API

Aside from the usual benefits of establishing a separation of concerns, splitting the portion of the notification agent
that records events from the portion that serves the REST API will allow us to scale the two services separately.

## Component Overviews

### Event Recorder

This service will listen for incoming AMQP messages. It will record each message that it receives in the notifications
database then publish related messages to two different AMQP topics: one for email requests, and one for general
notifications. The topic for email requests will be new; email requests are currently forwarded to `iplant-email` by the
notification agent itself. The topic for notifications already exists. This is the topic that the current DE and
de-webhooks listen to for incoming notifications.

### Email Requests

This service will listen to the new AMQP topic for incoming email requests and make calls to iplant-email in order to
actually send the messages. We may eventually want to modify other DE services to publish messages to this topic in
order to request outgoing emails. If and when all of the other DE services are modified to use AMQP then we may want to
modify iplant-email to listen to AMQP for incoming requests and remove this service.

### RESTful Services

This service will provide a general REST API for notifications. It will provide two incompatible versions of the
API. API version 1 will replicate the API served by the current notification agent. API version 2 will provide a much
more RESTful API.

This service will not record notifications in the database. In fact, the endpoint to publish notifications will not
exist in version 2 of the API. When a notification request is received, it will simply be published to the events topic
in the AMQP broker.

## Implementation Options

The three new services will be implemented from scratch in order to avoid issues with legacy code lingering after the
rewrite. We may want to write each respective service in either Go or Clojure. The advantages of Go are that the startup
times and memory footprints of the services will be much smaller. The primary advantage of Clojure is that it will allow
us to develop the services more rapidly.

Given the resource constraints that we’ve encountered in external deployments, I would strongly recommend going with Go
for this service despite the increased development time. If we end up having more than a few deployments in
resource-constrained environments then the reduction in deployment headaches will more than make up for the development
time.

## Component Detail

### Event Recorder

The event recorder is the first link in the notification processing chain. It listens to the AMQP exchange for incoming
messages indicating that an event has occurred. For the time being, this routing key will follow the format
`events.{category}.update.{update-type}`. The category component of the routing key will describe the category of the
event. The update type refers to the type of update that occurred. For example, if an app was published then the
category would be `apps` and the update type would be `published`, meaning that the routing key would be
`events.apps.update.published`. Having different routing keys for different types of updates will eventually allow us to
easily support different handlers for different types of updates if we wish to do so at some point in the
future. Backwards compatibilty will be maintained by adding an update type of `notification`, so all app updates that
are posted to the notification publishing route in the REST API will automatically have a routing key of
`events.apps.update.notification'.

Once the event recorder receives a message, it has to decide what if anything to do with the message. If the event
recorder decides that no user has to be informed of the event then it will simply acknowledge the message and wait for
the next one to arrive. If it decides that one or more users have to be informed of the event then processing will
continue. This decision will be influenced by the category and the update type, so each handler will likely make this in
a unique manner except in the case of the backwards compatible mode, where the decision has to be made in the same way
as in the curernt notification agent by necessity.

[Who should be notified?]

[Should an email be sent?]

[What should each message say? Should we add internationalization and localization now?]

[How should we maintain message order?]
