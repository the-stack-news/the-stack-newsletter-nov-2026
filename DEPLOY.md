# Publish The Stack — November 2026 to GitHub Pages

This folder is a **ready-to-push git repository**. One `git push` away from being live on the web.

## Folder contents

```
deploy/
  index.html            — the newsletter (served at / by GitHub Pages)
  assets/
    SAP-logo.png        — masthead logo
  DEPLOY.md             — this file
  .git/                 — local git repo, 1 commit staged
```

All asset paths inside `index.html` are relative. No `/Users/`, no `file://`.

The commit is attributed to a placeholder identity ("The Stack Newsletter <newsletter@local>") because no global git identity was set on this machine. If you want the commit under your own name, run this once before pushing:

```bash
cd "/Users/I540601/Report BTM 2/Newsletter November 2026/deploy"
git -c user.name="Your Name" -c user.email="you@example.com" commit --amend --reset-author --no-edit
```

---

## Step 1 — Create the public repo

Go to https://github.com/new and fill in:

| Field | Value |
| --- | --- |
| **Repository name** | `the-stack-newsletter-nov-2026` |
| **Visibility** | **Public** (required for free GitHub Pages) |
| Initialize with README / .gitignore / license | **No** — leave the repo empty |

Click **Create repository**.

## Step 2 — Push

GitHub's new-empty-repo page shows a "push an existing repository from the command line" box. From this folder:

```bash
cd "/Users/I540601/Report BTM 2/Newsletter November 2026/deploy"
git remote add origin https://github.com/<your-github-username>/the-stack-newsletter-nov-2026.git
git push -u origin main
```

(If you already set `user.name` / `user.email` globally on this machine, that first `git -c user.name=... commit --amend` step up-top isn't needed.)

**Prefer no CLI?** On the new empty-repo page, click *uploading an existing file*, then drag `index.html`, the `assets/` folder, and `DEPLOY.md` into the browser. Commit directly to `main`.

## Step 3 — Enable GitHub Pages

1. Go to the repo's **Settings → Pages**
2. Under **Build and deployment**:
   - **Source:** Deploy from a branch
   - **Branch:** `main` / `(root)`
3. Click **Save**
4. GitHub shows a yellow "building" banner for ~30–60 seconds, then a green "Your site is live at …" box

Your default URL will be:

```
https://<your-github-username>.github.io/the-stack-newsletter-nov-2026/
```

That URL exposes your GitHub username. If you want a branded URL that doesn't, see Step 4.

---

## Step 4 — (Optional) Custom branded domain

GitHub Pages supports your own domain for free, including free HTTPS via Let's Encrypt. You need a domain you control — this is purchased from a registrar (Namecheap, Cloudflare Registrar, Gandi, Google Domains / Squarespace, etc.), not from GitHub.

### Pick your URL shape

| Shape | Example | DNS record type you'll add | Notes |
| --- | --- | --- | --- |
| **Apex** (bare) | `thestack.example.com`'s apex, i.e. `example.com` | 4× **A** records + 4× **AAAA** records | Shortest; limited to one apex per domain |
| **Subdomain** (recommended) | `newsletter.example.com` | 1× **CNAME** record | Easiest to manage, works on any domain |

If you already use `example.com` for something else, use a subdomain. Subdomains are also faster to configure and don't force you to move the whole domain.

### 4a — Add a CNAME file to the repo

Create a file called `CNAME` (no extension, uppercase) at the repo root containing just your domain, one line, no `https://`:

```
newsletter.example.com
```

The quickest way:

```bash
cd "/Users/I540601/Report BTM 2/Newsletter November 2026/deploy"
echo "newsletter.example.com" > CNAME
git add CNAME
git commit -m "Add custom domain CNAME"
git push
```

Or just add it in the GitHub web UI: **Settings → Pages → Custom domain**, type the domain, click **Save**. GitHub creates the CNAME file for you.

### 4b — Add DNS records at your registrar

Open your registrar's DNS panel and add records.

**For a subdomain** (e.g. `newsletter.example.com`):

| Type | Host / Name | Value | TTL |
| --- | --- | --- | --- |
| CNAME | `newsletter` | `<your-github-username>.github.io.` (note the trailing dot) | 3600 (or auto) |

**For an apex** (e.g. `example.com`):

4× A records pointing `@` (or blank/apex) to these IPs:

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

4× AAAA records pointing `@` to:

```
2606:50c0:8000::153
2606:50c0:8001::153
2606:50c0:8002::153
2606:50c0:8003::153
```

(These are GitHub's documented Pages anycast IPs; they haven't changed in years but confirm them at https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site before publishing broadly.)

### 4c — Wait for DNS propagation

Usually 5–30 minutes. Verify with:

```bash
dig newsletter.example.com +short
# should return <your-github-username>.github.io
# then the four 185.199.* IPs
```

Or for an apex:

```bash
dig example.com +short
# should return the four 185.199.* IPs
```

### 4d — Verify in GitHub & enable HTTPS

Back in **Settings → Pages**:

1. Under **Custom domain**, your domain should now show a green checkmark ("DNS check successful").
2. Tick **Enforce HTTPS** — this is greyed out until GitHub finishes provisioning your Let's Encrypt cert (another 5–15 minutes after DNS resolves correctly). Come back and tick it as soon as it becomes enabled.

Once that's done the newsletter is live at `https://newsletter.example.com/` and the default `*.github.io` URL redirects to it.

### 4e — Hiding the username in the source

Even with a custom domain, the GitHub repo URL (`github.com/<username>/the-stack-newsletter-nov-2026`) is still public and discoverable. If you want to publish without ever exposing a personal username, two options:

- **Organization account.** Create a free GitHub Organization (e.g. `signavio-newsletter`), move the repo there, and the raw URL becomes `signavio-newsletter.github.io/...`. No personal username anywhere. The custom domain then points at the org's Pages.
- **Keep the repo under your account but make it look brand-neutral.** The custom domain is the only URL you share; the underlying GitHub repo is only discoverable if someone goes looking.

---

## Verify locally before publishing

```bash
cd "/Users/I540601/Report BTM 2/Newsletter November 2026/deploy"
python3 -m http.server 8000
# then open http://localhost:8000 in a browser
```

If the newsletter looks right at `http://localhost:8000` it will look right on GitHub Pages — the paths are identical.

## Known cosmetic note

The **Designing at the Summit** image is loaded from a SharePoint URL that requires SAP authentication. For public visitors it silently falls back to a soft blue-lavender gradient placeholder (this is existing in-page behavior, not something introduced by the deploy prep). All other content — the two editorial illustrations, article copy, buttons, Resources pills — renders identically for every visitor.
