# koalaloop.games

The KoalaLoop Games website: studio home, plus a folder per game holding its landing page
and the legal pages Apple requires at submission. Plain static HTML, no build step, no
dependencies. Served by GitHub Pages from `main` at the repo root.

This is the single source of truth for every KoalaLoop web page. It is deliberately a
separate repo from the game repos (which are private) because one site serves many games.

## Layout

```
/                        index.html                 studio home
/onlydraws/              onlydraws/index.html       landing / ad destination
/onlydraws/privacy       onlydraws/privacy/         privacy policy
/onlydraws/terms         onlydraws/terms/           terms of use
/onlydraws/support       onlydraws/support/         support
/onlycars/               onlycars/index.html        landing (mirror, see below)
/onlycars/privacy        onlycars/privacy/
/onlycars/terms          onlycars/terms/
/onlycars/support        onlycars/support/
CNAME                                               custom domain for GitHub Pages
```

Every page is a folder with an `index.html`, so URLs resolve with **no `.html` suffix**.
That matches the paths hardcoded in the apps. Adding a new game means adding a sibling
folder; nothing else changes.

## URL map: these are compiled into shipped apps

Changing a path here breaks the links inside apps already installed on phones, and breaks
the URLs on file in App Store Connect. Treat them as fixed.

| URL                            | Consumed by                                     |
|--------------------------------|-------------------------------------------------|
| `/onlydraws/privacy`           | OnlyDraws `App/Sources/Store/StoreConfig.swift`  |
| `/onlydraws/terms`             | OnlyDraws `StoreConfig.swift`                    |
| `/onlydraws/support`           | OnlyDraws `StoreConfig.swift`                    |
| `/onlydraws/`                  | App Store Connect marketing URL, ads, bio        |

## OnlyCars is a mirror, not the live site

OnlyCars shipped before this studio site existed. Its live site is the separate public repo
`MartingaleProject2025/onlycars-app`, served at **onlycars-app.com**, and that is the URL
baked into the shipped app binary and registered in App Store Connect. **Do not retire it**:
copies of the app already on phones link there permanently.

The `/onlycars/` folder here is a copy so the studio home has somewhere to point. Because the
content is duplicated, `onlycars/index.html` carries `<link rel="canonical">` pointing at
`https://onlycars-app.com/` so search engines treat the original as authoritative and the two
copies do not compete.

Consequences worth knowing:

- Edits to the real OnlyCars site go to `onlycars-app`, then get copied here. This repo does
  not feed onlycars-app.com.
- The flat `privacy.html` links in the original were rewritten here to `/onlycars/privacy/`
  to match the studio folder convention.
- A third repo, `onlycars-legal`, holds a byte-identical duplicate of the same legal pages on
  a `github.io` URL. It is a leftover and nothing points at it.

## Deploying

Push to `main`. GitHub Pages rebuilds automatically, usually within a minute. There is no
cache to invalidate and no certificate to renew.

DNS lives in AWS Route 53 (the domain is registered there); it points at GitHub's Pages
servers. Repo Settings → Pages holds the custom domain and the "Enforce HTTPS" setting.

### If you ever point a new domain here

GitHub often does **not** issue the TLS certificate on its own once the DNS records land. It
sits on its default `*.github.io` certificate indefinitely, so `http://` works while
`https://` fails with `SSL: no alternative certificate subject name matches target hostname`.

Fix: clear the custom domain in Settings → Pages, save, re-enter it, save. The certificate
appears within a minute, and "Enforce HTTPS" becomes tickable. This cost real time on the
koalaloop.games launch.

Verify on the scheme consumers actually use, not whichever one answers first:

```
curl -I -L https://koalaloop.games/onlydraws/privacy      # must be 200, not an SSL error
openssl s_client -connect koalaloop.games:443 -servername koalaloop.games | head -3
```

A certificate whose CN is `*.github.io` means none was ever issued for the domain. Before
blaming GitHub, confirm the A/AAAA records resolve and that no CAA record blocks Let's
Encrypt (`dig +short CAA koalaloop.games`, empty is fine).

## Open items before the OnlyDraws App Store submission

1. **Support inbox.** Pages advertise `support@koalaloop.games`. That address needs mail
   forwarding configured before submission; a reviewer may email it. OnlyCars pages use
   `only.cars.app.ai@gmail.com`, which already works.
2. **App Store link.** In `onlydraws/index.html`, `APP_STORE_URL` is `""`, so the download
   button renders as a non-clickable "Coming soon" pill. Paste the real
   `https://apps.apple.com/app/idXXXXXXXXX?ct=promo` once the app has an ID. A host guard
   only trusts `apps.apple.com`, so a typo cannot ship a broken CTA. The `?ct=` token is an
   Apple campaign tag; vary it per ad channel.
3. **Effective date.** OnlyDraws legal pages read "July 20, 2026". Bump on material changes.
4. **Legal entity.** Pages use the trade name "KoalaLoop Games". If you incorporate, add the
   full entity name to the privacy and terms pages.

## Backlog (not blocking anything)

- **Em dashes in the OnlyCars copy.** House style is no em dashes anywhere, including
  marketing copy, but `onlycars/index.html` carries about ten of them in its title, og:title,
  and body prose. They were kept deliberately so this mirror stays byte-faithful to the live
  original at onlycars-app.com rather than having its published wording quietly reworded.
  Decide whether to strip them here (the two copies then diverge) or in both places at once.
  Check with `grep -n '—' onlycars/index.html`.

- **`onlycars-legal` is a dead repo.** It holds a byte-identical duplicate of the same three
  legal pages, served on a `github.io` URL. Nothing links to it: the live OnlyCars site
  serves its own copies from `onlycars-app`, and this repo now serves a third. Worth deleting
  or archiving, but confirm first that no old App Store Connect metadata still points at the
  `github.io` URL, since shipped app versions cannot be changed.

- **Whether to consolidate OnlyCars.** Right now onlycars-app.com is the real site and
  `/onlycars/` here is a mirror, which means two places to edit and a canonical tag holding
  the two apart. Consolidating would give one home for everything. It is deliberately not
  done: the shipped OnlyCars binary and its App Store Connect metadata point at
  onlycars-app.com, and copies already installed on phones link there permanently. If you do
  consolidate, do it during a normal app update and keep the old domain serving redirects
  rather than retiring it.

- **New games launch here from day one.** The decision from the koalaloop.games launch: every
  game after OnlyDraws gets a `/<product>/` folder in this repo instead of its own repo and
  domain. OnlyCars is the one exception, for the reason above.
