# fingers

A small TLS-only plaintext request/response protocol inspired by [Finger](https://en.wikipedia.org/wiki/Finger_(protocol)). The concept of Finger has great potential as a low-bandwidth API endpoint. We want to secure it and give it some room to breathe, while making it easy to import nearly 50 years of forgotten Finger-based utilities.

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
- default port: `:8179`


## Scope

This draft standardizes only the protocol surface:

- transport and TLS behavior
- request-line syntax
- URI mapping
- response framing
- reserved syntax space

It does not standardize server internals. A conforming daemon may use plain files, scripts, CGI, templates, databases, or any other local mechanism.


## Examples

This spec is intended to enable Finger-style passing of encrypted plaintext data around like this:

#### Command:

```text
fingers user@hostname.com
fingers /FLAG /flag2=variable user@hostname.com
```

#### URI:

```text
fingers://hostname.com/user
fingers://hostname.com/user?FLAG&flag2=variable
```

#### Request Text:

```text
user<CRLF>
/FLAG /flag2=variable user<CRLF>
```

For more information, read the [Draft Specification](fingers-protocol-draft.md).


## Development

This draft is meant to be simple and easy to evolve. Much like Finger's [RFC 1288](https://datatracker.ietf.org/doc/html/rfc1288), the goal here is a small, readable protocol document that people can independently implement without needing a large software stack.

The [fingered](https://github.com/noveltylanterns/fingered) application, on top of functioning as a traditional `finger://` daemon, is also to be considered the Reference Implementation of the `fingers://` protocol, and is equally a work-in-progress.

