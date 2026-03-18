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
- `fingers://example.com/user`
- `fingers://example.com/host1/user?PLAN`

## 4. Authority Host and TLS Scope

The URI authority host is the only network endpoint for the request.

The client connects only to the authority host.

The authority host is the only TLS peer for the transaction.

The server certificate, if validated, is validated only against the authority host.

If client certificates are used, they are also scoped only to the authority host.

Path segments do not identify additional network hosts. They are only request components.

Example:

`fingers://example.com/host2/host1/target`

In this URI:
- `example.com` is the network endpoint
- `example.com` is the TLS identity scope
- `host2`, `host1`, and `target` are request components only

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

The authority host identifies the TLS peer for the transaction.

How that peer is trusted is implementation-defined.

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

However, some fields are syntactically restricted to smaller character sets. See **Section 11** for more information.

## 7. Request Syntax

A request is one line of text terminated by CRLF.

A request contains:

- zero or more flags
- optionally one target

Flags always come before the target.

The server reads one request line only.

Examples:

- empty request: `<CRLF>`
- target only: `user<CRLF>`
- flag and target: `/PLAN user<CRLF>`
- multiple flags and target: `/PLAN /mode=full user<CRLF>`
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

Trailing slashes do not change meaning.

These are equivalent:

- `fingers://example.com`
- `fingers://example.com/`

These are also equivalent:

- `fingers://example.com/target`
- `fingers://example.com/target/`

Likewise, extra trailing slashes do not create extra empty path segments.

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

## 13. Path and Target Mapping

The path is interpreted positionally.

The final path segment is the target.

Any earlier path segments are emitted before it using `@` syntax.

Path segments are emitted in reverse order using @ syntax to preserve compatibility with legacy Finger forwarding conventions, where the chain reads right-to-left (e.g., user@host1@host2):
- `fingers://example.com` maps to `<CRLF>`
- `fingers://example.com/user` maps to `user<CRLF>`
- `fingers://example.com/host1/user` maps to `user@host1<CRLF>`
- `fingers://example.com/host2/host1/user` maps to `user@host1@host2<CRLF>`

This specification assigns no required meaning to those earlier segments beyond syntax. A server may interpret that text however it wants. One server may treat it as a forwarding chain. Another may treat it as a local taxonomy or lookup key. Both are valid.

## 14. Flags

A request may include flags.

Flags are modifiers placed before the target in the request line.

Two forms are allowed:

- bare flag
- variable flag

### 14.1 Bare Flag

A bare flag looks like `/FLAG`.

URI form: `?FLAG`

Example:

`fingers://example.com/user?PLAN`

maps to:

`/PLAN user<CRLF>`

### 14.2 Variable Flag

A variable flag looks like `/flag=value`.

URI form: `?flag=value`

Example:

`fingers://example.com/user?mode=full`

maps to:

`/mode=full user<CRLF>`

### 14.3 Multiple Flags

Multiple flags are written in the URI query by joining them with `&`.

Example:

`fingers://example.com/user?PLAN&mode=full`

maps to:

`/PLAN /mode=full user<CRLF>`

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

Both map to `<CRLF>`.

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

## 17. Errors and Unsupported Input

Handling of malformed, unsupported, or unsuccessful requests is implementation-defined.

A server may:

- return ordinary plaintext output
- return a not-found style response
- return an error message
- close the connection without a response

This specification defines no protocol-level error format.

## 18. Limits

This specification does not define fixed maximum lengths for:

- requests
- targets
- flags
- path depth
- responses

Any such limits are implementation-defined.

A server may reject requests that exceed local limits.

## 19. Implementation Freedom

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

## 20. Examples

### Empty request

URI: `fingers://example.com`

Request sent: `<CRLF>`

### Target request

URI: `fingers://example.com/alice`

Request sent: `alice<CRLF>`

### Taxonomy-style path

URI: `fingers://example.com/people/alice`

Request sent: `alice@people<CRLF>`

### Multi-segment path

URI: `fingers://example.com/host2/host1/alice`

Request sent: `alice@host1@host2<CRLF>`

### Bare flag

URI: `fingers://example.com/alice?PLAN`

Request sent: `/PLAN alice<CRLF>`

### Variable flag

URI: `fingers://example.com/alice?mode=full`

Request sent: `/mode=full alice<CRLF>`

### Multiple flags

URI: `fingers://example.com/alice?PLAN&mode=full`

Request sent: `/PLAN /mode=full alice<CRLF>`

## 21. Summary

`fingers` is:

- TLS-only
- one request per connection
- one plaintext response per connection
- UTF-8 throughout
- simple URI mapping
- syntax-focused
- backend-agnostic

It does not standardize application behavior beyond the wire format.
