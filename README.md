# tadorne.org

The Tadorne website. Hugo, bilingual EN/FR, no theme dependency — every
layout lives in this repository. No JavaScript, no external requests at
runtime, no analytics, no cookies.

Built and verified against **Hugo 0.165.0 extended**. The extended build is
not required (there is no SCSS pipeline), but `hugo` must be recent enough
for the 0.146+ template layout used here (`layouts/_partials/`,
`layouts/_shortcodes/`, `layouts/_markup/`).

## Running it

```console
$ hugo server            # http://localhost:1313/  (FR at /fr/)
$ hugo --minify          # production build into public/
```

`public/` is not committed. Nothing else is generated.

## Structure

```
hugo.toml              site config, both languages, menus, GPG params
i18n/en.toml           UI strings (nav labels, "min read", footer headings)
i18n/fr.toml
content/en/            English content   → /
content/fr/            French content    → /fr/
assets/css/            tokens.css + fonts.css + main.css, concatenated by Hugo
static/fonts/          five self-hosted woff2 files
static/img/            logo and mark SVGs
static/favicon.svg     duck mark on the brand orange field
static/.well-known/    security.txt + the WKD placeholder for the GPG key
layouts/
  baseof.html          page shell
  home.html            identity block, pillars, project cards, commitments
  page.html            default single page (about, verify)
  projects/            section.html (card grid) + page.html (key figures header)
  log/                 section.html (dated list) + page.html (date, reading time)
  _partials/           head, header, footer, mark, langswitch, project-card
  _shortcodes/         callout
  _markup/             render-table.html — wraps tables so wide ones scroll
  robots.txt
```

### How the two languages are wired

Each language has its own `contentDir`, and French URLs are French
(`/fr/projets/`, `/fr/journal/`, `/fr/a-propos/`, `/fr/verifier/`). Because
the directory names differ between languages, pages are paired by an explicit
**`translationKey`** in front matter rather than by path. The language
switcher in the header uses that pairing to link to the translated
counterpart of the current page, falling back to the other language's home
page when no translation exists.

`hreflang` alternates and `og:locale:alternate` are emitted from the same
pairing, so they stay correct automatically.

## Adding content

### A new log post, in both languages

```console
$ hugo new content/en/log/my-note.md
$ hugo new content/fr/journal/ma-note.md
```

Both files need the **same `translationKey`** — that is the only thing tying
them together:

```yaml
---
title: "My note"
translationKey: log/my-note      # identical in both files
date: 2026-09-15
description: "One or two sentences. Used for the list page, the RSS item and og:description."
params:
  tags: ["penon"]
---
```

Reading time is computed by Hugo; the date is localised per language. The
section RSS feeds are at `/log/index.xml` and `/fr/journal/index.xml`.

### A new project, in both languages

Same pattern under `content/en/projects/` and `content/fr/projets/`, plus the
params the project card renders. Projects live on their own subdomain
(`penon.tadorne.org`, `vigie.tadorne.org`): the entry here is a card only —
`build.render: never` — and `params.external` carries the link, pointing at
that language's version of the project site.

```yaml
---
title: "Project name"
translationKey: projects/project-name
weight: 30                          # card order, home page and /projects/
description: "The one-paragraph summary shown on the card."
build:
  render: never                     # no page here, the project has its own site
  list: local                       # listed in the section, kept out of RSS
params:
  external: "https://project-name.tadorne.org/"
  status: "Design v1 complete"     # rendered as the orange status chip
  keyfigures:                       # rendered as the mono figures strip
    - label: "Wind speed"
      value: "±0.1 m/s"
---
```

The home page and `/projects/` both list every entry of the section, by
`weight`, through `layouts/_partials/project-card.html`. Without
`params.external` and the `build` block, an entry renders as a regular page
with `layouts/projects/page.html`.

### A new top-level section

Give its `_index.md` a `type` and a matching `cascade.type`, then add
`layouts/<type>/section.html` and `layouts/<type>/page.html`. There is
deliberately no generic `layouts/section.html` fallback.

### Writing conventions

The brand book (Tadorne brand book §8, kept outside this repository) is the authority. In short:
numbers first, engineer-to-engineer, no marketing adjectives, never first
person singular — "we" or the passive. French pages use proper French
typography, including **non-breaking spaces before `:` `;` `?` `!`** — these
are literal U+00A0 characters in the Markdown files, not entities.

## The GPG placeholders

No key exists yet. The placeholder fingerprint
`XXXX XXXX XXXX XXXX XXXX  XXXX XXXX XXXX XXXX XXXX` (two groups of five
blocks of four, separated by a double space) appears in exactly **four**
places. Replace all four:

| File | What to change |
|---|---|
| `hugo.toml` | `params.gpgFingerprint` — drives the site footer on every page |
| `content/en/verify.md` | the `<code class="fingerprint">` block, plus the sample `gpg --verify` output, plus the `{{< callout >}}` notice (delete the callout) |
| `content/fr/verifier.md` | the same three, in French |
| `static/.well-known/security.txt` | see below |

Then publish the key itself:

1. **WKD** — follow `static/.well-known/openpgpkey/README.md`. It needs an
   empty `policy` file and `hu/<zbase32>` containing the key in *binary*
   form. `gpg-wks-client --install-key` generates both. Delete that README
   once the real files are in place.
2. **DNS** — an `OPENPGPKEY` record (RFC 7929) plus a `TXT` record carrying
   the fingerprint, in the `tadorne.org` zone, **with DNSSEC enabled**. This
   is configured at the DNS provider, not in this repository, and it is the
   half that makes the identity independent of the web server.
