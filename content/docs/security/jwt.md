+++
title = "Embedding Tokens"
linktitle = "Tokens"
date = 2017-12-03

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 2

[menu.docs]
  parent = "security"
  weight = 2
+++

Most of the time when we deploy applications to the cloud, whether we're using Kubernetes or some other workload scheduler, an application's security profile is fully or partially decoupled from the application. We see this as a risk and the ability to _embed_ custom sections into a WebAssembly file is our ticket to mitigating that risk.

waSCC creates a custom section inside each WebAssembly module called `jwt`. This custom section contains a JSON Web Token (JWT) that has been cryptographically signed with the `ed25519` signature method.

There are two very important aspects of this signed token that enable all sorts of additional levels of security within the waSCC runtime:

* We do **not** require knowledge of, or access to, any secrets in order to verify the signature of the token
* Within a signature-verified token, we can trust that the information has not been altered

Unlike other security systems such as [macaroons](https://research.google/pubs/pub41892/), since we don't need access to an original or "root" secret to perform verification, the tokens are _freestanding_ and can be verified _offline_ without access to any additional security infrastructure.

This is a useful attribute, but it becomes absolutely critical when we want to work within dynamically scaling infrastructure as well as partially-connected infrastructure that you might see in **IoT** solutions. Being able to verify tokens without accessing services/servers is also ideal for _edge computing_, where we want to avoid round-trips to the cloud to keep things running as fast as possible.

## Wascap

waSCC uses the [wascap](https://github.com/wascc/wascap) library to extract and validate tokens from WebAssembly modules. You can use the CLI that comes with that library to sign modules and inspect their tokens.

## Token Contents

The following are the fields used by waSCC for its JSON Web Tokens (JWTs). Most of the fields are part of the JWT standard.

| Field | Description |
|---|---|
| `exp` | When the token expires, stored in "seconds since epoch" as per the RFC |
| `jti` | JSON Token ID |
| `iat` | Time when the token was issued (seconds since epoch) |
| `iss` | Issuer of the token. This is the _public key_ of the _[account](../accounts)_ that signed the token |
| `sub` | Subject of the token. This is the _public key_ of the _[module](../modules)_ and should be treated as the globally unique identity of this WebAssembly module |
| `nbf` | Indicates the "not before" time when the module can be used (seconds since epoch) |
| `prov` | Indicates whether this module is a _provider_ (true) or an _actor_ (false) |
| `hash` | A hash of the module's bytes as it existed prior to signature. This is used to verify that the module has not been altered since it was signed. |
| `caps` | List of capability attestations for actors, or the single capability for which a provider is responsible |
| `tags` | An arbitrary list of strings representing metadata that can be associated with the module |

## Sample Token

The following is a sample (serialized) JSON Web Token extracted from one of the sample modules. It's important to note that we can share _an entire token_ without ever risking the exposure of a secret. These tokens are verifiable in isolation and can only be created in the presence of secrets.

```raw
eyJ0eXAiOiJqd3QiLCJhbGciOiJFZDI1NTE5In0.eyJqdGkiOiJDM3JzeWt2cHFxNFFpc1ZhQnd2MGU4IiwiaWF0IjoxNTc1NDkzMDE0LCJpc3MiOiJBQUdSVVhYVEdTUDRDMjdSV1BUTUNIQ0pGNTZKRDUzRVFQQTJSN1JQQzVWSTRFMjc0S1BSTU1KNSIsInN1YiI6Ik1CSFJTSk9SQlhBUFJDQUxLNkVLT0JCQ05BUE1SVE02T0RMWE5MT1Y1VEtQRE1QWE1UQ01SNERXIiwiaGFzaCI6IjM4NTgwQjhCRkY3RjdDNTlBN0I0RENERTc4MTA5N0QxNzIzN0UwMkU3N0M5MjVGMUU0OURDN0QyMEFCNThBNEEiLCJ0YWdzIjpbXSwiY2FwcyI6WyJ3YXNjYzptZXNzYWdpbmciXX0.LC_zuDTItYME0xMNU0Pa4M6Aj5y1loF3BZUbWeMgFH3f0uhV0pUN6ZSkiA4n4vyRDV0gSwARTsXfQrvmfd4fDw
```

You can paste this token into a service like [jwt.io](https://jwt.io) and you'll see the underlying JSON structure (don't worry that the site says it has a bad signature... that website doesn't do ed25519 verification):

```json
{
  "jti": "C3rsykvpqq4QisVaBwv0e8",
  "iat": 1575493014,
  "iss": "AAGRUXXTGSP4C27RWPTMCHCJF56JD53EQPA2R7RPC5VI4E274KPRMMJ5",
  "sub": "MBHRSJORBXAPRCALK6EKOBBCNAPMRTM6ODLXNLOV5TKPDMPXMTCMR4DW",
  "hash": "38580B8BFF7F7C59A7B4DCDE781097D17237E02E77C925F1E49DC7D20AB58A4A",
  "tags": [],
  "caps": [
    "wascc:messaging"
  ]
}
```

You can see here that the issuer (account public key) is `AAGRUXXTGSP4C27RWPTMCHCJF56JD53EQPA2R7RPC5VI4E274KPRMMJ5`, the unique module identity is `MBHRSJORBXAPRCALK6EKOBBCNAPMRTM6ODLXNLOV5TKPDMPXMTCMR4DW`, and that this module requires the `wascc:messaging` capability only.

The `hash` field is generated at signing time and is used to verify that no one has altered the bytecode of the WebAssembly module since it was signed, giving us a truly immutable and portable build artifact.

You may also notice that account keys start with the `A` prefix while module keys start with the `M` prefix. These don't alter the raw binary value of the ed25519 keys, they just make it easier for humans to read.
