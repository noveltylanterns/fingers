# fingers Protocol Draft

## 1. Purpose

`fingers` is a small plaintext request/response protocol inspired by Finger.

A `fingers` client connects to a server over TLS, sends one request line, receives a plaintext response, and the server closes the connection.

This specification defines only:

- transport
- TLS use
- request syntax
- URI mapping
- response framing
- reserved syntax

This specification does not define:

- backend behavior
- file layout
- CGI behavior
- response meanings
- account systems
- login systems
- required output formats beyond plaintext framing

## 2. Basic Model

A `fingers` transaction works like this:

1. The client connects to the URI authority host.
2. The client starts TLS immediately.
3. The client sends exactly one request line.
4. The server returns UTF-8 plaintext.
5. The server closes the connection.

There is no plaintext upgrade mode.

There is no multi-request session on one connection.

There are no protocol headers, status codes, or length fields.

## 3. URI Scheme

The URI scheme is `fingers://`.

The default port is `8179`.

A custom port may also be used.

Examples:

- `fingers://example.com`
- `fingers://example.com:8179`
- `fingers://example.com:9443`
- `fingers://example.com/alice`
- `fingers://example.net/example.com/alice`
- `fingers://example.org/example.net/example.com/alice?PLAN`

## 4. Authority Host and TLS Scope

The URI authority host is the only network endpoint for the request.

The client connects only to the authority host.

The authority host is the only TLS peer for the transaction.

The server certificate, if validated, is validated only against the authority host.

If client certificates are used, they are also scoped only to the authority host.

Everything after the authority host is request text only.

Example:

`fingers://relay.example/inner.example/alice`

In this URI:

- `relay.example` is the network endpoint
- `relay.example` is the TLS identity scope
- `inner.example` and `alice` are request components

## 5. TLS Requirements

TLS is required from the start of the connection.

A `fingers` client must not send a plaintext request first.

There is no STARTTLS mode.

This protocol is TLS-only.

This specification does not require use of any particular public certificate authority system.

Implementations may use:

- CA-signed certificates
- self-signed certificates
- locally trusted certificates
- pinned certificates
- private CA certificates

This allows use on public networks, local networks, private networks, bare IP addresses, and other hobbyist or experimental deployments.

The authority host identifies the TLS peer for the transaction. How that peer is trusted is implementation-defined.

A client or server may use public CA validation, private CA validation, certificate pinning, self-signed certificates, TOFU-style trust, or any other local trust model.

A server may request a client certificate during the TLS handshake.

A client may present a client certificate if it is configured to do so.

A client certificate may be CA-signed, self-signed, locally trusted, pinned, or otherwise accepted according to local policy.

If a client certificate is presented, the server may use it for implementation-defined identity-aware, personalized, authenticated, or session-like behavior.

This specification defines no cookies, tokens, or protocol-level login flow.

## 6. Character Encoding

All text defined by this specification is UTF-8.

This includes:

- request text
- response text

However, some fields are syntactically restricted to smaller character sets. **See Section 11.**

## 7. Request Syntax

A request is one line of text terminated by `<CRLF>`.

A request contains:

- zero or more flags
- optionally one target expression

Flags always come before the target expression.

The server reads one request line only.

Examples:

- empty request: `<CRLF>`
- target only: `alice<CRLF>`
- target with one relay: `alice@example.com<CRLF>`
- target with two relays: `alice@example.com@example.net<CRLF>`
- flag and target: `/PLAN alice<CRLF>`
- multiple flags and target w/ two relays: `/PLAN /mode=full alice@example.com@example.net<CRLF>`
- flags only: `/PLAN /index=users<CRLF>`

## 8. Request Terminator

Clients must terminate the request line with CRLF.

The request is exactly one line.

## 9. Response Framing

A `fingers` response is UTF-8 plaintext.

The response may be empty.

Response lines may use LF or CRLF.

Clients must accept either LF or CRLF in responses.

The end of the response is the closing of the connection.

This specification defines no:

- response headers
- status codes
- length fields
- terminator markers

## 10. URI Normalization

A trailing slash after the authority host does not change meaning.

These are equivalent:

- `fingers://example.com`
- `fingers://example.com/`

When a target is present, trailing slashes do not change meaning.

These are equivalent:

- `fingers://example.net/example.com/alice`
- `fingers://example.net/example.com/alice/`

Extra trailing slashes do not create extra empty path segments.

## 11. Allowed Characters

### 11.1 Authority Host

The authority host may contain only:

- letters
- digits
- dash (`-`)
- underscore (`_`)
- period (`.`)

Tilde (`~`) is not allowed in the authority host.

### 11.2 Port

If a port is present, it must be decimal digits only.

Example: `:8179`

### 11.3 Path Segments

Path segments may contain only:

- letters
- digits
- dash (`-`)
- underscore (`_`)
- period (`.`)
- tilde (`~`)

The at sign (`@`) is not valid inside an individual path segment. It is inserted only by the mapping rules in Section 13 when constructing the emitted target expression.

### 11.4 Flag Names

Flag names may contain only:

- letters
- digits
- dash (`-`)
- underscore (`_`)

### 11.5 Flag Values

Flag values may contain only:

- letters
- digits
- dash (`-`)
- underscore (`_`)

## 12. Percent-Encoding