3. **`security.txt`** — replace it with a clearsigned version
   (`gpg --clearsign`) and set a real `Expires`.

`content/*/verify.md` documents all of this for visitors; it needs no
changes beyond the fingerprint.

## Fonts

Self-hosted, no third-party CDN anywhere in the output. Five woff2 files in
`static/fonts/`, latin + latin-ext subsets, fetched from the
google-webfonts-helper API:

| Face | Weights | File |
|---|---|---|
| Space Grotesk | 500, 700 | `space-grotesk-v22-latin_latin-ext-{500,700}.woff2` |
| Inter | 400, 600 | `inter-v20-latin_latin-ext-{regular,600}.woff2` |
| JetBrains Mono | 400 | `jetbrains-mono-v24-latin_latin-ext-regular.woff2` |

`@font-face` declarations are in `assets/css/fonts.css`. Inter 400 and Space
Grotesk 700 are preloaded. All three families are SIL OFL.

A production build makes exactly **seven** requests for a content page: the
document, one stylesheet, and the five fonts. Verified in Chrome — zero
scripts, zero third-party origins.

## Design notes

`assets/css/tokens.css` is a **verbatim copy** of the brand's `tokens.css`. Do
not edit it here; change the brand file and copy it across, so the site never
drifts from the source of truth. `main.css` builds on top and adds only what
the tokens do not cover (footer surface, shadow, gutters).

Dark mode follows `prefers-color-scheme` and also honours a
`data-theme="light"` / `data-theme="dark"` attribute on `<html>`, so a theme
toggle can be added later without touching the palette. There is no toggle
today because it would require JavaScript for no real gain.

Prose is capped at `--td-measure` (68ch) using a CSS grid with a named `text`
column; tables, code blocks and figures break out to a wider `full` column,
and wide tables scroll inside their own `.table-wrap` box. Focus-visible
outlines use the accent colour; `prefers-reduced-motion` is respected.

## Deployment

`hugo --minify` produces a fully static `public/`, but **not** any static
host will do. Two hard requirements, both coming from WKD rather than from
the site itself:

1. **HTTPS on the apex domain, non-negotiable.** WKD is HTTPS-only; GnuPG
   does not fall back to HTTP. A site served over plain HTTP has no working
   key distribution at all.
2. **Control over response headers**, for the `Content-Type` and CORS rules
   below. Hosts offering a `_headers` file (Netlify, Cloudflare Pages) make
   this trivial; GitHub Pages and Codeberg Pages do not expose it.

> **Surge.sh is not suitable for `tadorne.org`.** Its free SSL covers only
> `*.surge.sh` subdomains; SSL on a custom domain requires the paid Surge
> Plus plan. Serving the apex over HTTP would break WKD and leave the
> `/verify/` page claiming something untrue. Surge remains perfectly good
> for a **preview at `tadorne.surge.sh`**, which does get HTTPS:
> `surge public tadorne.surge.sh` (set `baseURL` accordingly).

Detailed requirements:

- HTTPS with a valid certificate.
- `/.well-known/openpgpkey/hu/<zbase32>` served as
  `Content-Type: application/octet-stream`, ideally with
  `Access-Control-Allow-Origin: *`.
- No redirect of `/.well-known/` to another host — WKD clients follow the
  domain of the address.

For **GitHub Pages or Codeberg Pages**, publish `public/` and keep the
`CNAME` file pointing at `tadorne.org`. Note that both platforms serve
`/.well-known/` fine but you do not control response headers, so the WKD
content type may be wrong; if `gpg --locate-keys` fails there, self-host or
put a CDN in front. **DNSSEC and the `OPENPGPKEY` / `TXT` records are
configured at the DNS provider and are independent of the host.**

`baseURL` in `hugo.toml` is `https://tadorne.org/`; override it per
environment with `hugo --baseURL https://staging.example/`.

## Open items

- **GPG key created 2026-09-01** (`5420 08E3 53EC A628 B4D6  ACAD 8C03 243A
  FF69 5BF7`, expires 2031-08-31); all four fingerprint placeholders are
  filled in. Still pending: the WKD files (`hu/<zbase32>` + `policy`), the
  DNS `OPENPGPKEY` + `TXT` records with DNSSEC, and a clearsigned
  `security.txt` — see "The GPG placeholders" above.
- **Penon's licence is undefined.** Pick one before the first tagged
  release; the project pages now live on `penon.tadorne.org`, so that is
  where it gets stated.
- **The duck mark is a derived asset.** `brand/logo/tadorne-mark-ink.svg` is
  described as a one-path silhouette, but that path is only the *outer body
  boundary* of the logo — the head, bill and wing are separate paths — so it
  renders as a featureless circle at every size, unusable in a header or a
  favicon. The mark used here (`static/img/tadorne-mark-duck.svg`, inlined by
  `layouts/_partials/mark.html`, and `static/favicon.svg`) is the union of
  the duck shapes from `tadorne-logo-transparent.svg` minus the crème body.
  It reads as a shelduck down to 16 px and uses only the brand's own
  geometry, but it is **not** an approved brand file: it needs sign-off, and
  ideally `tadorne-mark-ink.svg` should be regenerated the same way so the
  PCB silkscreen mark stops being a circle too.
- There is no PNG favicon fallback; the SVG favicon covers every browser
  released in the last several years.

## Licences

- Content (`content/`, images): [CC BY-SA 4.0](LICENSE-CONTENT).
- Templates, CSS and configuration: [MIT](LICENSE).
- Fonts (`static/fonts/`): SIL Open Font License 1.1, owned by their authors.
