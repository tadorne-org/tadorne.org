---
title: "Verify"
translationKey: verify
url: /verify/
description: "Everything Tadorne publishes is signed with one GPG key anchored in DNS. Here is how to fetch it and check a signature."
params:
  eyebrow: "Authenticity"
---

Everything Tadorne publishes is signed with one ed25519 GPG key, and that key
is published in two independent places — the DNS zone for `tadorne.org`, under
DNSSEC, and this website under `/.well-known/`. An attacker would need to
compromise both.

## The key

| Field | Value |
|---|---|
| User ID | `Tadorne <contact@tadorne.org>` — primary, used for key lookup |
| Algorithm | ed25519 (EdDSA), signing + certification |
| Fingerprint | see below |

<code class="fingerprint">5420 08E3 53EC A628 B4D6  ACAD 8C03 243A FF69 5BF7</code>

## Fetching the key

### Via WKD (Web Key Directory)

The simplest route. GnuPG resolves the address to a URL under
`https://tadorne.org/.well-known/openpgpkey/` and fetches the key over TLS:

```console
$ gpg --locate-keys contact@tadorne.org
```

Look the key up by `contact@tadorne.org`, never by `tadorne@pm.me`. Both are
user IDs on this key, but only `tadorne.org` is a zone Tadorne controls: a WKD
lookup on the `pm.me` address is answered by the mail provider and returns a
different, unrelated key. A key obtained that way will not carry the
fingerprint above.

`--locate-keys` also imports the key. To fetch without importing, use
`gpg --locate-external-keys` and inspect the output first.

### Via DNS (OPENPGPKEY, RFC 7929)

The key is also published as an `OPENPGPKEY` record in the `tadorne.org` zone,
which is DNSSEC-signed. This path does not depend on the web server at all:

```console
$ gpg --auto-key-locate clear,dane --locate-keys contact@tadorne.org
```

To look at the record directly, with DNSSEC validation shown by the `ad`
flag in the response:

```console
$ dig +dnssec OPENPGPKEY \
    $(printf 'contact' | sha256sum | cut -c1-56)._openpgpkey.tadorne.org
```

A `TXT` record in the same zone carries the fingerprint in text form, as a
third cross-check.

### Always confirm the fingerprint

However the key arrives, compare its fingerprint against the value above
before relying on it:

```console
$ gpg --fingerprint contact@tadorne.org
```

## If a signature fails

A failed verification means one of three things: the file was modified in
transit, the key you hold is not the Tadorne key, or the published key has
been rotated. Re-fetch the key by both routes above, compare the two, and if
they disagree — or if either disagrees with the fingerprint on this page —
treat the download as compromised and report it to `contact@tadorne.org`.

Key rotation, revocation or compromise will be announced here and in the
[log](/log/), signed with the key being replaced.
