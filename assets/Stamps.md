# DNS Stamps

Server stamps encode all the parameters required to connect to a secure DNS server as a single string.
Think about stamps as QR code, but for DNS.

## Online DNS stamp calculator

Stamps can be quickly viewed/modified/created using this [VueJS component](https://github.com/jedisct1/vue-dnsstamp).

An online demo is accessible here: [https://stamps.dnscrypt.info](https://stamps.dnscrypt.info)

An example list of [public secure DNS resolvers](https://github.com/DNSCrypt/dnscrypt-resolvers/blob/master/v2/public-resolvers.md), with their DNS stamps.

## Common definitions

- `a || b` is the concatenation of `a` and `b`
- `a | b` is the result of the logical `OR` operation between `a` and `b`.
- `len(x)` is a single byte representation of the length of `x`, in bytes. Strings do not have to be zero-terminated.
- `vlen(x)` is equal to `len(x)` if `x` is the last element of a set, and `0x80 | len(x)` if there are more elements in the set.
- `LP(x)` is `len(x) || x`, i.e `x` prefixed by its length.
- `VLP(x1, x2, ...xn)` encodes a set, as `vlen(x1) || x1 || vlen(x2) || x2 ... || vlen(xn) || xn`. Since `vlen(xn) == len(xn)` (length of the last element doesn't have the high bit set), for a set with a single element, we have `VLP(x) == LP(x)`.
- `[ Q ]` means that `Q` is optional.
- `base64url(s)` is the URL-safe base64 encoding of `s`.

Encoding examples:

- `LP("10.0.0.1")` is `{ 0x08, 0x31, 0x30, 0x2e, 0x30, 0x2e, 0x30, 0x2e, 0x31 }`.
- `VLP("10.0.0.1", "10.0.0.2")` evaluates to `{ 0x88, 0x31, 0x30, 0x2e, 0x30, 0x2e, 0x30, 0x2e, 0x31, 0x08, 0x31, 0x30, 0x2e, 0x30, 0x2e, 0x30, 0x2e, 0x32 }`.
- `base64url(VLP("10.0.0.1", "10.0.0.2"))` is `iDEwLjAuMC4xCDEwLjAuMC4y`.

## DNSCrypt stamps

Format:

```text
"sdns://" || base64url(0x01 || props || LP(addr) || LP(pk || LP(providerName))
```

`0x01` is the protocol identifier for DNSCrypt.

`props` is a small-endian 64 bit value that represents informal properties about the resolver. It is a logical `OR` combination of the following values:

- `1`: the server supports DNSSEC
- `2`: the server doesn't keep logs
- `4`: the server doesn't intentionally block domains

For example, a server that supports DNSSEC, stores logs, but doesn't block anything on its own should set `props` as the following 8 bytes sequence: `{ 0x05, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 }`.

`addr` is the IP address, as a string, with a port number if the server is not accessible over the standard port for the protocol (443). IPv6 strings must be included in square brackets: `[fe80::6d6d:f72c:3ad:60b8]`.

`pk` is the DNSCrypt provider's Ed25519 public key, as 32 raw bytes.

`providerName` is the DNSCrypt provider name.

## DNS-over-HTTPS stamps

Format:

```text
"sdns://" || base64url(0x02 || props || LP(addr) || VLP(hash1, hash2, ...hashn) ||
                       LP(hostname) || LP(path)
                       [ || VLP(bootstrap_ip1, bootstrap_ip2, ...bootstrap_ipn) ])
```

`addr` is the IP address of the server. It can be an empty string, or just a port number.
In that case, the host name will be resolved to an IP address using another resolver.

`hashi` is the SHA256 digest of one of the TBS certificate found in the validation chain,
typically the certificate used to sign the resolver's certificate. Multiple hashes can
be provided for seamless rotations.

`hostname` is the server host name which will also be used as a SNI name.

`path` is the absolute URI path, such as `/dns-query`.

`bootstrap_ipi` is the IP address of a recommended resolver accessible over standard DNS
in order to resolve `hostname`. This is optional, and clients can ignore this information.

## DNS-over-TLS stamps

Format:

```text
"sdns://" || base64url(0x03 || props || VLP(addr) || VLP(hash1, hash2, ...hashn) ||
                       LP(hostname) ||
                       [ || vlen(bootstrap_ip) || bootstrap_ip ])
```

`addr` is the IP address of the server. It can be an empty string, or just a port number.
In that case, the host name will be resolved to an IP address using another resolver. IPv6 strings must be included in square brackets: `[fe80::6d6d:f72c:3ad:60b8]`.

`hashi` is the SHA256 digest of one of the TBS certificate found in the validation chain,
typically the certificate used to sign the resolver's certificate.  Multiple hashes can
be provided for seamless rotations.

`hostname` is the server host name which will also be used as a SNI name.

`bootstrap_ipi` is the IP address of a recommended resolver accessible over standard DNS
in order to resolve `hostname`. This is optional, and clients can ignore this information.

## Plain DNS stamps

Format:

```text
"sdns://" || base64url(0x00 || props || LP(addr))
```

`addr` is the IP address of the server. IPv6 strings must be included in square brackets: `[fe80::6d6d:f72c:3ad:60b8]`.

## Anonymized DNSCrypt relay stamps

Format:

```text
"sdns://" || base64url(0x81 || LP(addr))
```

`0x81` is the protocol identifier for a DNSCrypt relay.

`addr` is the IP address and port, as a string. IPv6 strings must be included in square brackets: `[fe80::6d6d:f72c:3ad:60b8]:443`.

## Implementations

- A [Go implementation of DNS stamps](https://github.com/jedisct1/go-dnsstamps)
- Another [Go implementation of DNS stamps](https://github.com/ameshkov/dnsstamps)
- A [.NET implementation of DNS stamps](https://github.com/bitbeans/DnsCrypt.Toolbox/tree/master/DnsCrypt.Stamps) by @bitbeans
- A [Rust implemenetation of DNS stamps](https://github.com/jedisct1/rust-dnsstamps/)
- A [Python implementation of DNS stamps](https://github.com/chrisss404/python-dnsstamps)
- A [Javascript implementation of DNS stamps](https://github.com/rs/node-dnsstamp)