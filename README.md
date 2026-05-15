# primerize.github.io

Static website for [Primerize](https://github.com/ribokit/Primerize), hosted on GitHub Pages. Migrated from `primerize.stanford.edu` (Django/MySQL on AWS EC2) in May 2026.

## Site structure

Plain HTML — no build step, no Jekyll, no framework.

```
index.html              # About / landing page
protocol/index.html     # Protocol (Sections 1–5)
code/index.html         # Code / download page
assets/
  css/
    bootstrap.min.css
    theme.css           # Custom navbar, colors, layout
    palette.css         # Label and panel color classes
    code.css            # Monospace / syntax highlight styles
  js/
    jquery.min.js
    bootstrap.min.js
  fonts/                # Glyphicon icon font (Bootstrap 3)
  images/
    docs/               # Protocol figures (JPG/SVG)
    ...                 # Site logo and background images
CNAME                   # Custom domain: primerize.stanford.edu
```

## Previewing locally

No build step needed. Serve any directory with a static file server so that absolute paths (`/assets/...`) resolve correctly.

**Python (simplest):**
```bash
cd primerize.github.io
python3 -m http.server 8000
# open http://localhost:8000
```

**Node (if you have it):**
```bash
npx serve .
```

Check all three pages:
- `http://localhost:8000/` — About
- `http://localhost:8000/protocol/` — Protocol (verify plate images and section anchors)
- `http://localhost:8000/code/` — Code

## How GitHub Pages is configured

1. Repo is at `github.com/DasLab/primerize.github.io`.
2. In **Settings → Pages**, source is set to the `main` branch, root directory.
3. The `CNAME` file in the repo root contains `primerize.stanford.edu`. GitHub Pages reads this to serve the custom domain.
4. In **Settings → Pages**, "Enforce HTTPS" is enabled once the DNS record is in place.

Pushes to `main` deploy automatically (usually within 1–2 minutes).

## DNS cutover (Stanford IT)

These steps point `primerize.stanford.edu` at GitHub Pages and decommission the AWS EC2 instance.

**Prerequisites:**
- Confirm the site looks correct at `https://daslab.github.io/primerize.github.io/` first.
- Confirm `CNAME` file contains exactly `primerize.stanford.edu` (no trailing newline issues).

**Steps:**

1. **Contact Stanford IT** at `ithelp@stanford.edu` (or use the same DNS management portal used for `daslab.stanford.edu`) and request:
   - Remove the existing `A` or `CNAME` record for `primerize.stanford.edu`
   - Add a new `CNAME` record: `primerize.stanford.edu` → `daslab.github.io`

2. **In GitHub Pages settings** (Settings → Pages on this repo):
   - Set "Custom domain" to `primerize.stanford.edu` and save.
   - Once DNS propagates, the "Enforce HTTPS" checkbox will become available — enable it.

3. **Verify** (DNS propagation can take minutes to hours):
   ```bash
   dig primerize.stanford.edu CNAME
   # Should return: primerize.stanford.edu → daslab.github.io
   curl -I https://primerize.stanford.edu/protocol/#IDT
   # Should return HTTP 200 with cert for primerize.stanford.edu
   ```
   Also check these URLs in a browser:
   - `https://primerize.stanford.edu/` — About page
   - `https://primerize.stanford.edu/protocol/#IDT` — scrolls to IDT anchor
   - `https://primerize.stanford.edu/protocol/#temp_design`
   - `https://primerize.stanford.edu/code/`

4. **Terminate the AWS EC2 instance** (IP `52.33.204.5`) after DNS is confirmed working. SSH key is `~/.ssh/amazon.pem`. Before terminating, make sure the archive in `aws_archive/` is uploaded to DasLab Google Drive (see `../aws_archive/README.md`).

## Making changes

Edit HTML/CSS/JS directly and push to `main`. No compilation needed.

The plate diagrams in `assets/images/docs/` are SVGs generated from JSON data in the [Server_Primerize](https://github.com/DasLab/Server_Primerize) repo (`media/images/docs/*.json`). If the protocol plate data ever needs updating, regenerate them with the Python snippet in `../aws_archive/README.md`.

## Key inbound links to preserve

The [Primerize documentation site](https://ribokit.github.io/Primerize/) links directly to these anchors — do not rename them:

| URL | Anchor in `protocol/index.html` |
|-----|--------------------------------|
| `/protocol/#temp_design` | DNA Template Design |
| `/protocol/#IDT` | IDT Oligo Ordering |
| `/protocol/#PCR` | PCR Assembly |
| `/protocol/#TX` | In Vitro Transcription |
| `/protocol/#par_prep` | Massively Parallel Preparation |
