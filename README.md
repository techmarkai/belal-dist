# belal-dist

Distribution artifacts for **Belal**, a prayer-table update system for ESP32
mosque relay controllers.

This repository contains **only** published release artifacts: manifests and
content-addressed BTRP payloads. It contains no source code, no device
firmware, and no configuration.

It is intentionally **public**: devices fetch from
`raw.githubusercontent.com` anonymously, over HTTPS, with no credential of any
kind. A private repository could not work — raw content for one requires a
token, and Belal firmware carries none.

## What a device verifies, and what it does not

A device accepts a release only after **all** of:

1. a SHA-256 for every payload, taken from the manifest
2. the structural validity of every BTRP table
3. a monotonic counter, rejecting replayed or downgraded releases
4. a payload path that encodes the same year, region and digest the manifest
   declares

> **There is no signature.** Belal does **not** cryptographically authenticate
> releases. SHA-256 binds the manifest to its payloads — that is **integrity**,
> not **authenticity**.

Ed25519 signing was removed from the architecture. Any `.sig` file or signing
language found in the history of this repository is obsolete; nothing verifies
it, and it must not be taken as evidence that a release is signed.

## This repository IS a trust anchor

This is the direct consequence of having no signature, and it is stated
plainly rather than buried:

**Whoever can push to this repository decides what every device installs.**
Write access must be restricted accordingly.

What still holds even against a hostile origin: no rollback (counter
monotonicity), no structurally invalid table, no cross-origin redirect, no
oversized transfer, and a device's currently installed table survives every
failed update.

HTTP metadata — `Date`, `ETag`, `Last-Modified`, CDN headers — is never
trusted for authenticity and is used only for caching.

## Layout

```
gaten/manifest.json                                   PRODUCTION release index
gaten/tables/<year>/<region>.<64-hex-sha256>.btrp     content-addressed payloads
```

The production raw base is:

```
https://raw.githubusercontent.com/techmarkai/belal-dist/master/gaten/
```

Payload paths embed the full SHA-256 of the file, so any change of content is
also a change of URL. A cached copy can never go stale against its own URL, and
a republished payload occupies a URL that was never fetched before.

A release is **Jordan-wide**: one manifest, one counter, every supported
region. A device downloads the manifest and then **only its own region's**
table.

## Current release

```
counter : 3
year    : 2026
regions : 10
```

Prayer times are the Ministry of Awqaf's **published local civil time for
Jordan (UTC+3), stored without conversion**. Counter 3 corrects counter 2,
which carried an inherited −60 minute transformation and therefore called every
prayer an hour early. Do not reintroduce any offset in the publisher or on the
device.

## `test/` is DEVELOPMENT ONLY

`test/` holds an older development release (counter 1) from the era before the
above corrections. It is not a production release and must not be treated as
one. It is retained only as a historical artifact of the distribution path.

## Publishing

Releases are built and verified with the publisher in the Belal source
repository, then uploaded here **manually**. There is no CI, no Actions
workflow, no Pages deployment and no CDN in this path.

```
python belal/tools/release.py build --year <YYYY> --shifted belal/data \
       --raw awqaf_<YYYY> --out <dist>
python belal/tools/verify_release.py --dist <dist>
```

`--raw` enables the time-basis guardrail, which aborts the build if the device
tables differ from the Ministry's published times by any offset at all.
