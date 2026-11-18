# CRISPR-Net — MLSN F26 Group 2

The talk deck for our MLSN presentation, as a web page.

> **Reproducing the Nature paper** *"Optimized CRISPR guide RNA design for two
> high-fidelity Cas9 variants by deep learning"* — Wang et al.,
> *Nature Communications* **10**, 4284 (2019).

Samuel (tech lead) · Lloyd · Caleb · AJ · Saishyam · Vincent

---

## What is in here

| File | What it is |
|---|---|
| `index.html` | The whole deck: 20 talk slides, a thank-you slide, and 4 backup slides |
| `SPEAKER_SCRIPT.md` | Per-slide script, timings, and likely Q&A |
| `render.yaml` | Optional Render blueprint (see below — you can ignore it) |

`index.html` is **completely self-contained**. Every chart is drawn in the browser
from the measured numbers, and the two screenshots are embedded in the file. The
only thing it loads from the network is Google Fonts, and it still reads fine
without them. There is no build step and no `node_modules`.

## Using the deck

Open `index.html` in any browser, or visit the deployed URL.

| Key | Does |
|---|---|
| `→` / `Space` / click | Next slide |
| `←` | Previous slide |
| `N` | Presenter notes panel, with an elapsed clock |
| `R` | Reset the clock |
| `F` | Fullscreen — **use this when presenting** |
| `Home` / `End` | First / last slide |

The slide canvas is a fixed 1920×1080 stage scaled to whatever window it is in, so
it looks identical on a laptop, a projector, or a phone. It will letterbox rather
than reflow.

Budget is **9:40** across the 20 talk slides. Slide 21 is the thank-you slide, and 22 to 25
are backup: outside the clock, there only to answer questions.

---

## Deploying to Render

Render serves this as a **Static Site**: no server, no build, free tier.

### 1. Put this folder on GitHub

From inside this folder:

```bash
git init -b main
```

```bash
git add -A && git commit -m "CRISPR-Net talk deck"
```

Create an empty repository on GitHub (no README, no .gitignore — this folder
already has what it needs), then:

```bash
git remote add origin https://github.com/<your-username>/<your-repo>.git
```

```bash
git push -u origin main
```

### 2. Create the Render service

1. Sign in at <https://dashboard.render.com> with your GitHub account.
2. **New → Static Site**.
3. Pick this repository. Authorise Render to read it if it asks.
4. Fill in:
   - **Name** — becomes the URL, e.g. `crispr-net-slides` →
     `https://crispr-net-slides.onrender.com`
   - **Branch** — `main`
   - **Build Command** — **leave empty**
   - **Publish Directory** — `.`
5. **Create Static Site**.

The first deploy takes under a minute. Render serves `index.html` at the root, so
the bare URL opens the deck.

### 3. Updating it

Push to `main` and Render redeploys automatically:

```bash
git add -A && git commit -m "Update slides" && git push
```

---

### Notes and gotchas

- **Publish Directory must be `.`**, not `public` or `dist`. The deck sits at the
  repository root. Getting this wrong is the usual cause of a "Not Found" page.
- **Build Command must be empty.** There is nothing to build, and a stray command
  will fail the deploy.
- **The free tier does not sleep for static sites.** Unlike Render's free web
  services, static sites are served from a CDN and are always up — so the URL will
  not be cold when you present. It is still worth opening it once beforehand.
- **Have a local copy open as a fallback.** Any live URL depends on the room's
  wifi; `index.html` works offline, only losing the web fonts.
- Using `render.yaml` instead: choose **New → Blueprint** rather than **New →
  Static Site**, and Render reads the settings from the file. The dashboard route
  above is simpler, and the two are interchangeable.
