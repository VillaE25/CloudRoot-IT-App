# CloudRoot Support PWA

Customer-facing support portal. Installable to home screen on iOS, Android, and desktop. Works offline (queues submissions until back online).

---

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole app — form, logic, styles |
| `manifest.json` | PWA install metadata |
| `sw.js` | Service worker — offline caching |
| `icon.svg` | Scalable app icon |
| `icon-192.png` / `icon-512.png` | Required PNG icons for iOS / Android |

Single self-contained app. No build step, no npm, no dependencies.

---

## How it works right now (v1 — email mode)

1. Customer fills out the form
2. On submit, the app opens their default email client pre-filled with a structured ticket body addressed to `Support@cloudroot-it.com`
3. Customer hits send → it lands in your existing support inbox
4. A unique ticket reference (`CR-YYYYMMDD-XXXX`) is generated and shown to the customer
5. If offline: the ticket is saved to localStorage and the mailto is triggered when they come back online

**Why mailto for v1:** zero backend, zero hosting cost beyond a static page, zero new integrations. You can deploy this today and route everything through your existing email workflow.

---

## Deployment options (cheapest → most polished)

### Option A: GitHub Pages (free)
1. Create a public repo
2. Drop all files in the root
3. Settings → Pages → Deploy from branch → `main` / root
4. URL: `https://<user>.github.io/<repo>/`
5. Buy a domain (e.g. `support.cloudroot-it.com`) and point a CNAME at the Pages URL

### Option B: Azure Static Web Apps (free tier)
Better fit since you're on Microsoft 365 already.
1. Azure Portal → Create Static Web App → connect to a GitHub repo
2. Build preset: "Custom" — app location `/`, no build command
3. Free SSL, free custom domain, integrates with Entra ID if you ever want auth

### Option C: Any web host
Just upload the 5 files to any folder with HTTPS. Must be HTTPS for PWA to work.

---

## Swap in real branding

When you share the logo + brand colors, edit these spots in `index.html`:

**Colors** — top of the `<style>` block:
```css
--brand-primary: #0a2540;   /* main color (header, button) */
--brand-accent:  #00a878;   /* success / highlights */
--brand-warm:    #f4a261;   /* offline / warning */
```

**Logo** — replace the `.brand-mark` div (currently just "C" in a colored square) with an `<img src="logo.svg">` or similar.

**Icons** — replace `icon.svg`, `icon-192.png`, `icon-512.png` with branded versions. Keep the same filenames so the manifest still works.

**Theme color** (status bar on mobile when installed) — in `index.html`:
```html
<meta name="theme-color" content="#0a2540" />
```
And in `manifest.json` (`theme_color` + `background_color`).

---

## Upgrade path: from email to Autotask

When you're ready to push tickets directly into Autotask PSA, replace the `submitTicket()` function in `index.html` (around line ~600) with a `fetch()` POST to either:

1. **Power Automate / Logic App** — a flow that takes a webhook and creates an Autotask ticket via their REST API. Easiest because Power Automate has connectors and you avoid CORS issues.
2. **Azure Function** — a small HTTP-triggered function that calls Autotask's REST API. More flexible, slightly more setup.

Either way, the form data is already structured and ready to send as JSON. The mailto can stay as a fallback if the API call fails.

Sketch:
```js
async function submitTicket(data) {
  const res = await fetch('https://your-flow-url', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data),
  });
  if (!res.ok) throw new Error('submit failed');
  return res.json();
}
```

---

## Test locally

```bash
cd cloudroot-support
python3 -m http.server 8000
# open http://localhost:8000
```

Service worker only works on `localhost` or HTTPS — `file://` won't register it. Mailto still works either way.

---

## What you get out of the box

- ✓ Installable to home screen (iOS / Android / desktop)
- ✓ Offline-tolerant (saves locally, retries when online)
- ✓ Mobile-first responsive layout
- ✓ Form validation
- ✓ File attachments (up to 5 files, 10 MB each — referenced by name in email)
- ✓ Priority selection with visual treatment for urgent tickets
- ✓ Auto-generated ticket reference numbers
- ✓ Online/offline status indicator
- ✓ Accessible (proper labels, keyboard nav, focus states)
- ✓ Zero external JS dependencies — only fonts come from Google
