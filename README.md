# oregonheals.org

Static site for the Oregon Heals Coalition. Two pages:

- `/` — coalition landing page (About / Mission / Members / Newsletter)
- `/summit/` — Inaugural Oregon BIPOC Mental Health Summit (August 22, 2026)

Built as flat HTML + CSS to deploy on GitHub Pages with no build step.

## Repo layout

```
.
├── index.html              # Coalition landing page
├── summit/
│   └── index.html          # Summit landing page
├── 404.html                # Custom 404 page
├── assets/
│   ├── css/main.css        # Shared styles for both pages
│   ├── fonts/
│   │   ├── fonts.css       # @font-face declarations
│   │   └── *.woff          # Brandon Grotesque + New Kansas (Latin subset)
│   └── img/                # Logos as SVG
├── CNAME                   # oregonheals.org
├── .nojekyll               # Tell GH Pages to skip Jekyll processing
├── robots.txt
├── sitemap.xml
└── README.md
```

## Deploying

### 1. Push to GitHub

```bash
git init
git add .
git commit -m "Initial site"
git branch -M main
git remote add origin git@github.com:YOUR_ORG/oregonheals.git
git push -u origin main
```

### 2. Enable GitHub Pages

In repo settings → **Pages**:

- Source: **Deploy from a branch**
- Branch: `main` / `root`
- Custom domain: `oregonheals.org`
- Check **Enforce HTTPS** (will be available after DNS propagates)

### 3. Point the domain

At your DNS host for `oregonheals.org`, create either:

**A records** pointing apex (`@`) to GitHub Pages:
```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

And a `CNAME` for `www`:
```
www → YOUR_ORG.github.io
```

Propagation usually takes 10–30 minutes. HTTPS may take an additional hour once the cert provisions.

## What still needs to be wired up

### Form endpoints

Both pages have a form (summit waitlist + newsletter signup) that POSTs JSON to
a placeholder URL. Search the HTML for `REPLACE_ME` and set the actual endpoint.

Two paths:

**Option A — EveryAction iframe.** Replace the entire `<form>` block with the
iframe embed code from EveryAction. Make sure the Activist Code or Source Code
field is set to `OBHS26-Waitlist` (summit) or `OregonHeals-Newsletter` (root).

**Option B — Google Apps Script (same pattern as your other sites).** Deploy
a Web App that accepts `{ email, source_code }` POSTs and writes to a Google
Sheet. Paste the deployed Web App URL into the `data-endpoint` attribute on
each form. The current JS uses `mode: 'no-cors'` so the script doesn't need
CORS headers.

A minimal Apps Script handler:

```javascript
function doPost(e) {
  var data = JSON.parse(e.postData.contents);
  SpreadsheetApp.openById('YOUR_SHEET_ID')
    .getSheetByName('Signups')
    .appendRow([new Date(), data.email, data.source_code]);
  return ContentService.createTextOutput(JSON.stringify({ ok: true }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

### Production fonts

The font files in `/assets/fonts/` are `.woff`. For ~30% smaller files
in production, regenerate as `.woff2` — see `assets/fonts/README.md` (if you
keep the originals around) or regenerate from the source OTFs with:

```bash
pip install fonttools brotli
pyftsubset Brandon_reg.otf \
  --output-file=brandon-grotesque-regular.woff2 \
  --flavor=woff2 \
  --unicodes="U+0020-007E,U+00A0-00FF,U+2010-2027,U+2030-205E" \
  --layout-features='kern,liga,clig'
```

Then update each `@font-face` block in `assets/fonts/fonts.css` to prefer woff2:

```css
src: url('brandon-grotesque-regular.woff2') format('woff2'),
     url('brandon-grotesque-regular.woff')  format('woff');
```

### Contact email

The footer and 404 page link to `hello@oregonheals.org`. Either set up that
forwarding address at the registrar / mail host, or update the links to a
real existing inbox.

## Local development

The site is plain HTML/CSS — no build step. To preview locally:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

Or use any other static server (`npx serve`, VS Code Live Server, etc).

## Accessibility notes

- Skip link to main content on each page
- Visible focus states on all interactive elements
- Proper heading hierarchy (one h1 per page)
- Alt text on all logos
- `aria-current="page"` on active nav item
- `aria-live` regions for form success messages
- `prefers-reduced-motion` respected

## License

Site code: feel free to adapt.
Brand assets (logos, fonts) and copy: © Oregon Heals Coalition.