Percent-encoding is not part of this specification.

The percent sign (`%`) is not valid in:

- authority hosts
- path segments
- flag names
- flag values

There is no percent-decoding step.

What appears in the URI is what is parsed.

## 13. Target and Relay Mapping

If no target path is present, the emitted request is empty.

If a target is present, the final path segment is the terminal target.

Any earlier path segments are relay hosts.

The authority host is included only when a target is present. In that case, it is always the outermost relay host.

The emitted target expression is built in this exact order:

1. start with the final path segment
2. append each earlier path segment from right to left, separated by `@`
3. append the authority host last, separated by `@`

In other words:

- the final path segment is first in the emitted target expression
- the authority host is last in the emitted target expression

Examples:

- `fingers://example.com` produces a request string of `<CRLF>`
- `fingers://example.com/alice` produces `alice<CRLF>`
- `fingers://example.net/example.com/alice` produces `alice@example.com<CRLF>`
- `fingers://example.org/example.net/example.com/alice` produces `alice@example.com@example.net<CRLF>`

The authority host is always the network endpoint and TLS peer.

The relay chain in the emitted request line does not change the actual network destination.

This specification defines only the syntax of this mapping.

A server may treat the resulting target expression as an actual relay request.

Another server may treat it as a local lookup key.

Both are valid.

## 14. Flags

A request may include flags.

Flags are modifiers placed before the target expression in the request line.

Two forms are allowed:

- bare flag
- variable flag

### 14.1 Bare Flag

A bare flag looks like `/FLAG`.

URI form: `?FLAG`

Example:

`fingers://example.com/alice?PLAN`

maps to:

`/PLAN alice<CRLF>`

### 14.2 Variable Flag

A variable flag looks like `/flag=value`.

URI form: `?flag=value`

Example:

`fingers://example.com/alice?mode=full`

maps to:

`/mode=full alice<CRLF>`

### 14.3 Multiple Flags

Multiple flags are written in the URI query by joining them with `&`.

Example:

`fingers://example.net/example.com/alice?PLAN&mode=full`

maps to:

`/PLAN /mode=full alice@example.com<CRLF>`

### 14.4 Duplicate Flags

If the same flag name appears more than once, only the first occurrence is kept.

Later occurrences are ignored.

Examples:

- `?PLAN&PLAN` keeps only `/PLAN`
- `?mode=full&mode=short` keeps only `/mode=full`

## 15. Reserved Flags

All single-character flags are reserved.

This includes all single letters and all single digits.

Implementations must not assign their own permanent meaning to single-character flags unless a future protocol revision allows it.

The flag `/W` is reserved for historical purposes and adapting legacy finger utilities.

This specification does not define a required meaning for `/W`.

If a server responds to `/W`, how it responds is implementation-defined.

Multi-character flags are implementation-defined unless a future protocol revision defines them.

## 16. Empty Requests and Flag-Only Requests

An empty request is valid.

Examples:

- `fingers://example.com`
- `fingers://example.com/`

Both map to `<CRLF>`

A flag-only request is also valid.

Example:

`fingers://example.com?PLAN`

maps to:

`/PLAN<CRLF>`

The meaning of empty requests and flag-only requests is implementation-defined.

A server may return:

- an index
- a menu
- a help page
- generated text
- an error
- nothing

All are valid.

## 17. Implementation Freedom

This protocol only defines the externally visible syntax and transport behavior.

A server may produce response text from:

- plain files
- generated files
- CGI
- scripts
- databases
- templates
- legacy Finger data
- any other local mechanism

This specification does not require any particular storage model or backend design.

These rules also apply to malformed, unsupported, or unsuccessful requests.

Since the protocol has no error format and all responses are plaintext, how the server handles them is beyond the scope of this specification.

A server may:

- return ordinary plaintext output
- return a not-found style response
- return an error message
- close the connection without a response

This specification defines no protocol-level error format or fixed maximum lengths for requests or responses.

Any such limits are implementation-defined.

## 18. Examples

### Empty request

URI: `fingers://example.com`

Request sent: `<CRLF>`

### Simple request

URI: `fingers://example.com/alice`

Request sent: `alice<CRLF>`

### One inner relay

URI: `fingers://example.net/example.com/alice`

Request sent: `alice@example.com<CRLF>`

### Two inner relays

URI: `fingers://example.org/example.net/example.com/alice`

Request sent: `alice@example.com@example.net<CRLF>`

### Bare flag

URI: `fingers://example.com/alice?PLAN`

Request sent: `/PLAN alice<CRLF>`

### Variable flag

URI: `fingers://example.com/alice?mode=full`

Request sent: `/mode=full alice<CRLF>`

### Multiple flags with relay target

URI: `fingers://example.net/example.com/alice?PLAN&mode=full`

Request sent: `/PLAN /mode=full alice@example.com<CRLF>`

### Mapping pattern

General pattern:

`fingers://host/relayN/.../relay2/relay1/target?flag=var`

maps to:

`/flag=var target@relay1@relay2@...@relayN<CRLF>`

## 19. Summary

`fingers` is:

- TLS-only
- one request per connection
- one plaintext response per connection
- UTF-8 throughout
- simple URI mapping
- syntax-focused
- backend-agnostic

It does not standardize application behavior beyond the wire format.
