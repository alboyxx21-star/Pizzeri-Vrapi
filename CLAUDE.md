# Piceri Vrapi — project notes

Single-page marketing site for **Piceri Vrapi**, a pizza / fast-food / hallall
restaurant in Kombinat, Tiranë (since 2014). Albanian-language copy.

## What this is
- The whole site is ONE file: **`Piceri Vrapi.dc.html`**. Edit that.
- It's a "DC" document: `<x-dc>` template + a `<script type="text/x-dc">`
  `class Component extends DCLogic`. The runtime that renders it is
  **`support.js`** (generated — do not edit). `image-slot.js` powers the
  `<image-slot>` map placeholder.
- Third-party libs via CDN: **GSAP + ScrollTrigger** (scroll animations),
  **Lenis** (smooth scroll). Fonts: Instrument Serif, Outfit, Poppins.
- `uploads/` holds all images + `hero-video.mp4`. `menu1..menu7.jpg` are the
  menu pages (menu8 was a dup of menu7). `gal-*.webp` are the gallery.

## How to run it
Static file, but it must be served over http (not file://) for the fonts/video
/menu images to load. There's no build step and no dev server in package.json,
so serve the folder yourself:

```bash
# from the project root — a tiny static server on :3000, then open it
node <<'EOF'
const http=require('http'),fs=require('fs'),path=require('path');
const ROOT=process.cwd(),MIME={'.html':'text/html','.js':'text/javascript','.css':'text/css','.json':'application/json','.png':'image/png','.jpg':'image/jpeg','.jpeg':'image/jpeg','.webp':'image/webp','.mp4':'video/mp4','.svg':'image/svg+xml'};
http.createServer((q,s)=>{let u=decodeURIComponent(q.url.split('?')[0]);if(u==='/')u='/Piceri Vrapi.dc.html';const f=path.join(ROOT,u);fs.readFile(f,(e,d)=>{if(e){s.writeHead(404);return s.end()}s.writeHead(200,{'Content-Type':MIME[path.extname(f).toLowerCase()]||'application/octet-stream'});s.end(d)})}).listen(3000,()=>console.log('http://localhost:3000'));
EOF
```
Then open http://localhost:3000/ .

## How to verify visually (this machine has no Chrome)
`puppeteer-core` is a dependency (`npm install` if `node_modules` missing).
Drive **Microsoft Edge** headless — Chrome is NOT installed:
`executablePath: 'C:\\Program Files (x86)\\Microsoft\\Edge\\Application\\msedge.exe'`.
When running a puppeteer script from the scratchpad, require it by absolute
path: `require('<project>/node_modules/puppeteer-core')`.
Always screenshot after a change and LOOK at it; also assert
`document.documentElement.scrollWidth === window.innerWidth` (no horizontal
overflow) on mobile viewports.

## Page sections (in order), all inside `#vrapi-root`
`#vrapi-nav` (fixed header) · `#hero` (bg video) · `#about` · `#showcase`
(pinned spinning-pizza swap) · `#menu-open` (3D flip-book menu) · `#gallery`
(pinned cover-flow) · `#reviews` · `#location` · `#vrapi-footer`.

## Design decisions already made (don't undo without reason)
- **Global colour override lives at the top of `<style>`**: `h1..h6,p,a,span
  {color:#1C1712!important}` + light section backgrounds. This forces a
  light theme on the content bands. Because it's blanket `!important`, any
  section that should stay dark (nav, hero, `#showcase`, footer) needs its
  own scoped `!important` rule — that's why those exist.
- **Section bands alternate shades** so each reads as its own block:
  white (`#FFF`) ↔ warm paper (`#F7F0E3`), with `#showcase` + footer as dark
  anchors between them. Light→light borders get a 1px hairline.
- **3D flip-book menu** (`#vrapi-flipbook`): real page-turn, replaced an older
  scroll-scrub "open the cover" sequence. 5 sheets = branded cover + 7 menu
  pages + back cover. 64 clickable dish hotspots (`.vrapi-hs`) are injected in
  JS from the `PAGES` array onto each `[data-page]` face; clicking opens the
  detail popup. Navigate via ‹ › buttons, swipe, or click the cover.
  - **Phones (≤640px) use a DIFFERENT menu**: a flat single-page pager
    (`#vrapi-mobile-page` img + `#vrapi-mobile-hs` hotspots), NOT the 3D book.
    Reason: on mobile GPUs, `preserve-3d` + `backface-visibility:hidden` layers
    rasterize at 1× and look blurry on a 2–3× screen. Flat 2D = full-res + more
    readable. Chosen at init via `matchMedia('(max-width:640px)')` → `this._mobile`;
    `go()` branches to `goMobile()`, and CSS hides `[data-sheet]` / resizes the
    book to one portrait page. Desktop 3D book is unchanged.
- **Gallery cover-flow is autoplay + drag + buttons, NOT scroll-pinned.** The
  old pinned `ScrollTrigger(end:'+=1800')` fought Lenis and trapped vertical
  scrolling ("can't scroll down"). Now a continuous `cf.auto` tween drives
  `LOOP_HEAD.totalTime` via `cf.render()`; drag/buttons add `cf.manual` offset
  and pause autoplay while held. The page never traps scroll on any device.
- **Loading screen** (`#vrapi-loader`, first child of `#vrapi-root`): white
  overlay, logo pops in + two SVG rings draw + name/tagline rise (CSS keyframes),
  fades out at ~2.5s. `componentDidMount` force-removes it at 3.4s (safety net for
  reduced-motion, where the CSS fade is disabled) and sets `window.__vrapiLoaderDone`
  so it never replays on a remount. Base states are the FINAL values with
  `animation-fill-mode:both`, so reduced-motion shows a static assembled logo.
- **Reviews slider** (`#vrapi-rev-track`): ported from a CodePen image slider —
  one review per slide, flex track translated by `-i*100%`, ‹ › arrows (DC
  `revPrev/revNext` handlers), dots + pointer-swipe (wired in `initAnim`, guarded by
  `dataset.built`), autoplay every 5.5s (`revAuto`), GSAP entrance per slide
  (`revGo`). Track height is fixed to the tallest review (no jump between slides).
  A red "Lini një vlerësim në Google" CTA links to `maps?cid=13659342507174827606`
  (the real TE VRAPI listing). Reviews are real Google reviews, translated to Albanian.
- **Location** uses a live Google Maps `<iframe>` embed (not the old image-slot).
- **Premium footer** (`#vrapi-footer`): 3 columns (brand+socials / Explore /
  Contact) + bottom bar. Instagram/Facebook links are placeholders
  (instagram.com / facebook.com) — swap for the real profiles when known.
  WhatsApp = wa.me/355699191821.

## Responsive (works iPhone SE → Samsung → iPad → desktop)
- Breakpoints in `<style>`: `≤900px` (tablet: single-col grids, burger nav,
  reviews 2-up), `≤640px` (phones: stacked showcase, reviews 1-up, trimmed
  padding, full-width CTAs), `≤380px` (small phones: smaller hero title/logo).
- **Mobile menu = CSS checkbox-hack hamburger.** Critical gotcha: the overlay
  (`.vr-mobile-menu`) and its `#vrapi-nav-cb` checkbox live **outside**
  `#vrapi-nav`, as direct children of `#vrapi-root`. Reason: GSAP leaves a
  `transform` on `#vrapi-nav-inner` and the header has `backdrop-filter`; both
  create a containing block that would trap a `position:fixed` overlay inside
  the tiny nav bar. The burger `<label>` stays in the nav (references the
  checkbox by `for=`). Burger→X uses `#vrapi-root:has(.vr-nav-cb:checked)`
  (sibling `~` can't reach it across the DOM split). JS in
  `componentDidMount` closes the menu on link tap.

## Gotchas
- The DC runtime **re-serialises inline styles with spaces**:
  `rgba(242, 234, 217, …)`. Attribute selectors like `[style*="242,234,217"]`
  must ALSO include the spaced form or they silently miss.
- `componentDidMount` / `initAnim` can re-run (there's a watchdog that rebuilds
  ScrollTrigger if the host detaches the DOM). Guard one-time DOM setup with a
  `dataset` flag (see `fbk.dataset.built`, `mm.dataset.wired`).
- The user's editor sometimes rewrites values on save (e.g. `black` →
  `rgb(5,5,5)`); re-Read before an Edit if a match fails.
- `prefers-reduced-motion` disables GSAP and falls back to static layouts
  (flip-book opens flat, gallery becomes a wrapped row).
