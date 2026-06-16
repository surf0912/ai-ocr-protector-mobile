# 預言家日報 — PWA shell

Installable mobile/PWA wrapper for the **AI / OCR Protection Image Processor**.
It shows a parchment splash screen, then loads the Streamlit app in an iframe, and
provides "add to home screen", offline shell caching, and app icons.

- **Processor app (separate repo):** `surf0912/ai-ocr-protector` →
  `https://ai-ocr-protector.streamlit.app`
- This shell iframes `…streamlit.app/?embed=true`.

## Files

| File | Purpose |
|------|---------|
| `index.html` | Splash + iframe + load-failure fallback. References manifest, icons, theme-color. |
| `manifest.json` | PWA name (`預言家日報`), colours, icons (`any` + `maskable`). |
| `service-worker.js` | Caches the shell for offline (bump `CACHE_NAME` when assets change). |
| `favicon.png`, `apple-touch-icon.png` | Browser tab + iOS home-screen icon. |
| `icon-192.png`, `icon-512.png` | `purpose: any` icons. |
| `maskable-192.png`, `maskable-512.png` | `purpose: maskable` — Android masks these to circle/squircle without cropping. |

## App name on home screen

`預言家日報` — set in three places, all must agree:
- `manifest.json` → `name` and `short_name`
- `index.html` → `<meta name="apple-mobile-web-app-title">`

iOS/Android fix the name **at install time** — after changing it, remove the old
home-screen icon and re-add.

## Regenerating icons (from an Android adaptive-icon export)

Maskable icons must keep content inside the inner ~80% "safe zone". Composite the
adaptive **background + foreground** layers into a full-bleed square:

```python
from PIL import Image
bg = Image.open("adaptive_icon_background_1024.png").convert("RGBA")
fg = Image.open("adaptive_icon_foreground_1024.png").convert("RGBA")
mask = Image.alpha_composite(bg, fg).convert("RGB")
for s in (512, 192):
    mask.resize((s, s), Image.LANCZOS).save(f"maskable-{s}.png", optimize=True)
```

Then add `maskable-*` to `manifest.json` (`purpose: maskable`) and to the service
worker `ASSETS`, and bump `CACHE_NAME`. Preview safe-zone at <https://maskable.app>.

## Deploy

GitHub Pages on this repo (Settings → Pages → branch `main`).
After changing the service worker or icons, users may need to remove + re-add the
home-screen app to pick up changes.
