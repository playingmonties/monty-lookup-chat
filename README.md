
# Monty Chat - Developer Lookup PWA

A Progressive Web App for searching Dubai real estate developers with WhatsApp/Slack-style interface.

## Features
- 🔍 Real-time type-ahead search
- 📱 Installable as a mobile app
- ⚡ Offline support via Service Worker
- 🎨 Beautiful chat-like interface
- 🔗 n8n webhook integration

## Setup

### 1. Create App Icons
You need to create two icon files in the project root:
- `icon-192.png` (192x192px)
- `icon-512.png` (512x512px)

You can use tools like:
- [Favicon.io](https://favicon.io/)
- [RealFaviconGenerator](https://realfavicongenerator.net/)
- Canva, Figma, or Photoshop

### 2. Deploy
Host these files on a web server with HTTPS:
- `index.html`
- `manifest.json`
- `sw.js`
- `icon-192.png`
- `icon-512.png`

**Free hosting options:**
- **Netlify**: Drag & drop files at netlify.com/drop
- **Vercel**: Connect GitHub repo at vercel.com
- **GitHub Pages**: Push to repo and enable in Settings
- **Cloudflare Pages**: Upload via dashboard

### 3. Install on Phone

#### iPhone (iOS):
1. Open the hosted URL in Safari
2. Tap the Share button (⬆️)
3. Scroll and tap "Add to Home Screen"
4. Tap "Add"

#### Android:
1. Open the hosted URL in Chrome
2. Tap the menu (⋮)
3. Tap "Install app" or "Add to Home Screen"
4. Tap "Install"

## Local Development
```bash
# Serve locally (requires Python 3)
python3 -m http.server 8000

# Or with Node.js
npx serve .
```

Then open: `http://localhost:8000`

## Database
- **Platform**: Supabase
- **Project**: monty chat (jfimvpvwpyrttiitvdbr)
- **Table**: `developers` (id, developer, created_at)

## Webhook
Sends selected developer to n8n:
```
POST https://thomasmccone.app.n8n.cloud/webhook-test/montychat
{
  "developer": "Emaar Properties - Company Profile",
  "timestamp": "2025-10-27T14:56:26.164Z"
}
```