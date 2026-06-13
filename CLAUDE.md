# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal portfolio site (Pavan Tej) deployed via GitHub Pages from the `main` branch. Pure static HTML/CSS/JS — no build step, no package manager, no tests, no linter. Changes go live by committing and pushing to `main`.

## Development

Serve locally over HTTP (Barba.js page transitions fetch pages via XHR, so `file://` won't work):

```bash
python3 -m http.server 8000
```

## Architecture

Five pages, each a full standalone HTML document using directory-style clean URLs:

- `/index.html` (home), `/work/`, `/about/`, `/contact/`, `/accomplishments/` — each is `<dir>/index.html`

Despite being separate documents, the site behaves like a SPA: **Barba.js** intercepts navigation, fetches the target page, and animates between them. Each page's `<main>` carries `data-barba="container"` and a `data-barba-namespace` (`home`, `work`, `about`, `contact`, `archive`) that keys the transitions.

All behavior lives in one shared script, `assets/js/index-new.js` (~1500 lines), loaded by every page:

- `initPageTransitions()` / `barba.init(...)` — page transitions; the `home` namespace gets a special multilingual loader (`initLoaderHome`), others get `initLoader`
- `initNextWord(data)` — during a transition, the loading screen shows the destination page's name, pulled from the `.loading-words` `<h2>` list that is duplicated in every page's markup
- `initScript()` — re-run after every Barba transition; any new interactive element must be wired up here or it will break after client-side navigation
- Locomotive Scroll (vendored at `assets/js/locomotive-scroll.min.js`) provides smooth scrolling; GSAP + ScrollTrigger drive all animations; vanilla-lazyload loads images from `data-bg` attributes

External libraries (jQuery, GSAP 3.9.1, ScrollTrigger, Barba, vanilla-lazyload, js-cookie, Spline viewer) are loaded from CDNs in each page's footer — there are no local copies except Locomotive Scroll.

## Things to watch

- **Markup is duplicated per page.** Header, nav, footer, the `.loading-words` list, and the CDN script block exist in every `index.html`. A change to shared chrome must be applied to all five pages.
- **Asset paths are relative.** The root page uses `assets/...`; subpages use `../assets/...`. Copying markup between pages requires fixing these prefixes.
- CSS is split across `assets/css/`: `normalize.css`, `locomotive-scroll.css`, `styleguide.css` (design tokens), `components.css`, and `style-new.css` (main styles), loaded in that order on every page.
