# Q2FA — 2FA QR Code Generator

A privacy-first, zero-server 2FA QR code generator. Paste any TOTP URI, get a perfect QR code instantly. Everything runs 100% in your browser — nothing is ever sent to any server.

---

## Features

- **Instant QR generation** from `otpauth://totp/...` URIs
- **Fully client-side** — no data leaves your browser
- **Smart URI parsing** — extracts label and issuer automatically
- **Copy URI** and **Download PNG** actions
- **Error handling** — validates URI format, missing secret, and bad encoding
- **Sample URI** — one click to see it in action
- **Deploys in seconds** — single static HTML file on Cloudflare Pages (free)

---

## Usage

1. Open `index.html` in any browser, or
2. Visit the live deployment at `https://q2fa.pages.dev`

### Generating a QR Code

1. Paste a TOTP URI from your 2FA setup (e.g. from GitHub, Google, etc.)
2. Click **Generate QR Code** (or press `Enter`)
3. Scan the QR with Google Authenticator, Authy, or any TOTP app
4. Use **Copy URI** to save the full URI, or **Download PNG** to save the QR image

### Example TOTP URI

```
otpauth://totp/GitHub:yxxiao@example.com?secret=JBSWY3DPEHPK3PXP&issuer=GitHub&algorithm=SHA1&digits=6&period=30
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
