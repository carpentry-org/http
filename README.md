# http

An HTTP request/response parser and serializer for Carp.

## Installation

```clojure
(load "git@github.com:carpentry-org/http@0.3.0")
```

## Usage

### Parsing requests

```clojure
(match (Request.parse "GET / HTTP/1.1\r\nHost: example.com\r\n\r\n")
  (Result.Success req)
    (println* (Request.verb &req) " " &(URI.str (Request.uri &req)))
  (Result.Error e) (IO.errorln &e))
```

### Parsing responses

```clojure
(match (Response.parse "HTTP/1.1 200 OK\r\nContent-Length: 5\r\n\r\nhello")
  (Result.Success resp)
    (println* (Response.code &resp) " " (Response.body &resp))
  (Result.Error e) (IO.errorln &e))
```

### Building requests

```clojure
(let [req (Request.get (URI.zero) [] {} @"")]
  (println* &(Request.str &req)))
```

### Case-insensitive header lookup

```clojure
(match (Request.header &req "content-type")
  (Maybe.Just ct) (println* "Content-Type: " &ct)
  (Maybe.Nothing) (println* "no content-type"))
```

### Chunked transfer decoding

When a message uses `Transfer-Encoding: chunked`, the parsed body holds the raw
chunk framing. Detect it with `chunked?` and decode it with
`TransferEncoding.dechunk`:

```clojure
(match (Response.parse resp)
  (Result.Success r)
    (if (Response.chunked? &r)
      (match (TransferEncoding.dechunk (Response.body &r))
        (Result.Success body) (println* &body)
        (Result.Error e) (IO.errorln &e))
      (println* (Response.body &r)))
  (Result.Error e) (IO.errorln &e))
```

### Form body parsing

```clojure
(match (Form.parse "name=carp&version=1")
  (Result.Success m)
    (println* "name=" &(Map.get &m "name"))
  _ ())
```

### Authentication

`Auth` parses and builds the RFC 7235 headers: the credentials a client sends in
`Authorization`, and the challenges a server answers a 401 with in
`WWW-Authenticate`. One `WWW-Authenticate` value may carry several challenges.

```clojure
(match (Auth.parse "Basic YWxhZGRpbjpvcGVuIHNlc2FtZQ==")
  (Result.Success c)
    (match (Auth.basic-credentials &c)
      (Result.Success up) (println* (Pair.a &up) ":" (Pair.b &up))
      (Result.Error e) (IO.errorln &e))
  (Result.Error e) (IO.errorln &e))

(Auth.basic "aladdin" "open sesame")
; => (Success "Basic YWxhZGRpbjpvcGVuIHNlc2FtZQ==")
(Auth.bearer "mF_9.B5f-4.1JqM") ; => "Bearer mF_9.B5f-4.1JqM"

(let [cs (Auth.parse-challenges "Basic realm=\"a\", Digest realm=\"b\"")]
  (println* (Credentials.realm (Array.unsafe-nth &cs 1)))) ; => (Just "b")

(Response.unauthorized (Auth.basic-challenge "WallyWorld") {} @"go away")
```

### Content negotiation

`Accept`, `AcceptEncoding` and `AcceptLanguage` parse their header into weighted
entries and pick the best of the server's offers, returning `(Maybe String)`.
The `Request` wrappers read the header off the request and apply the right
default when it is absent.

```clojure
(Accept.negotiate "text/html;q=0.8, application/json;q=0.9"
                  &[@"text/html" @"application/json"])
; => (Just "application/json")

(Request.negotiate &req &[@"text/html" @"application/json"])
```

`Accept-Encoding` follows RFC 9110 §12.5.3: `identity` is acceptable unless the
header refuses it, `*` stands in for any coding the header does not name, and
`q=0` means unacceptable rather than least preferred. A request with no
`Accept-Encoding` accepts anything; one with an empty `Accept-Encoding` accepts
only `identity`.

```clojure
(AcceptEncoding.negotiate "*;q=0, gzip" &[@"gzip" @"identity"]) ; => (Just "gzip")
(AcceptEncoding.negotiate "gzip;q=0" &[@"gzip"])                ; => (Nothing)

(Request.negotiate-encoding &req &[@"gzip" @"identity"])
```

`Accept-Language` matches ranges against tags by RFC 4647 §3.3.1 basic
filtering, so `en` matches `en-US` but not `eng`, and `en-US` does not match
`en`. More specific ranges win at equal weight.

```clojure
(AcceptLanguage.negotiate "de, en;q=0.5" &[@"en-US" @"de-AT"]) ; => (Just "de-AT")

(Request.negotiate-language &req &[@"en" @"de"])
```

`Accept-Charset` has no counterpart: RFC 9110 §12.5.2 deprecates it.

### Status codes

```clojure
Status.ok           ; => 200
Status.not-found    ; => 404
(Status.reason 200) ; => "OK"
```

## Types

| Type | Purpose |
|------|---------|
| `Request` | HTTP request (verb, version, URI, headers, cookies, body) |
| `Response` | HTTP response (code, message, version, headers, cookies, body) |
| `Cookie` | HTTP cookie with attributes |
| `SameSite` | Cookie SameSite attribute (Lax, Strict, None) |
| `Status` | Status code constants and reason phrases |
| `Form` | URL-encoded form body parser |
| `MediaType` | `Content-Type` / media-type parser (type, subtype, parameters) |
| `Accept` | `Accept` parser and media-type negotiation (RFC 7231) |
| `MediaRange` | one weighted media range of an `Accept` header |
| `AcceptEncoding` | `Accept-Encoding` parser and content-coding negotiation (RFC 9110) |
| `AcceptLanguage` | `Accept-Language` parser and language negotiation (RFC 4647) |
| `Weighted` | one entry of a weighted header list (value and `q`) |
| `Auth` | `Authorization` / `WWW-Authenticate` parser and builder (RFC 7235) |
| `Credentials` | one authentication scheme with its token68 or auth-params |
| `Multipart` | `multipart/form-data` body decoder |
| `FormPart` | a single decoded multipart part (name, filename, content-type, body) |
| `TransferEncoding` | Chunked transfer-encoding decoder |
| `HttpDate` | HTTP-date parser and formatter (RFC 9110 §5.6.7) |

## Testing

```
carp -x test/http.carp
```

You can find the API documentation [online](https://veitheller.de/http/)!

<hr/>

Have fun!
