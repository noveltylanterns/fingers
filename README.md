# fingers

A small TLS-only plaintext request/response protocol inspired by [Finger](https://en.wikipedia.org/wiki/Finger_(protocol)).

You can read a working draft of the `fingers://` specification at [fingers-protocol-draft.md](fingers-protocol-draft.md).


## Status

This is an early draft.

The protocol is intentionally small and syntax-focused. It defines transport, request syntax, URI mapping, and response framing. It does not require any specific backend model.


## Core ideas

- mandatory TLS from connection start
- one request per connection
- one plaintext response per connection
- UTF-8 text
- no headers
- no status codes
- no length fields
- URI scheme: `fingers://`
- default port: `8179`


## Scope

This draft standardizes only the protocol surface:

- transport and TLS behavior
- request-line syntax
- URI mapping
- response framing
- reserved syntax space

It does not standardize server internals. A conforming daemon may use plain files, scripts, CGI, templates, databases, or any other local mechanism.


## Notes

- The authority host is the only network endpoint and TLS identity.
- Additional path segments are request components only.
- Optional client certificates may be used for implementation-defined identity-aware behavior.

## Example

URI:

```text
fingers://example.com/alice?PLAN&mode=full
```

Request sent:

```text
/PLAN /mode=full alice<CRLF>
```

## Development

This draft is meant to be simple and easy to evolve. Much like finger's [RFC 1288](https://datatracker.ietf.org/doc/html/rfc1288), the goal here is a small, readable protocol document that people can independently implement without needing a large software stack.
