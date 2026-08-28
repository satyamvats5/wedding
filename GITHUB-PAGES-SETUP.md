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

- [x] **RSVP form tested from the live URL** — submissions reach the Google Form correctly
- [x] **`noindex` added** — `robots` and `googlebot` meta tags set to `noindex, nofollow`,
      so search engines are asked not to list the page

Still open:

- [ ] **Replace the `example.com` RSVP placeholder emails** with the real address.
- [ ] **Consider adding `<!DOCTYPE html>` and `<meta charset="utf-8">`.** The file has
      neither, so browsers render it in *quirks mode* and guess the text encoding. See
      "Notes on this specific page" below.

Part 2 (custom domain) is deferred and entirely optional.

---

# Part 1 — Get it live  ✅ done

Kept for reference / redoing from scratch.

## Step 1 — Authenticate the GitHub CLI

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

- Don't put anything on the page you'd mind a stranger reading. Currently it's clean: the
  RSVP emails are `example.com` placeholders, and the only real identifiers are Google
  Form field IDs, which exist to receive submissions.
- To keep it out of search results, add this inside `<head>`:
  ```html
  <meta name="robots" content="noindex, nofollow">
  ```
  It's a request to crawlers, not enforcement — no substitute for leaving private details
  off the page.

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

### Missing doctype and charset

The file begins directly with `<title>` — there's no `<!DOCTYPE html>`, no `<html>`/`<head>`
wrapper, and no `<meta charset>`. It was written as an artifact *body*, which normally gets
wrapped automatically at publish time; served as a standalone file, nothing wraps it.

Browsers cope, but with two consequences:

- **Quirks mode.** Without a doctype, browsers emulate legacy layout rules — `box-sizing`,
  percentage heights, and `line-height` can all compute differently than intended.
- **Guessed encoding.** Without a charset declaration the browser infers one. The page uses
  Devanagari and Bangla text, so a wrong guess shows mojibake.

The fix is two lines at the very top:

```html
<!DOCTYPE html>
<meta charset="utf-8">
```

Worth doing, but test afterward: switching from quirks to standards mode can shift the
layout, which is exactly why it hasn't been applied automatically.

One thing to test on the live URL: the RSVP form submits to Google Forms, which is blocked
in some preview environments but should work from real hosting. Send a test RSVP once the
site is up and confirm it lands in the linked form's responses.

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