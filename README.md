# http

HTTP for Carp, expressed as a parser and a data structure.

A very early work in progress. Currently implemented: Simple HTTP requests.

## Installation

We recommend you don’t use this library yet. If you have to, use:

```clojure
(load "git@github.com:carpentry-org/http@master")
```

## Usage

Currently, all you can do is parse HTTP requests, by using `HTTP.parse`.
Hopefully you’ll soon be able to do more than that!

```clojure
(load "git@github.com:carpentry-org/http@master")

(def txt "POST / HTTP/1.1
Host: https://veitheller.de
User-Agent: curl/7.54.0
Accept: */*

hi!")

(defn main []
  (let [x (Request.parse txt)]
    (match x
      (Result.Success r) (println* &r)
      (Result.Error e)   (println* &e))))
```

<hr/>

Have fun!
