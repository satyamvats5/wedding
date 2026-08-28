# Hosting the Wedding Invitation on GitHub Pages

Publish `wedding-invitation.html` at a custom domain. Total cost: ~$10–15/yr for the
domain; hosting and HTTPS are free.

**Time:** ~15 minutes of work, plus up to a few hours of waiting for DNS and the TLS
certificate.

---

## Prerequisites

- A GitHub account.
- `git` and the GitHub CLI (`gh`). Check with:
  ```bash
  git --version && gh auth status
  ```
  If `gh` isn't installed: `brew install gh`, then `gh auth login`.

---

## Step 1 — Buy a domain

Any registrar works. Good options:

| Registrar | Notes |
|---|---|
| Cloudflare Registrar | Sold at wholesale cost, no renewal markup. Requires moving DNS to Cloudflare. |
| Porkbun | Cheap, clean UI, free WHOIS privacy. |
| Namecheap | Widely used; watch for higher renewal pricing than year one. |

Notes:
- `.com` is the cheapest and most trusted. Novelty TLDs (`.wedding`, `.love`) often cost
  3–5× more and can renew even higher.
- Enable WHOIS privacy (usually free) so your name and address aren't in a public registry.
- Set auto-renew on, or the site goes dark a year from now.

You can skip this step entirely and stop after Step 3 — the free
`https://<username>.github.io/<repo>/` URL works fine on its own.

---

## Step 2 — Create the repository

GitHub Pages serves `index.html` as the root of the site, so the invitation gets copied
to that name.

```bash
cd /Users/satkumar/Downloads/Helms/SplitTest/5Aug

git init
cp wedding-invitation.html index.html
git add index.html
git commit -m "Wedding invitation"
```

Then create the remote and push:

```bash
gh repo create wedding --public --source=. --push
```

Replace `wedding` with whatever repo name you want — it appears in the free
`github.io` URL, but not in the custom domain URL.

### Two things to know about the repo

- **It must be public.** GitHub Pages on private repos requires a paid plan. The repo is
  the site's source, so treat everything in it as published.
- **Nothing sensitive on the page.** Since it's world-readable, keep home addresses,
  phone numbers, and guest lists off it. Anyone who guesses or is forwarded the URL can
  read it, and search engines may index it. To discourage indexing, add this inside
  `<head>` of `index.html`:
  ```html
  <meta name="robots" content="noindex, nofollow">
  ```
  This is a request, not enforcement — it is not a substitute for leaving private
  details off the page.

### Keeping edits in sync

`index.html` is now a *copy*. When you change `wedding-invitation.html`, re-copy and push:

```bash
cp wedding-invitation.html index.html
git add index.html
git commit -m "Update invitation"
git push
```

Alternatively, drop the copy and rename the original to `index.html` so there's only one
file to edit.

---

## Step 3 — Enable GitHub Pages

1. Go to `https://github.com/<username>/wedding/settings/pages`
2. Under **Build and deployment → Source**, choose **Deploy from a branch**
3. Branch: `main`, folder: `/ (root)`
4. **Save**

Within a minute or two the site is live at:

```
https://<username>.github.io/wedding/
```

Load it in a browser and confirm it renders before moving on. If you get a 404, check
that the file is named exactly `index.html` at the repo root and that the push landed
(`git log origin/main -1`).

---

## Step 4 — Point the domain at Pages

### 4a. Tell GitHub about the domain

In **Settings → Pages → Custom domain**, enter your domain (e.g. `example.com`) and save.
This commits a file named `CNAME` to the repo containing that one line. You can also
create the file by hand:

```bash
echo "example.com" > CNAME
git add CNAME && git commit -m "Add custom domain" && git push
```

Keep the file — deleting it un-sets the custom domain.

### 4b. Add DNS records at your registrar

For an apex domain (`example.com`), create four `A` records plus a `www` alias:

| Type  | Name / Host | Value             |
|-------|-------------|-------------------|
| A     | `@`         | `185.199.108.153` |
| A     | `@`         | `185.199.109.153` |
| A     | `@`         | `185.199.110.153` |
| A     | `@`         | `185.199.111.153` |
| CNAME | `www`       | `<username>.github.io` |

The four A records are GitHub's load-balanced IPs — add all of them. Some registrars
write the apex as blank or as the domain itself instead of `@`.

If you'd rather use `www.example.com` as the primary address, the single `CNAME` record
is enough and you can skip the A records.

**Using Cloudflare DNS?** Set these records to **DNS only** (grey cloud, not orange).
Cloudflare's proxy interferes with GitHub's certificate issuance.

### 4c. Verify and enforce HTTPS

Check that DNS has propagated:

```bash
dig +short example.com
dig +short www.example.com
```

The first should return the four `185.199.x.153` addresses; the second should chain to
`<username>.github.io`.

Then return to **Settings → Pages** and tick **Enforce HTTPS**. The checkbox stays
greyed out until GitHub finishes issuing the Let's Encrypt certificate — usually a few
minutes after DNS resolves, occasionally up to 24 hours. Don't skip this; without it
the site is served over plain HTTP.

Optionally, claim the domain under **Settings → Pages → verified domains** to prevent
anyone else from pointing the same domain at their own Pages site.

---

## Notes on this specific page

`wedding-invitation.html` is a single self-contained file. Its only external references are:

- **Google Fonts** (`fonts.googleapis.com`, `fonts.gstatic.com`) — loads normally over
  HTTPS on a live site.
- **Google Maps** embeds and search links — likewise fine.

There are no local images, CSS, or JS files to move, so nothing else needs to be added
to the repo.

---

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| 404 at the `github.io` URL | File isn't named `index.html`, isn't at the repo root, or the push didn't land. Check the Actions tab for the Pages build. |
| Custom domain shows "improperly configured" | DNS hasn't propagated yet, or the A records are wrong. Wait, then re-check with `dig`. |
| Certificate error / HTTPS unavailable | DNS not yet resolving to GitHub, or Cloudflare proxying is on. Fix DNS, then wait for the cert. |
| Domain worked, then broke after an edit | The `CNAME` file was deleted. Re-add it. |
| Changes not showing | Browser cache — hard-reload. Or you edited `wedding-invitation.html` without re-copying to `index.html`. |
| Old version stuck for hours | Check the Actions tab; a failed Pages build leaves the previous version live. |