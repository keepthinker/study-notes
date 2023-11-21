# HTTP Authentication: Basic and Digest Access Authentication

## Basic Authentication Scheme

The "basic" authentication scheme is based on the model that the client must authenticate itself with a user-ID and a password for each realm.  The realm value should be considered an opaque string which can only be compared for equality with other realms on that server. The server will service the request only if it can validate the user-ID and password for the protection space of the Request-URI. There are no optional authentication parameters. 

For Basic, the framework above is utilized as follows:

```properties
challenge   = "Basic" realm
credentials = "Basic" basic-credentials
```

Upon receipt of an unauthorized request for a URI within the protection space, the origin server MAY respond with a challenge like the following:

```properties
WWW-Authenticate: Basic realm="WallyWorld"
```

 where "WallyWorld" is the string assigned by the server to identify the protection space of the Request-URI. A proxy may respond with the same challenge using the Proxy-Authenticate header field. To receive authorization, the client sends the userid and password, separated by a single colon (":") character, within a base64  encoded string in the credentials.

```properties
basic-credentials = base64-user-pass
base64-user-pass = <base64 encoding of user-pass, except not limited to 76 char/line>
user-pass   = userid ":" password
userid      = *<TEXT excluding ":">
password    = *TEXT
```

Userids might be case sensitive.
If the user agent wishes to send the userid "Aladdin" and password "open sesame", it would use the following header field:

```properties
   Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
```

A client SHOULD assume that all paths at or deeper than the depth of the last symbolic element in the path field of the Request-URI also are within the protection space specified by the Basic realm value of the current challenge. A client MAY preemptively send the corresponding Authorization header with requests for resources in that space without receipt of another challenge from the server. Similarly, when a client sends a request to a proxy, it may reuse a userid and password in the Proxy-Authorization header field without receiving another challenge from the proxy server. See [section 4](https://www.rfc-editor.org/rfc/rfc2617#section-4) for security considerations associated with Basic authentication.

## 参考

[RFC 2617: HTTP Authentication: Basic and Digest Access Authentication](https://www.rfc-editor.org/rfc/rfc2617#page-3)
