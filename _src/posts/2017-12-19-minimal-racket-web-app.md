    Title: Minimal Racket web app
    Date: 2017-12-19T20:15:41
    Tags: racket, web

This is the simplest module that responds to HTTP requests.

```racket
#lang racket/base

(require
 (only-in web-server/http
          response/full)
 (only-in web-server/http/request-structs
          make-header)
 (only-in web-server/servlet-env
          serve/servlet))

;; hello: request? -> response?
(define (hello req)
  ;; struct response/full
  (response/full
   200                          ;; code
   (string->bytes/utf-8 "OK")   ;; message
   (current-seconds)            ;; timestamp in s
   #f                           ;; mime or #f
   (list (make-header           ;; list of headers
          (string->bytes/utf-8 "Server")
          (string->bytes/utf-8 "Racket")))
   (list                        ;; body: list of bytes
     (string->bytes/utf-8 "Hello world!\n"))))

(module+ main
  (serve/servlet
   hello
   #:port 8080
   #:servlet-regexp #rx""))
```

Save it as `hello.rkt` and run it with `$ racket hello.rkt`

Check that it works:

```shell
$ curl -v http://localhost:8080/
*   Trying ::1...
* TCP_NODELAY set
* Connection failed
* connect to ::1 port 8080 failed: Connection refused
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Tue, 19 Dec 2017 18:13:14 GMT
< Last-Modified: Tue, 19 Dec 2017 18:13:14 GMT
< Content-Length: 13
< Server: Racket
<
Hello world!
* Connection #0 to host localhost left intact
```

