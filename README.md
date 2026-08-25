# timko-filip.github.io

Personal portfolio of Filip Timko — Salesforce Marketing Cloud Consultant / Developer.

**Live:** https://timko-filip.github.io

## Structure

```
index.html              markup, bilingual EN/SK via data-en / data-sk attributes
favicon.svg
assets/css/style.css    all styling; both themes defined as CSS custom properties
assets/js/main.js       language switch, theme switch, section highlighting,
                        and the live journey animation on <canvas>
assets/img/filip.jpg
```

No build step, no framework, no dependencies. The only external request is Google Fonts
(IBM Plex Sans and IBM Plex Mono). Edit a file, commit, and GitHub Pages rebuilds it.

## Notes

- **Themes** — dark is the default. `data-ft-theme="light"` on `<html>` switches to light;
  the choice is stored in `localStorage` and applied by a small inline script in `<head>`
  before first paint, so the page never flashes the wrong theme. Every colour comes from a
  custom property defined in both blocks — nothing is hard-coded in component rules.
- **Languages** — English always loads first. Every translatable node carries `data-en`
  and `data-sk`; the switch swaps `textContent`.
- **Motion** — the journey animation pauses when it scrolls out of view and when the tab is
  hidden, and never starts at all for visitors who prefer reduced motion. The page is fully
  readable with JavaScript disabled.
