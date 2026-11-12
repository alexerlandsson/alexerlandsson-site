# alexerlandsson.com

The source for [alexerlandsson.com](https://alexerlandsson.com/), the personal site of Alex Erlandsson.

The site is a single page. One sentence, set in Fraunces, drifts across the screen on a slow wave. It can be pulled, flicked and tapped, and the colours change on each visit. The animation can be paused, and it holds still for visitors who prefer reduced motion.

## How it is built

Everything is in `index.html`: plain HTML, CSS and JavaScript, with no framework, no build step, no tracking and no cookies. Without JavaScript the sentence is set as ordinary text.

## Run it locally

```sh
python3 -m http.server 8000
```

Then open <http://localhost:8000/>. Asset paths start at the root, so opening `index.html` straight from disk loses the icons and the manifest.

## Deploy

Pushing to `main` publishes the site through GitHub Pages. Every file in the repo root is served on the site, which is why this README lives in `.github/`.
