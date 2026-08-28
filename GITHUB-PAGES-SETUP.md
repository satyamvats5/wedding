# Hosting the Wedding Invitation on GitHub Pages

Publish `wedding-invitation.html` as a live web page. Hosting and HTTPS are free.

The plan is in two parts. **Part 1** gets the site live at a free `github.io` URL — that's
all that's needed for now. **Part 2** puts it behind a custom domain later, without
disturbing anything set up in Part 1.

---

## Status

**Part 1 is complete. The site is live:**

### https://satyamvats5.github.io/wedding/

- [x] Git repo initialized, first commit authored as `satyamvats5@gmail.com`
      (configured **repo-local only**, in `.git/config` — global git config untouched)
- [x] `index.html` created as a copy of `wedding-invitation.html` (Pages serves
      `index.html` as the site root)
- [x] `.gitignore` added, excluding the unrelated diet-plan files and the working copy
- [x] Pushed to `github.com/satyamvats5/wedding` (public)
- [x] Pages enabled via API — `main` branch, `/` root, build status `built`
- [x] Verified: HTTP 200 over HTTPS, HSTS set, plain HTTP 301s to HTTPS, and the served
      bytes are identical to local `index.html`
- [x] `https_enforced` is `true`

### Content and behaviour

- [x] **RSVP form tested from the live URL** — submissions reach the Google Form correctly
- [x] **`noindex` added** — `robots` and `googlebot` meta tags set to `noindex, nofollow`,
      so search engines are asked not to list the page
- [x] **RSVP fallback contacts are real** — `whatsApp: "919748609947"` (India's `91`
      prefix is required; the code strips non-digits, so bare 10 digits would build a
      broken `wa.me` link) and `email: "satyamkumar13254@gmail.com"`. WhatsApp takes
      priority whenever the number is non-empty. This path fires when a Google Form
      submit *throws* — offline guest, flaky mobile data, ad-blocker — so before this fix
      those RSVPs opened a draft to `rsvp@example.com` and vanished.
- [x] **Language switcher is live on the cover**, not just after the doors open, so the
      invocation's script can be changed before reading it. Music still waits for the
      open, since audio needs a user gesture first.
- [x] **Invocation stays in Devanagari in all languages** — deliberate. `invocation.en`
      is intentionally identical to `invocation.hi`.

- [x] **`<!DOCTYPE html>` and `<meta charset="utf-8">` added** — the file had neither, so
      browsers rendered it in quirks mode and guessed the encoding. Worth eyeballing the
      Devanagari and Bangla text on a device that isn't yours.
- [x] **Discreet link preview** — `og:title` / `og:description` carry an emoji and no
      details. No `og:image` (would need a public, permanently-cached asset) and no
      `og:url` (so it survives a later move to a custom domain).
- [x] **Browser chrome tinted to match** — `theme-color` is dark for the cover, then the
      page's warm ground once the doors open. `requestFullscreen()` fires on the cover tap
      where allowed (Android/desktop; iPhone Safari has no element fullscreen, so it
      no-ops). `body.is-locked` uses `100dvh`, so the cover no longer overflows behind a
      mobile URL bar. True chrome-free viewing is not achievable — see "Fullscreen" below.
- [x] **Hero shows month + city only** — the exact day, weekday and time now arrive under
      the scratch card, so the reveal reveals something. The timeline still states the full
      date in plain text.
- [x] **`venues[].city` holds a city, not a street** — the street lives in `addr`, which
      also removed a duplication on the venue cards.
- [x] **CONFIG is the single source of truth for content** — the couple's names used to be
      hardcoded in the hero markup *as well as* in CONFIG. The markup is now empty and
      `renderStatic()` fills it, so a name is written in exactly one place.

Still open — content only:

- [ ] **Replace the template's placeholder details.** See the checklist below. This is the
      last thing standing between the site and being shareable; everything above is done.

Not defects, don't "fix" these:

- The three remaining `example.com` hits are `fEmailPh` — greyed-out hint text inside the
  email input, in each language. `you@example.com` is correct there.
- On the cover, toggling **en↔hi changes no verse text**, because English deliberately
  keeps the Devanagari. Only the "tap to open" label and the button's own label change.
  Switching to **Bangla does** change the verse. The switcher is working.

Part 2 (custom domain) is deferred and entirely optional.

---

## Content checklist — everything you need to edit

**All guest-facing content lives in one object: `const CONFIG`, roughly lines 992–1208 of
`wedding-invitation.html`.** Nothing needs to be changed outside it. The sections are
numbered in the source, so you can work straight down the list:

