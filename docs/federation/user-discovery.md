# User Discovery

> **Note:** The terms "the server" and "the requesting server" are used in this section. "The server" refers to the server that is being discovered, and "the requesting server" refers to the server that is trying to discover the server.

Servers **MUST** implement the [WebFinger](https://tools.ietf.org/html/rfc7033) protocol to allow other servers to discover their endpoints. This is done by serving a `host-meta` file at the address `/.well-known/host-meta`.

The document **MUST** contain the following information, as specified by the WebFinger protocol:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<XRD xmlns="http://docs.oasis-open.org/ns/xri/xrd-1.0">
    <Link rel="lrdd" type="application/xrd+xml" template="https://example.com/.well-known/webfinger?resource={uri}" />
</XRD>
```

The `template` field **MUST** be the URI of the server's WebFinger endpoint, which is usually `https://example.com/.well-known/webfinger?resource={uri}`.

The `resource` field **MUST** be the URI of the user that the server is trying to discover (in the format `acct:uuid@example.com`)

Breaking down this URI, we get the following:

- `acct`: The protocol of the URI. This is always `acct` for Lysand.
- `uuid`: The UUID of the user that the server is trying to discover.
- `example.com`: The domain of the server that the user is on. This is usually the domain of the server. This can also be a subdomain of the server, such as `lysand.example.com`.

This format is reminiscent of the `acct` format used by ActivityPub, but with a UUID instead of a username. Users should typically not use the `id` of an actor to identify it, but instead its `username`.

---

Once the server's WebFinger endpoint has been discovered, it can receive a `GET` request to the endpoint to discover the endpoints of the user.

The requesting server **MUST** send a `GET` request to the endpoint `https://example.com/.well-known/webfinger`.

The requesting server **MUST** send the following headers with the request:

- `Accept: application/jrd+json`
- `Accept: application/json`

The requestinng server **MUST** send the following query parameters with the request:

- `resource`: The URI of the user that the server is trying to discover (in the format `acct:uuid@example.com` (replace `uuid` with the user's ID)

---

The server **MUST** respond with a `200 OK` response code, and a JSON object in the body. This JSON object **MUST** contain the following information, as specified by the WebFinger protocol:

```json5
{
    "subject": "acct:uuid@example.com",
    "links": [
        {
            "rel": "self",
            "type": "application/json",
            "href": "https://example.com/users/uuid"
        },
        // The following links are optional but could be added to server software to display XML feeds and an HTML profile page
        {
            "rel": "http://webfinger.net/rel/profile-page",
            "type": "text/html",
            "href": "https://example.com/users/uuid"
        },
        {
            "rel": "http://schemas.google.com/g/2010#updates-from",
            "type": "application/atom+xml",
            "href": "https://example.com/users/uuid"
        },
    ]
}
```

> **Note:** The `subject` field **MUST** be the same as the `resource` field in the request.

> **Note:** The server implementation is free to add any additional links to the `links` array, such as for compatibility with other federation protocols. However, the links specified above **MUST** be included.

>The `href` values of these links can be anything as long as it includes the `uuid` of the user, such as `https://example.com/accounts/uuid` or `https://example.com/uuid.`.
