# Pranjal Jain — Cybersecurity Portfolio

A single-file, dependency-free cybersecurity portfolio built from scratch with vanilla **HTML + CSS + JavaScript**. Dark "cyber" theme in red / yellow / white, with an animated neural-network background, a binary-code portrait that assembles itself, and scroll-driven progressive loading.

**Repo:** https://github.com/Pranjal01819/Portfolio
**Live:** deploy on Vercel (see [Deploy](#deploy))

---

## Tech stack

| Layer | Choice |
|---|---|
| Markup | HTML5, one file (`index.html`) |
| Styling | Hand-written CSS (custom properties, no framework) |
| Logic | Vanilla JavaScript (ES2015+, no libraries, no build step) |
| Fonts | Google Fonts — **Chakra Petch**, **Sora**, **JetBrains Mono** |
| Graphics | `<canvas>` 2D (network + bats + portrait), inline SVG (click cracks) |
| Contact form | [Web3Forms](https://web3forms.com) — static-site mail relay, no backend |
| Hosting | Vercel (static) or GitHub Pages |

There is **no package.json, no bundler, no dependencies**. The whole site is one HTML file you can open by double-clicking.

---

## How it was made

### 1. Design system

Everything is driven by CSS custom properties at the top of the file:

```css
:root {
  --bg: #080506;              /* near-black page background        */
  --text: #f2e9e9;            /* body text                          */
  --muted: #a3949a;           /* secondary text                     */
  --red: #ff4757;             /* primary accent                     */
  --yellow: #ffd166;          /* secondary accent                   */
  --white: #f7f3ee;           /* highlights                         */
  --grad: linear-gradient(90deg, #ff4757, #ffd166);
  --surface: rgba(255,255,255,0.035);
  --surface-border: rgba(255,71,87,0.14);
}
```

Layout is a fixed 1100px max-width column with sections: **hero → about → experience → project → skills → certifications → contact → footer**. No purple/pink anywhere; the palette is strictly red (`#ff4757`), yellow (`#ffd166`) and off-white.

### 2. Background layer stack

Layers are `position:fixed` and stacked with negative z-indexes so content always sits on top:

| Layer | z-index | What it is |
|---|---|---|
| `.bg-grid` | -3 | CSS grid lines, radially masked, 44px cells |
| `#bgnet` | -2 | Canvas: drifting network graph with glowing packets |
| `#bats` | -1 | Canvas: binary "bat" swarm made of `0`/`1` characters |
| `.glow` ×3 | -4 | Blurred colour blobs, parallax-scrolled |
| `.scanline` | -1 | CRT sweep animation |

The network uses a simple O(n²) link pass (`LINK_DIST = 170px`), cursor repulsion, and packets that hop between connected nodes. Nodes/edges are intentionally faint (`rgba(34,211,238,.22)` edges, `.75` node alpha) and the canvas is blurred 2px so it reads as ambience, not noise.

The bats are Canvas text glyphs (`0`/`1`) arranged into wings, with soft repulsion between bats, a no-fly zone around the portrait, and cursor-chasing behaviour.

### 3. Binary portrait (the centrepiece)

`#faceField` is a full-viewport canvas; `#faceBox` is an invisible anchor that defines where the face sits (right side of the hero, hidden below 980px).

Pipeline used to turn a photo into binary:

1. Load the portrait from an embedded base64/data URI (`FACE_URI`).
2. Downscale to a **120-column grid**, measuring each cell's average luminance at 6× supersampling.
3. Normalise luminance to `LO 60 / HI 238`, apply unsharp masking (0.95) and a Sobel edge pass (threshold 115).
4. Convert each cell to a digit with density `0.93 + 0.09·density`, gamma `0.85`, opacity `min(1, 0.13 + 0.98·density)`; edge or dense cells render bold.
5. Every cell becomes a particle that flies in along a curved Bézier path from off-screen, lands with a flash + screen shake, then drifts in zero-g (`sin`/`cos` drift, never cursor-driven).
6. Extras: random bit flips, a slow scanline band, and cached layout so rendering does zero forced reflows while scrolling.

### 4. Scroll & interaction systems

- **Progressive loading** — sections start slightly translated/faded and get `.loaded` from an `IntersectionObserver`; a second observer adds `.in` to `.reveal` elements with stagger. All hiding CSS is scoped to `html.js`, so with JS disabled nothing is ever invisible.
- **Tag assemble/scatter** — `.tag`, `.chip`, `.metric` and contact buttons physically scatter when scrolling up and re-assemble when scrolling down, using per-element random CSS variables.
- **Click cracks + shake** — clicking empty page background spawns SVG crack lines from the click point, floating `−1 packet lost` toasts, and a damped screen-shake on `#pageShake`. Interactive elements are immune (`e.target.matches('body, html, section, #pageShake')`).
- **Filters** — Skills and Certifications have data-driven filter bars (`data-filter`, `data-cat`), live counts, and fade in/out animations.
- **Performance** — rAF loops capped at ~60fps, pause on `document.hidden`, cached element positions, throttled resize, passive listeners, reduced particle counts.

### 5. Contact form

Web3Forms handles delivery without a server:

```html
<form id="secureMail">
  <input type="hidden" name="access_key" value="YOUR-WEB3FORMS-KEY">
  ...
</form>
```

JS posts JSON to `https://api.web3forms.com/submit` and shows inline status messages. Replace the hidden `access_key` value with your own key from web3forms.com to receive messages.

### 6. Accessibility & resilience

- `prefers-reduced-motion` disables scanlines/animations, hides background canvases, and forces all content visible.
- Content visibility never depends on an animation succeeding — observers are progressive enhancement only.
- Semantic landmarks (`nav`, `section`, `footer`), keyboard-focus styles on filter buttons.

---

## Run locally

No install, no build:

```bash
git clone https://github.com/Pranjal01819/Portfolio.git
cd Portfolio
# open index.html in your browser
```

Or simply double-click `index.html`.

---

## Deploy

### Vercel (recommended, zero config)

1. Push this repo to GitHub.
2. Go to https://vercel.com/new and import the repository.
3. Framework preset: **Other**. Build command: **empty**. Output directory: **default (root)**.
4. Click **Deploy** — Vercel serves `index.html` as a static site instantly.
5. Every push to `main` redeploys automatically.

### GitHub Pages (alternative)

1. Repo → **Settings → Pages**.
2. Source: **Deploy from a branch** → branch `main`, folder `/ (root)`.
3. Site appears at `https://<username>.github.io/Portfolio/` within a minute or two.

---

## Customise

| What | Where |
|---|---|
| Colours / fonts | CSS `:root` variables and the `@import` at the top |
| Portrait photo | Replace the `FACE_URI` data URI inside `binaryPortrait()` |
| Text content | Directly in the HTML sections (`#about`, `#experience`, …) |
| Contact form inbox | `access_key` hidden input in `#secureMail` |
| Social links | LinkedIn / GitHub buttons in `#contact` |
| Filters & counts | `data-cat` on `.skill-cat` / `.cert`, buttons in `.filter-bar` (counts are computed from the DOM) |

---

## Project structure

```
Portfolio/
├── index.html   ← the entire site (markup + CSS + JS + portrait data)
└── README.md
```

---

## Author

**Pranjal Jain** — B.Tech CSE (Cybersecurity), GLA University
TryHackMe Top 5% · CTF player · builder of security tooling

- GitHub: [@Pranjal01819](https://github.com/Pranjal01819)
- LinkedIn: [pranjal-jain-pj1819](https://www.linkedin.com/in/pranjal-jain-pj1819)
- Email: pranjaljain1819@gmail.com

---

*Designed & secured — © 2026 Pranjal Jain*
