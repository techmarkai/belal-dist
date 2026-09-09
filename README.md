# belal-dist

Distribution artifacts for **Belal**, a signed prayer-table update system for
ESP32 mosque relay controllers.

This repository contains **only** published release artifacts: signed
manifests, their detached signatures, and content-addressed BTRP payloads.
It contains no source code, no device firmware, and no configuration.

## This repository is not a trust anchor

A Belal device authenticates a release **exclusively** through an Ed25519
public key compiled into its firmware. It verifies:

1. the Ed25519 signature over the exact bytes of `manifest.json`
2. a SHA-256 for every payload, taken from that verified manifest
3. the structural validity of every BTRP table
4. a monotonic counter, to reject replayed or downgraded releases

GitHub and Cloudflare are **distribution and availability** layers. Nothing
served from here is trusted because of where it came from. Neither host can
forge a release, and neither can be used to authenticate one. HTTP metadata —
`Date`, `ETag`, `Last-Modified`, CDN headers — is never trusted for
authenticity and is used only for caching.

## Layout

```
test/manifest.json                                   signed manifest
test/manifest.sig                                    64-byte Ed25519 signature
test/tables/<year>/<region>.<64-hex-sha256>.btrp     content-addressed payloads
```

Payload paths embed the full SHA-256 of the file, so any change of content is
also a change of URL. Cached copies can never go stale against their own URL.

## `test/` is DEVELOPMENT ONLY

Everything currently published lives under `test/` and is signed with a
**development key**. Production firmware does not carry that key and will
**reject** these artifacts. They exist to validate the distribution path
end to end and must never be treated as a production release.

No production release has been published. The production signing-key ceremony
has not been performed.
