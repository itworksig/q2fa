# Q2FA — 2FA QR Code Generator

A privacy-first, zero-server 2FA QR code generator. Paste any TOTP URI **or enter a Base32 Secret**, get a perfect QR code instantly. Everything runs 100% in your browser — nothing is ever sent to any server.

---

## Features

- **Instant QR generation** from `otpauth://totp/...` URIs or raw Base32 secrets
- **Fully client-side** — no data leaves your browser
- **Smart URI parsing & building** — extracts metadata from URI, or builds a valid URI from Secret + optional Issuer/Account
- **Copy URI** and **Download PNG** actions
- **Error handling** — validates URI format, missing secret, and bad encoding
- **URI + Secret modes** — works with full URIs or manual secret input
- **Deploys in seconds** — single static HTML file on Cloudflare Pages (free)

---

## Usage

1. Open `index.html` in any browser, or
2. Visit the live deployment at `https://q2fa.pages.dev`

### Generating a QR Code

1. Paste a TOTP URI from your 2FA setup (e.g. from GitHub, Google, etc.), **or** input your Base32 Secret directly
2. Click **Generate QR Code** (or press `Enter`)
3. Scan the QR with Google Authenticator, Authy, or any TOTP app
4. Use **Copy URI** to save the full URI, or **Download PNG** to save the QR image

### Quick Tutorial

#### Method A: Paste URI

1. Paste your full `otpauth://totp/...` URI in the URI box.
2. Click **Generate QR Code**.
3. Scan with your authenticator app (Google Authenticator, Authy, 1Password, etc.).
4. Use the generated TOTP code to verify login.

#### Method B: Input Secret manually

1. Keep URI box empty.
2. Enter your Base32 value into **Secret (Base32)**.
3. Optionally fill **Issuer** and **Account** (example: `email@example.com`).
4. Click **Generate QR Code**, then scan in your authenticator app.

#### Notes

- Supported secret format: Base32 (`A-Z`, `2-7`; spaces and `=` are ignored).
- If URI and Secret are both filled, URI takes priority.
- All data stays in your browser.

### Example TOTP URI

```
otpauth://totp/GitHub:email@example.com?secret=JBSWY3DPEHPK3PXP&issuer=GitHub&algorithm=SHA1&digits=6&period=30
```

---

## Deploy to Cloudflare Pages (Free)

### Option A: Drag & Drop (Easiest)

1. Go to [dash.cloudflare.com](https://dash.cloudflare.com)
2. Select **Workers & Pages** → **Create application** → **Pages** → **Upload assets**
3. Drag the `dist/` folder (or just `index.html`) into the drop zone
4. Done — you'll get a `*.pages.dev` URL instantly

### Option B: Wrangler CLI

```bash
# Install Wrangler
npm install -g wrangler

# Login to Cloudflare
npx wrangler login

# Deploy
npx wrangler pages deploy dist
```

### Option C: GitHub Actions (CI/CD)

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Cloudflare Pages

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        uses: cloudflare/pages-action@v1
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          directory: dist
```

---

## Project Structure

```
q2fa/
├── index.html         ← source file (root)
├── dist/
│   └── index.html    ← copy for deployment
├── wrangler.toml      ← Cloudflare Pages config
├── SPEC.md            ← design specification
└── README.md
```

---

## Technical Details

- **Zero dependencies** in the deployment artifact
- `qrcode@1.5.3` loaded from jsDelivr CDN (with SRI integrity hash)
- Google Fonts: **Syne** (display) + **JetBrains Mono** (body)
- Pure vanilla JS — no framework, no build step
- ~15 KB total (HTML + inline CSS/JS), loads in milliseconds
