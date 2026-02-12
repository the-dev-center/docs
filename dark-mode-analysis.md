# Antora Dark Mode Analysis

## Summary

Dark mode now works by layering a supplemental UI over the Antora default UI. The toggle button is injected at runtime and applies a `dark-theme` class on the `<html>` element. Dark styling is implemented with explicit CSS selectors so it doesnâ€™t depend on CSS variables being present in the default UI bundle.

## Root Cause

The original toggle logic attached its click handler only on the `DOMContentLoaded` event. Because the script is loaded at the end of the page, the event had already fired in some cases, so the handler never attached and the theme class never changed.

## Fix Applied

- Updated the toggle script to run immediately when the DOM is already loaded, and to always bind the click handler if a `#theme-toggle` button already exists.
- Switched dark mode styles to explicit selectors under `html.dark-theme` instead of only overriding CSS variables. The default UIâ€™s compiled CSS does not rely on variables, so variable overrides alone had no effect.

## How It Works

- `supplemental-ui/partials/head-meta.hbs`
  - Loads `site-extra.css` and pre-applies the `dark-theme` class if a saved preference exists or the OS is in dark mode.
- `supplemental-ui/partials/footer-scripts.hbs`
  - Loads `site-dark-mode.js` after the default UI scripts.
- `supplemental-ui/js/site-dark-mode.js`
  - Injects or binds the toggle button and persists the choice in `localStorage` under `antora-theme`.
- `supplemental-ui/css/site-extra.css`
  - Applies dark colors using explicit selectors under `html.dark-theme`.

## Scope of the Fix

This fix **does not change the Antora default UI bundle** itself. It only affects this site (and any other site built with this repo and the `supplemental-ui` configuration in the playbook). If you want dark mode across other Antora sites, you would need to apply the same supplemental UI approach to those playbooks or fork/customize the UI bundle for those projects.

## Files Involved

- `antora-playbook.online.yml`
- `antora-playbook.testing.yml`
- `supplemental-ui/partials/head-meta.hbs`
- `supplemental-ui/partials/footer-scripts.hbs`
- `supplemental-ui/js/site-dark-mode.js`
- `supplemental-ui/css/site-extra.css`

## Validation

Run locally:

```bash
pnpm build:testing
```

Then open `build/site/home/index.html`, click the toggle, and confirm `dark-theme` is present on the `<html>` element and styles change.
