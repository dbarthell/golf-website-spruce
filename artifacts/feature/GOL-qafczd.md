---
id: GOL-qafczd
type: feature
title: Host on actual website
status: Planned
---
# Host on actual website

What are the steps I need to take to host the page on an actual website? Want to see the plan.

---

## Implementation Plan

### Current State

Your golf website is a **static site** (plain HTML, CSS, vanilla JS, and JSON data files) with zero build dependencies. This makes hosting extremely simple — the files just need to be served by any web server. There is no build step, no `package.json`, and no server-side logic.

**Files to deploy:**

```
index.html
css/styles.css
js/app.js
data/clubs.json
data/putting.json
```

---

### Approach: GitHub Pages (Free)

Since the repo already lives on GitHub (`dbarthell/golf-website`), GitHub Pages is the fastest path with zero cost. Every push to `main` will auto-deploy.

**Live URL:** `https://dbarthell.github.io/golf-website/`

---

### Steps

#### Step 1: Enable GitHub Pages

1. Go to repo **Settings → Pages**
2. Under "Source", select **Deploy from a branch**
3. Choose branch: `main`, folder: `/ (root)`
4. Click **Save**
5. Wait \~1 minute for the first deploy to complete
6. Verify the site is live at `https://dbarthell.github.io/golf-website/`

#### Step 2 (Optional): Add a Custom Domain

1. **Buy a domain** (\~$10-15/year from Namecheap, Cloudflare, Porkbun, etc.)
2. In repo **Settings → Pages → Custom domain**, enter your domain
3. **Add DNS records** at your registrar:

- **CNAME** record: `www` → `dbarthell.github.io`
- **A** records pointing to GitHub's IPs (GitHub will display these)

4. Check **Enforce HTTPS** (free SSL certificate from GitHub)
5. Wait for DNS propagation (5 min – 48 hours)

---

### What You Get

| Feature | Details |
|---------|---------|
| **Cost** | Free |
| **Auto-deploy** | Every push to `main` goes live automatically |
| **HTTPS** | Free SSL certificate included |
| **CDN** | GitHub's global CDN for fast load times |
| **Custom domain** | Supported (optional) |