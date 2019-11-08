---
layout: page
title: DE API Documentation
---

## Discovery Environment API (Terrain)

Terrain provides the primary point of communication between the Discovery Environment (DE) UI and backend services. It's charged with two primary tasks: to handle authentication and authorization for endpoints that require it, and to orchestrate calls to other lower-level services.

### Additional Documentation

The primary and most current source for Terrain API endpoint documentation
are the Swagger docs hosted by the service itself:
[https://de.cyverse.org/terrain/docs](https://de.cyverse.org/terrain/docs)

We are still in the process of migrating the original endpoint documentation found on this site to the Swagger docs hosted by the service.
Any endpoints listed under the [default](https://de.cyverse.org/terrain/docs/index.html#!/default)
Swagger category have not been documented in Swagger yet.
The documentation for those endpoints may still be found below.

* [Endpoints](endpoints)
* [Errors](errors.html)
* [Schema](schema.html)
* [Endpoint Index](endpoint-index.html)

### Authentication

Authentication to the DE services is currently handled by passing signed JSON Web Tokens (JWTs) in the custom HTTP header, `X-Iplant-De-Jwt`, in each call to the service. The JWT payload used by the DE is structured as follows:

```json
{
    "sub": "username",
    "email": "username@example.org",
    "given_name": "Joe",
    "family_name": "User",
    "name": "Joe User",
    "org.iplantc.de:entitlement": [
        "some-group",
        "some-other-group",
        "yet-another-group"
    ]
}
```

The DE doesn't currently support the `name` field, but there are plans to add support for this field in an upcoming release. The rest of the field values are obtained indirectly from LDAP via CAS. The fields are (or will be in the case of the `name` field) obtained as follows:

| JWT Claim (Field)          | LDAP Attribute    | CAS Attribute |
| -------------------------- | ----------------- | ------------- |
| sub                        | uid               | uid           |
| email                      | mail              | email         |
| given_name                 | givenName         | firstName     |
| family_name                | sn                | lastName      |
| name                       | cn                | name          |
| org.iplantc.de:entitlement | _aggregate field_ | entitlement   |

<br>The `entitlement` attribute is aggregated from LDAP by placing the name of each group that the user belongs to into an array. The list is obtained by extracting the `cn` attribute of every LDAP group that has the username of the authenticated user as one of its `memberUid` field values.

The JWT is then signed and encoded according to [RFC 7519](https://tools.ietf.org/html/rfc7519) and [RFC 7515](https://tools.ietf.org/html/rfc7515). Note that the only signing algorithm currently supported by Terrain is `RS256`. For more information, please see [jwt.io](http://jwt.io/).

For information on development testing, please see the [documentation for make-jwt](/tools/make-jwt).