| § | Field | Currently holds |
|---|---|---|
| 0 | `invocation`, `heroInvocation` | Ganesha shloka — **keep as-is** (Devanagari in all languages, deliberate) |
| 1 | `couple.a.name` / `.parents` | **Aarav**, Shri Rajesh Sharma & Smt. Anita Sharma |
| 1 | `couple.b.name` / `.parents` | **Meera**, Shri Viren Iyer & Smt. Shaila Iyer |
| 1 | `couple.hashtag` | `#AaravWedsMeera` |
| 2 | `weddingAt` | `2026-12-13T08:00:00+05:30` — drives countdown, scratch card, calendar link |
| 3 | `timeline[]` | 5 sample events (Mehendi, Haldi, …) with dates and rooms |
| 4 | `venues[]` | 2 × The Leela Palace — `name`, `city`, `addr`, `maps`, `embed` |
| 5 | `preEvents[]` | Welcome Dinner, Ganesh Puja, … |
| 6 | `transport`, `gifts`, `rsvpBy` | sample text |
| 7 | `rsvp` | **already real** — Google Form ID, WhatsApp, email. Don't touch. |
| 8 | `assets.photo` | `demo_Img.jpg` placeholder — see note below |
| 8 | `assets.cover/hero/ganesha/song` | empty; art is drawn in canvas until you add files |

Three things to know while editing:

- **Every text field takes either a plain string or a `{ en, hi, bn }` object.** A plain
  string shows identically in all three languages — fine for names and places. Anything
  prose-like wants all three, or Hindi and Bangla guests see English.
- **`weddingAt` must stay ISO 8601 with the offset** (`+05:30`). It feeds the countdown and
  the "add to calendar" link, so a malformed value breaks both silently.
- **The photo wants ~1400×1050 (4:3 landscape).** The current placeholder is 236×353
  portrait, so the frame's `object-fit: cover` crops half its height and upscales it into a
  480px column. A 4:3 image drops in with no crop and no softness.

After editing, publish with the usual three steps:

```bash
cp wedding-invitation.html index.html
git add index.html && git commit -m "Real wedding details"
git push
```

---

# Part 1 — Get it live  ✅ done

Kept for reference / redoing from scratch.

## Step 1 — Authenticate the GitHub CLI

Run in your terminal:

```bash
gh auth login
```

Choose `GitHub.com` → `HTTPS` → authenticate in the browser. Verify with:

```bash
gh auth status
```

Make sure it logs in as the account tied to `satyamvats5@gmail.com`. If `gh` isn't
installed: `brew install gh`.

## Step 2 — Create the repo and push

```bash
cd /Users/satkumar/Downloads/Helms/SplitTest/5Aug
gh repo create wedding --public --source=. --push
```

`wedding` becomes part of the free URL, so pick a name you don't mind being visible.

### On repo visibility

The repo has to be **public** on a free GitHub account — Pages from a private repo
requires a paid plan (Pro, Team, or Enterprise).

More importantly: **the published site is public either way.** Repo visibility only
controls whether the source and commit history are browsable. Even on a paid plan with a
private repo, the rendered page is served to anyone who requests the URL; genuinely
access-controlled Pages sites are an Enterprise Cloud feature.

So the site is *unlisted, not private* — fine for an invitation, since only the people you
send the link to will know it exists. Two things follow:

- Don't put anything on the page you'd mind a stranger reading. **As of the RSVP fix, the
  page does carry real contact details** — a WhatsApp number and a Gmail address, both in
  the page source and in this public repo. That's a deliberate trade (guests need a
  fallback that works), but it's a real, permanent exposure: scrapers harvest `wa.me`
  links and `mailto:` addresses, and git history keeps them even if you later edit them
  out. Expect some spam on both.
- `noindex` is already in place — `robots` and `googlebot` meta tags:
  ```html
  <meta name="robots" content="noindex, nofollow">
  <meta name="googlebot" content="noindex, nofollow">
  ```
  It's a request to well-behaved crawlers, not enforcement. It does nothing about scrapers
  that ignore it, and nothing about the repo itself being public — so it is no substitute
  for leaving private details off the page.

## Step 3 — Enable Pages

1. Open `https://github.com/<username>/wedding/settings/pages`
2. **Build and deployment → Source**: `Deploy from a branch`
3. Branch `main`, folder `/ (root)`
4. **Save**

Live within a minute or two at:

```
https://<username>.github.io/wedding/
```

Open it and confirm it renders. Part 1 is done — this URL is shareable as-is, and it's
served over HTTPS.

## Editing the page after it's live

`index.html` is a *copy*, so editing `wedding-invitation.html` alone won't change the
site. After each edit:

```bash
cp wedding-invitation.html index.html
git add index.html
git commit -m "Update invitation"
git push
```

