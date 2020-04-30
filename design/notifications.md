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
that records events from the portion that serves the REST API will allow us to scale the two services independently.

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
modify iplant-email to listen to AMQP for incoming requests and remove this service. It would also be nice to be able to
send out something other than plain text email messages. Eventually, `iplant-email` will also be modified to add support
for sending email messages formatted in HTML.

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
future. Backwards compatibilty will be maintained by using `notification` as the category and placing the current
notification type in the update type field, so all app updates that are posted to the notification publishing route in
the REST API will automatically have a routing key of `events.notification.update.apps`. This may be slightly confusing,
but it's a good way to ensure that all backwards compatible notifications are managed by the same handler.

Once the event recorder receives a message, it has to decide what if anything to do with the message. If the event
recorder decides that no user has to be informed of the event then it will simply acknowledge the message and wait for
the next one to arrive. If it decides that one or more users have to be informed of the event then processing will
continue. This decision will be influenced by the category and the update type, so each handler will likely make this
decision in a unique manner except. In the case of the backwards compatible mode, the decision will be made in the same
way as in the curernt notification agent out of necessity.

The next step is to determine who should receive the notifications. This is another decision that is likely to vary
depending on category and update type. The handler for backwards compatible notifications will only send the
notification to the user mentioned in the `user` field of the notification request.

Next, the event recorder will have to determine whether or not an email should be sent. And once again, this decision
will probably vary depending on the category and update type. Eventually, we'll probably also want to have a
subscription mechanism so that users can choose which types of notifications they get emails for. Eventually, we'll also
want to be able to batch emails for cases when several similar updates arrive in rapid succession. For the backwards
compatibility case, however, the user will only receive an email if the notificaton request indicates that an email
should be sent.

The next step is to determine exactly what each message should say. At the risk of sounding repetitive, this is
something that will vary depending on the category and update type. We may want to explore internationalization and
localization for our notification system in the near future, too. The current system, where each service determines the
wording to use for each type of notification, doesn't really lend itself to internationalization and localization, so
this won't be implemented right away. We may want to investigate adding `go-i18n` (assuming we use Go) to one or more
notification services when we start adding handlers other than the backwards compatible handler, though.

Another thing to consider is how we're going to ensure message order. We can't really provide guarantees for the order
in which messages are processed, but we can take some steps to control the order in which messages are displayed to the
user. To this end, each update message will have a timestamp associated with it when it's posted to the AMQP
exchange. This timestmap will be recorded in the notification database, and it will be used to determine notification
order when requests are made to list notifications. Current notification requests don't have a timestamp associated with
them because they're recorded using synchronuous HTTP calls. Because of this, the endpoint in the service that provides
the backwards compatible REST API for notifications will add the timestamp to the notification request before publishing
it to the exchange. For updates that are posted directly to AMQP, the service itself will be responsible for including
the timestamp in the request.

#### Structure

The portion of the event recorder service that listens for incoming requests and dispatches them to handlers doesn't
have to change based on message category, so we can code it in a fairly straight forward manner. The event recorder will
subscribe to the routing key wildcard pattern, `events.*.update.*`. As mentioned above, the second and fouth components
of the routing key are the category and the update type. Once a message arrives, it will be dispatched to a handler
based on the category. If no handler was registered for the category, then the AMQP message is simply acknowledged and
ignored. If a handler has been registered for the category then the handler is called in order to process the message
further. The main listener function does very little else at this point. If the handler returns without an error, then
the message will be acknowleged. If an error that is likely to be recoverable (that is, if reprocessing the message at a
later time is likely to succeed) is returned, then the message is negatively acknowleged goes back into the queue for
later processing. At this time, no limits will be placed on message processing reattempts so that we can be sure to
process each message that has a reasonable chance of succeeding. If an error that is not likely to be recoverable is
returned then the most likely cause is a bug somewhere in our system, either in the service that sent the initial
request or in the event recorder itself. For this reason, an email will be sent to a configurable email address whenever
an unrecoverable error is detected. In CyVerse's DE deployment, we'll use `intercom-de@cyverse.org` so that an Intercom
issue will be created and we can follow up later. (At some later time, we may want to have alerts like these go to a
system monitoring tool such as Prometheus instead.)

Unrecoverable errors will be distinguished from recoverable errors by type. Errors of type `UnrecoverableError` will be
treated as unrecoverable. Errors of any other type will be treated as recoverable. A special `RecoverableError` type
will also be created to help make it clear when recoverable and unrecoverable errors are being created.

