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
| `Auth` | `Authorization` / `WWW-Authenticate` parser and builder (RFC 7235) |
| `Credentials` | one authentication scheme with its token68 or auth-params |
| `Multipart` | `multipart/form-data` body decoder |
| `FormPart` | a single decoded multipart part (name, filename, content-type, body) |
| `TransferEncoding` | Chunked transfer-encoding decoder |

## Testing

```
carp -x test/http.carp
```

You can find the API documentation [online](https://veitheller.de/http/)!

<hr/>

Have fun!
