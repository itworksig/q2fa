# SPEC.md — Q2FA

## 1. Concept & Vision

A minimalist, single-page 2FA QR code generator that feels like a premium developer tool. The experience is calm, focused, and precise — like a beautifully crafted CLI tool made visual. Input a TOTP URI, get a perfect QR code instantly. No accounts, no servers, no noise. Pure client-side magic that deploys as a single HTML file on Cloudflare Workers.

## 2. Design Language

**Aesthetic Direction:** "Obsidian Terminal" — the refined intersection of a high-end dark IDE and a typographic poster. Think VS Code meets a Swiss design poster. Precision, contrast, and whitespace doing heavy lifting.

**Color Palette:**
- Background: `#0b0b0f` (near-black with a hint of blue)
- Surface: `#13131a` (card/panel background)
- Border: `#1e1e2e` (subtle dividers)
- Primary text: `#e2e2f0` (cool white)
- Secondary text: `#6b6b80` (muted labels)
- Accent: `#7c6af7` (electric violet)
- Accent glow: `rgba(124, 106, 247, 0.15)`
- Success: `#4ade80`
- Error: `#f87171`

**Typography:**
- Display/heading: `Syne` (Google Fonts) — geometric, modern, distinctive
- Body/UI: `JetBrains Mono` — monospace, developer-native, readable
- Fallback: `monospace`

**Spatial System:**
- Base unit: 8px
- Generous padding on the main card (48px)
- Input area separated from QR output by breathing room
- Max content width: 560px centered

**Motion Philosophy:**
- QR code fades in and scales up (opacity 0→1, scale 0.95→1, 400ms ease-out)
- Subtle glow pulse on the accent border when QR is generated
- Input focus ring uses smooth transition (200ms)
- Error shake animation on invalid URI (300ms)
- No gratuitous animations — everything purposeful

**Visual Assets:**
- QR code rendered via `qrcode` library (canvas)
- Lucide-style inline SVG icons (shield, copy, refresh)
- No external images — all CSS/SVG

## 3. Layout & Structure

```
┌──────────────────────────────────────┐
│  [Shield Icon]  Q2FA                 │  ← Header (minimal)
│  Two-Factor Auth QR Generator        │
├──────────────────────────────────────┤
│  TOTP URI Input                      │  ← Label
│  ┌────────────────────────────────┐  │
│  │  otpauth://totp/...            │  │  ← Monospace input
│  └────────────────────────────────┘  │
│  [Generate QR Code]                  │  ← Accent button
├──────────────────────────────────────┤
│                                      │
│         ┌──────────────┐             │
│         │              │             │
│         │   QR CODE    │             │  ← QR canvas (centered)
│         │              │             │
│         └──────────────┘             │
│         Account: GitHub              │  ← Extracted label
│         Issuer: user@example.com     │
│         [Copy] [Download PNG]        │  ← Action buttons
│                                      │
├──────────────────────────────────────┤
│  247 193           30s  [====--]     │  ← TOTP live code + countdown
│                                      │
├──────────────────────────────────────┤
│  All processing happens in-browser.  │  ← Privacy footer
│  Zero data is sent to any server.    │
└──────────────────────────────────────┘
```

- Single-column, vertically stacked, max-width centered
- Fully responsive — looks great at 320px to 1440px+
- Mobile: full-width with reduced padding
- Desktop: card floats with shadow on the dark background

## 4. Features & Interactions

### Core Feature: TOTP URI → QR Code

**Input:** A full `otpauth://totp/...` URI (Google Authenticator / Authy format)

**Process:**
1. User pastes or types URI into the monospace input
2. Clicks "Generate QR Code" (or presses Enter)
3. App parses the URI, extracts label + issuer + secret
4. QR code is rendered on a `<canvas>` element
5. QR fades in with animation
6. Live TOTP code appears below and updates every second with a 30s countdown timer

**Parse & Display:**
- Extract `label` from path (decode URI component)
- Extract `issuer` from query param or label prefix
- Display these as metadata below the QR
- If label is empty, show "Unnamed Account"

**Error Handling:**
- Invalid URI format → red border on input, shake animation, inline error message
- Missing secret → "URI must contain a `secret` parameter"
- Empty input → no action (button disabled when empty)

**Actions on QR:**
- **Copy to clipboard** — copies the full URI, shows "Copied!" feedback for 2s
- **Download PNG** — downloads the QR as `q2fa-qr.png`
- **Regenerate** — clears and refocuses input

**Edge Cases:**
- Very long issuer/label → truncate with ellipsis at 30 chars
- URI with spaces or special chars → properly handled via `decodeURIComponent`
- Double-click in input → selects all text

## 5. Component Inventory

### Input Field
- States: default (dark border), focused (violet glow border), error (red border + shake), disabled
- Monospace font, full-width
- Placeholder: `otpauth://totp/Example:user@example.com?secret=JBSWY3DPEHPK3PXP&issuer=Example`

### Generate Button
- States: default (violet background), hover (lighter violet + glow), active (pressed scale), disabled (muted, no pointer)
- Full-width on mobile, auto-width on desktop
- Icon: shield+qrcode inline SVG

### QR Canvas Card
- Appears only when a valid QR is generated
- White QR on a slightly lighter card surface (contrast for scanning)
- Padding around QR: 24px
- Subtle border, rounded corners (12px)

### Metadata Display
- Two lines below QR: "Account: ..." and "Issuer: ..."
- Muted text, monospace font
- Truncated if too long

### Action Buttons (Copy / Download)
- Icon + text, ghost style (no background, border on hover)
- Copy shows checkmark icon on success

### Error Message
- Red text below input
- Shake animation on the input container
- Auto-clears when user starts typing

### Privacy Footer
- Very muted, small text
- Shield icon + message

## 6. Technical Approach

**Runtime:** Cloudflare Workers (single HTML file deployment)

**Architecture:**
- Single `index.html` — fully self-contained
- Zero external dependencies that require a build step
- CDN-loaded libraries (qrcode.js) with integrity hash
- Google Fonts loaded via `<link>` with `display=swap`

**Libraries:**
- `qrcode-generator` v1.4.4 by Kazuhiko Arase via jsDelivr CDN — industry-standard QR encoder, MIT licensed
- Google Fonts: Inter + JetBrains Mono

**Key Implementation Notes:**
- All JS is vanilla — no framework overhead (critical for CF Workers cold starts)
- URI parsing via `URL` and `URLSearchParams` — no regex hacks
- TOTP generation via Web Crypto API (`crypto.subtle`) — HMAC-SHA1, RFC 6238 compliant
- Base32 decoding implemented from scratch (no external deps)
- Canvas API for QR rendering with configurable size (240×240)
- `canvas.toBlob()` + download link for PNG export
- `navigator.clipboard.writeText()` for copy

**Cloudflare Deployment:**
- Save as `dist/index.html`
- `wrangler dev` for local testing
- `wrangler deploy` for production
- No build step required — pure static file

**File Structure:**
```
q2fa/
├── SPEC.md
├── index.html        ← single deployable file
├── wrangler.toml     ← CF Workers config
└── README.md
```