One need that is likely to arise is code reuse. I entertained the idea of using interface embedding along with a
template method pattern, but after some thought, this approach seems to be unnecessary. Instead, each message handler
will implement this interface:

``` go
type MessageHandler interface {
    HandleMessage(messageBody map[string]interface{}) error
}
```

Bits of code that are good candidates for reuse will be implemented in custom types so that they can be used from within
each message handler. Some good candidates are posting AMQP messages to the email and notification topics. We _could_
use the same signature for both cases, but we can provide a little bit of compile-time type checking by creating three
different types:

``` go
type MessagePublisher interface {
    Publish(message interface{}) error
}

type EmailRequestPublisher interface {
    Publish(request *EmailRequest) error
}

type NotificationRequestPublisher interface {
    Publish(request *NotificationRequest) error
}
```

The detals of the `EmailRequest` and `NotificationRequest` structs will be fleshed out in the implementation
phase. Also, the types may or may not be defined separately from their interfaces. I used the interface syntax in this
document because it provides a good way to describe a set of behavior implemented by a type in Go.

The `MessagePublisher` type will provide a way to publish an arbitrary message to an AMQP topic. Details about how and
to which topic messages will be published will be managed by instances of the `MessagePublisher` type. For example, the
`MessagePublisher` structue might contain a field to store the name of the AMQP topic that the messages will be
published to.

The other two types will be wrappers around `MessagePublisher` that validate their respective requests before actually
publishing them to the AMQP topic. If the message validation fails then the request publisher will return an
`UnrecoverableError`. All other errors encountered by the request publisher will result in a `RecoverableError` being
returned.

Another piece of code that's not likely to vary amongst message handlers is the code that stores the notification in the
database. Another type, `NotificationStorer`, will be created for this so that it can be shared amongst the different
message handler implementations.

### Email Requests

The email requests service has one, admittedly very simple, job. It reads messages from an AMQP topic and uses them to
format and send requests to the `iplant-email` service. Message acknowledgement will depend largely on the HTTP status
returned by `iplant-email`. If the status indicates success then the message is positively acknowledged. For the time
being, all status codes that indicate failure will cause the email requests service to log an error message and
positively acknowledge the AMQP message.

### RESTful Services

As mentioned above, this component will implement two incompatible APIs. API version 1 will implement the user
notifications portion of the current `notification-agent` service for backwards compatibility. System notifications
aren't used anymore, so they will not be implemented in this new version of the notification agent.

#### API Version 1

##### `POST /notification`

This endpoint posts an AMQP message to the `events.notification.update.{notification-type}` endpoint where it can be
processed by the event recorder component described above. See [the notification agent README][1] for details about the
request and response bodies.

##### `GET /messages`

This endpoint lists messages that have been stored in the database for the user. See [the notification agent README][2]
or details.

##### `GET /unseen-messages`

This endpoint lists messages that have been stored in the database for the user that haven't been marked as seen. See
[the notification agent README][2] or details.

##### `GET /last-ten-messages`

This endpoint lists only the ten most recent messages that were stored for the user in ascending order by
timestamp. This endpoint is a special case because it sorts messages in ascending order, but only returns the ten most
recent. See [the notification agent README][3] for details.

##### `GET /count-messages`

This endpoint counts messages of various types, and it will not be fully implemented in the new service because system
notifications are no longer being supported. See [the notification agent README][4] for endpoint details.

##### `POST /seen`

This endpoint marks notifications as having been seen. See [the notification agent README][5] for details.

##### `POST /mark-all-seen`

This endpoint marks all notifications for a user as having been seen. See [the notification agent README][6] for
details.

##### `POST /delete`

This endpoint marks notifications as having been deleted. See [the notification agent README][7] for details.

##### `POST /delete-all`

This endpoint marks all notifications for a user as having been deleted. See [the notification agent README][8] for
details.

#### API Version 2

##### `GET /messages`

This endpoint lists messages that have been stored in the database for a user, and will remain largely unchanged from
its counterpart in version 1 of the API. A couple of changes will be made, however. First the camelCased query parameter
names will be replaced with kebab-cased names instead. Second, another new Boolean parameter, `count-only`, will be
added that instructs the endpoint to return just the message counts rather than the actual messages. Setting this query
parameter to `true` replaces the functionality of the `count-messages` from version 1 of the API. The response body will
take the following form:

```
{
    "total": 32,
    "messages": []
}
```