Pages redeploys automatically, usually in under a minute. (`wedding-invitation.html` is
gitignored so it doesn't clutter `git status` — `index.html` is the published file.)

If juggling two files gets annoying, just rename the original to `index.html` and edit
that directly.

---

# Part 2 — Custom domain (later)

Nothing here is required for the site to work. Everything from Part 1 keeps running, and
the `github.io` URL continues to work after you add a domain.

## Step 4 — Buy a domain

| Registrar | Notes |
|---|---|
| Cloudflare Registrar | Wholesale cost, no renewal markup. Requires using Cloudflare DNS. |
| Porkbun | Cheap, clean UI, free WHOIS privacy. |
| Namecheap | Widely used; check renewal price, often above year one. |

Roughly $10–15/yr for a `.com`. Novelty TLDs (`.wedding`, `.love`) run 3–5× that and can
renew higher still. Turn on WHOIS privacy and auto-renew.

## Step 5 — Tell GitHub the domain

**Settings → Pages → Custom domain**, enter the domain, save. This commits a `CNAME` file
containing that one line. Equivalent by hand:

```bash
echo "example.com" > CNAME
git add CNAME && git commit -m "Add custom domain" && git push
```

Keep the file — deleting it un-sets the domain and the custom URL breaks.

## Step 6 — DNS records

For an apex domain (`example.com`):

| Type  | Name / Host | Value                  |
|-------|-------------|------------------------|
| A     | `@`         | `185.199.108.153`      |
| A     | `@`         | `185.199.109.153`      |
| A     | `@`         | `185.199.110.153`      |
| A     | `@`         | `185.199.111.153`      |
| CNAME | `www`       | `<username>.github.io` |

Add all four A records — they're GitHub's load-balanced IPs. Some registrars express the
apex as a blank host or the full domain rather than `@`.

To use `www.example.com` as the primary address instead, the single `CNAME` suffices and
the A records can be skipped.

**On Cloudflare DNS:** set these to **DNS only** (grey cloud, not orange). The proxy
interferes with GitHub's certificate issuance.

## Step 7 — Verify and enforce HTTPS

```bash
dig +short example.com          # expect the four 185.199.x.153 addresses
dig +short www.example.com      # expect a chain to <username>.github.io
```

Once those resolve, go back to **Settings → Pages** and tick **Enforce HTTPS**. The
checkbox stays greyed out until GitHub issues the Let's Encrypt certificate — usually
minutes after DNS propagates, occasionally up to 24 hours. Don't skip it, or the site is
served over plain HTTP.

Optionally claim the domain under **Settings → Pages → verified domains** so nobody else
can point it at their own Pages site.

---

## Notes on this specific page

`wedding-invitation.html` is one self-contained file. Its only external references:

- **Google Fonts** (`fonts.googleapis.com`, `fonts.gstatic.com`) — load normally over HTTPS
- **Google Maps** embeds and search links — likewise fine

No local images, CSS, or JS to move, so `index.html` is the entire site.

### Fullscreen: why the browser bar can't be removed

The page asks for fullscreen on the cover tap, but "only the content, nothing else" is not
reachable, for three independent reasons:

- **Fullscreen requires a user gesture.** It can't be requested on page load, only from a
  tap. That's why the call sits on the cover tap, before any `await`, so it stays inside
  the gesture window.
- **iPhone Safari has no element fullscreen at all** — the API is video-only there. The
  call no-ops.
- **Most guests won't be in a browser.** Tapping a link inside WhatsApp opens WhatsApp's
  own webview, whose header no web API can touch. This is the real ceiling.

So the approach is to make the chrome *blend* rather than vanish: `theme-color` tints the
browser's toolbars to match the page (dark on the cover, warm afterwards). The only route
to genuine fullscreen is a PWA manifest plus "Add to Home Screen", which requires each
guest to install the invitation — not worth building.

### Doctype and charset — fixed, but worth understanding

The file used to begin directly with `<title>`: no `<!DOCTYPE html>`, no `<html>`/`<head>`
wrapper, no `<meta charset>`. It was written as an artifact *body*, which normally gets
wrapped automatically at publish time — but nothing wraps a standalone file on Pages.

Both lines are now in place at the top:

```html
<!DOCTYPE html>
<meta charset="utf-8">
```

What they fixed:

- **Quirks mode.** Without a doctype, browsers emulate legacy layout rules — `box-sizing`,
  percentage heights and `line-height` all compute differently than the CSS intends.
- **Guessed encoding.** Without a charset the browser infers one, and a wrong guess turns
  the Devanagari and Bangla text into mojibake.

Note the doctype is the one change here that can *move pixels*: leaving quirks mode is a
real rendering change. It's live and the layout was checked at 320, 390, 768 and 1280px
wide, but if anything ever looks subtly off in a way that predates your content edits, this
is the first thing to suspect.

Still missing, harmlessly: there's no `<html>` or `<head>` wrapper. Browsers imply both, so
it works — it's just not strictly valid HTML.

---

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| 404 at the `github.io` URL | File not named exactly `index.html`, not at repo root, or the push didn't land. Check the Actions tab for the Pages build. |
| Changes not showing | Edited `wedding-invitation.html` without re-copying to `index.html`. Or browser cache — hard-reload. |
| Old version stuck for hours | Check the Actions tab; a failed Pages build leaves the previous version live. |
| Custom domain "improperly configured" | DNS not propagated, or wrong A records. Wait, re-check with `dig`. |
| Certificate error / HTTPS unavailable | DNS not yet pointing at GitHub, or Cloudflare proxying is on. Fix, then wait for the cert. |
| Domain worked, then broke | The `CNAME` file was deleted by a push. Re-add it. |
| `gh` push rejected | Authenticated as the wrong account. `gh auth status`, then `gh auth switch`. |