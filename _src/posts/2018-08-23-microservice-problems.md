    Title: Microservice problems
    Date: 2018-08-23T17:24:20
    Tags: DRAFT,architecture

- service discovery
- alerting / monitoring
- load distribution, including:
  - fallbacks & retries
  - caching
  - grouped requests
- metrics & analytics
- cross language clients
- versioning of clients

The last two points are hiding a coupling between client and service that was to
be avoided with the microservice architecture at all.

Can this be overcome with a pattern "Everything is a REPL"?