The `total` element will contain the total number of messages that match the request. This element will always be
present in the response body. The `messages` element will contain the actual notification messages. It will not be
present if the `count-only` query parameter is present and set to `true`. The format of each message in the list of
messages will be the same as in API version 1, except that camelCased elements in the top level of the response will be
replaced by snake_cased elements instead.

Available query parameters:

| Query Parameter | Description                                                                                  |
| --------------- | -------------------------------------------------------------------------------------------- |
| user            | The username of the user to retrieve notification messages for (required).                   |
| limit           | The maximum number of messages to return in the response body (default: all messages).       |
| offset          | The offset of the first message to return in the response body (default: 0).                 |
| seen            | If `true` messages that have been seen will be included in the response (default: false).    |
| sort-field      | The field to sort messages by (default: timestamp).                                          |
| sort-dir        | Allows the response body to sorted in ascending or descending order (default: descending).   |
| message-type    | If specified, only messages of the given type will be included in the response.              |
| count-only      | If `true` only the number of matching messages will be returned.                             |

##### `GET /messages/{id}`

This endpoint responds with details about the message with the given message ID. The message must be directed toward the
user specified in the `user` query parameeter for this endpoint to succeed. If the selected message doesn't exist or
refers to a notification that isn't directed toward the user then a 404 status will be returned. The response body will
be the same as the format of the the messages in the `GET /messages` endpoint.

Available query parameters:

| Query Parameter    | Description                                                                                  |
| ------------------ | -------------------------------------------------------------------------------------------- |
| user               | The username of the user to retrieve notification messages for (required).                   |

##### `POST /messages/{id}/seen`

This endpoint marks an individual message as having been seen. If the message doesn't exist or isn't directed to the
user mentioned in the `user` query parameter then a 404 status will be returned. This endpoint has no response body.

Available query parameters:

| Query Parameter    | Description                                                                                  |
| ------------------ | -------------------------------------------------------------------------------------------- |
| user               | The username of the user to update notification messages for (required).                     |

##### `DELETE /messages/{id}`

This endpoint marks an individual message as having ben deleted. If the message doesn't exist or isn't directed to the
user mentioned in the `user` query parameter then a 404 satus will be returned. This endpoint has no response body.

Available query parameters:

| Query Parameter    | Description                                                                                  |
| ------------------ | -------------------------------------------------------------------------------------------- |
| user               | The username of the user to delete notification messages for (required).                     |

##### `POST /messages/seen`

This endpoint replaces the `/seen` endpoint in the original API. It allows callers to mark multiple notifications as
having been seen at once. The request body will change slightly, however. The new request body will contain two optional
elements: `ids` and `all_notifications`. The `ids` element will contain a list of notification IDs to mark as seen. The
`all_notifications` element contains a Boolean flag indicating whether or not all notificatons for the user should be
marked as seen. The `ids` element will be ignored if this element is included and set to `true`.

Available query parameters:

| Query Parameter    | Description                                                                                  |
| ------------------ | -------------------------------------------------------------------------------------------- |
| user               | The username of the user to update notification messages for (required).                     |

Request body:

```
{
    "ids": [],
    "all_notifications": false
}
```

This endpoint has no response body.

##### `POST /messages/delete`

This endpoint replaces the `/delete` endpoint in the original API. It allows callers to mark multiple notifications as
having been deleted at once. The request body will change slightly, however. The new request body will contain two
optional elements: `ids` and `all_notifications`. The `ids` element will contain a list of notification IDs to mark as
deleted. The `all_notifications` element contains a Boolean flag indicating whether or not all notificatons for the user
should be marked as deleted. The `ids` element will be ignored if this element is included and set to `true`.

Available query parameters:

| Query Parameter    | Description                                                                                  |
| ------------------ | -------------------------------------------------------------------------------------------- |
| user               | The username of the user to delete notification messages for (required).                     |

Request body:

```
{
    "ids": [],
    "all_notifications": false
}
```

This endpoint has no response body.

[1]: https://github.com/cyverse-de/notification-agent#requesting-an-arbitrary-notification
[2]: https://github.com/cyverse-de/notification-agent#getting-notifications-from-the-notification-agent
[3]: https://github.com/cyverse-de/notification-agent#getting-the-ten-most-recent-notifications
[4]: https://github.com/cyverse-de/notification-agent#counting-notifications
[5]: https://github.com/cyverse-de/notification-agent#marking-notifications-as-seen
[6]: https://github.com/cyverse-de/notification-agent#marking-all-notifications-as-seen
[7]: https://github.com/cyverse-de/notification-agent#deleting-notifications
[8]: https://github.com/cyverse-de/notification-agent#deleting-all-notifications
