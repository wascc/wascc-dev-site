+++
title = "Embedding Tokens"
linktitle = "Tokens"
date = 2017-12-03

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 60

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

**waSCC**'s decentralized authorization system using embedded JWTs is inspired by the decentralized authorization system used by the newest versions of the [NATS Server](https://docs.nats.io/nats-server/configuration/securing_nats/jwt).

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
| `wascap` | A field containing an object that holds all waSCC/wascap metadata |

## waSCC-Specific Token Metadata

| Field | Description |
|--|--|
| `name` | A human-friendly name for this entity |
| `prov` | Indicates whether this module is a _provider_ (true) or an _actor_ (false) |
| `hash` | A hash of the module's bytes as it existed prior to signature. This is used to verify that the module has not been altered since it was signed. |
| `caps` | List of capability attestations for actors, or the single capability for which a provider is responsible |
| `tags` | An arbitrary list of strings representing metadata that can be associated with the module |

## Sample Token

The following is a sample (serialized) JSON Web Token extracted from one of the sample modules. It's important to note that we can share _an entire token_ without ever risking the exposure of a secret. These tokens are verifiable in isolation and can only be created in the presence of secrets.

```raw
eyJ0eXAiOiJqd3QiLCJhbGciOiJFZDI1NTE5In0.eyJqdGkiOiJ5UThzbXJvYnRKcXBhdnNsQ1RGckdMIiwiaWF0IjoxNTg5NTY5ODM0LCJpc3MiOiJBQVNLQk4yU1RGQ1VETlhNSEFTNUpIWjJaQlFSMkw1R1kyWUFDTkhZQTRBNTdGVVVDUVVLQVZOMiIsInN1YiI6Ik1DSVhKVlhBWEtEWDdVRllERlcyNzM3U0hWSVJOWklMUzNVTE9ER0VRT1ZDVFdRN0hTR09IVVk3Iiwid2FzY2FwIjp7Im5hbWUiOiJHYW50cnkgQ2F0YWxvZyIsImhhc2giOiIiLCJ0YWdzIjpbXSwiY2FwcyI6WyJ3YXNjYzptZXNzYWdpbmciLCJ3YXNjYzpsb2dnaW5nIiwid2FzY2M6ZXh0cmFzIiwid2FzY2M6Z3JhcGhkYiJdLCJwcm92IjpmYWxzZX19.BAwORkrg2MDNYIoNPCeXm2NAkNcyFVngqCZgvSyVSIh2dqX3M89t9d798en5wcEpmndTZlW599UtPyDBj9m0BA
```

You can paste this token into a service like [jwt.io](https://jwt.io) and you'll see the underlying JSON structure (don't worry that the site says it has a bad signature... that website doesn't do ed25519 verification):

```json
{
  "jti": "yQ8smrobtJqpavslCTFrGL",
  "iat": 1589569834,
  "iss": "AASKBN2STFCUDNXMHAS5JHZ2ZBQR2L5GY2YACNHYA4A57FUUCQUKAVN2",
  "sub": "MCIXJVXAXKDX7UFYDFW2737SHVIRNZILS3ULODGEQOVCTWQ7HSGOHUY7",
  "wascap": {
    "name": "Gantry Catalog",
    "hash": "(base64 encoded hash)",
    "tags": [],
    "caps": [
      "wascc:messaging",
      "wascc:logging",
      "wascc:extras",
      "wascc:graphdb"
    ],
    "prov": false
  }
}
```

You can see here that the issuer (account public key) is `AASKBN2STFCUDNXMHAS5JHZ2ZBQR2L5GY2YACNHYA4A57FUUCQUKAVN2`, the unique module identity is `MBHRSJORBXAPRCALK6EKOBBCNAPMRTM6ODLXNLOV5TKPDMPXMTCMR4DW`, and that this module requires four different capabilities.

The `hash` field is generated at signing time and is used to verify that no one has altered the bytecode of the WebAssembly module since it was signed, giving us a truly immutable and portable build artifact.

You may also notice that account keys start with the `A` prefix while module (actor) keys start with the `M` prefix. These don't alter the raw binary value of the ed25519 keys, they just make it easier for humans to read.

## Further Reading

While actors are the only files in which we embed raw **JWT**s, the waSCC ecosystem makes use of these tokens in a number of different places. The following different entities all have JWTs:

* [Operators](../operators)
* [Accounts](../accounts)
* [Actors](../actors)

You can see how the tokens for all three entity types are used extensively within the [Gantry](../../gantry) registry application.
