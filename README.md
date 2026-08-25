# timko-filip.github.io

Personal portfolio of Filip Timko — Salesforce Consultant at IBM, Marketing Cloud.

**Live:** https://timko-filip.github.io

## Structure

```
index.html                    markup; bilingual EN/SK via data-en / data-sk attributes
favicon.svg
robots.txt                    search engines welcome, AI training crawlers declined
sitemap.xml
assets/css/style.css          all styling; both themes as CSS custom properties
assets/js/main.js             language switch, theme switch, section highlighting,
                              and the live journey animation on <canvas>
assets/img/filip.jpg
assets/img/og.png             1200×630 social preview card
assets/img/badges/            Salesforce certification badges
```

No build step, no framework, no dependencies. The only external request is Google Fonts
(IBM Plex Sans and IBM Plex Mono). Edit a file, commit, and GitHub Pages rebuilds it.

## Notes

- **Themes** — dark is the default. `data-ft-theme="light"` on `<html>` switches to light;
  the choice is stored in `localStorage` and applied by a small inline script in `<head>`
  before first paint, so the page never flashes the wrong theme. Every colour comes from a
  custom property defined in both blocks — nothing is hard-coded in component rules, and the
  light palette is tuned separately rather than inverted, so both clear WCAG AA.
- **Languages** — English always loads first, whoever the visitor is. Every translatable node
  carries `data-en` and `data-sk`; the switch swaps `textContent`, so translatable elements
  are always leaf nodes.
- **Motion** — the journey animation pauses when it scrolls out of view and when the tab is
  hidden, repaints itself when the theme changes, and never starts at all for visitors who
  prefer reduced motion. The page is fully readable with JavaScript disabled.
- **Icons and flags** are inline SVG rather than an icon font or emoji — emoji flags do not
  render on Windows. Icons that replace a text label keep that label as visually hidden text,
  so screen readers still announce it.
- **Certifications** link to independent verification rather than showing document scans.
