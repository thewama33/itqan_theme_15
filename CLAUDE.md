# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Itqan Theme 15 is a custom Frappe 15 theme app that replaces the default Frappe desk UI with a side-menu layout, custom navbar, configurable color themes, custom login page, and additional UI plugins. It runs as a standard Frappe app managed by `bench`.

## Commands

```bash
# Install the app on a site
bench --site <site-name> install-app itqan_theme_15

# Build assets (CSS/JS bundles)
bench build --app itqan_theme_15

# Watch for changes during development
bench watch --app itqan_theme_15

# Run the development server
bench start

# Run tests
bench --site <site-name> run-tests --app itqan_theme_15

# Run a single test
bench --site <site-name> run-tests --module itqan_theme_15.itqan_theme_15.doctype.theme_settings.test_theme_settings
```

## Architecture

### App Structure

The inner `itqan_theme_15/` directory is the Python package. Key paths:

- **`hooks.py`** — Registers CSS/JS includes, website context (favicon, splash image), and web page assets. This is the main configuration entry point for the Frappe app.
- **`api.py`** — Whitelisted API endpoints: `get_theme_settings` (guest-accessible), `change_language`, `get_company_logo`, `update_theme_settings`, `get_events`, `update_menu_modules`.
- **`www/app.py` + `www/app.html`** — Overrides Frappe's default desk page (`/app`). Loads theme settings from the DB, injects them into the Jinja context, and renders a custom HTML shell with side-menu, theme colors, font family, and CSS variables.
- **`www/login.py` + `www/login.html`** — Overrides the default login page with custom branding, background slideshow support, and social login integration.

### DocTypes

- **`Theme Settings`** (Single) — Central configuration doctype. Controls theme color (Blue/Green/Red/Orange/Yellow/Pink/Violet), font family, login page background (single photo or slideshow), side menu behavior, favicon, logos, and layout color application flags (navbar, menu, dashboard, workspace).
- **`Slideshow Photos`** (Child Table) — Stores photos for the login page slideshow, parented to Theme Settings.

### Frontend

- **`public/js/itqan_theme.js`** — Main desk-side JS. Uses jQuery and Vue 2 instances (not SFCs) for navbar components (logo, user info, language switcher). Handles side-menu toggling, fullscreen, search, and module navigation.
- **`public/js/itqan_theme.app.min.js`** — Minified app JS bundle included in desk.
- **`public/js/itqan_theme.web.min.js`** — Minified JS for web (login) pages.
- **`public/scss/`** — SCSS source files:
  - `itqan_theme.bundle.scss` — Main bundle entry point (built by Frappe's asset bundler)
  - `itqan_theme.scss` — Core styles
  - `partials/` — Modular SCSS: `_header`, `_side-menu`, `_body`, `_content`, `_forms-ui`, `_grid`, `_workspace`, `_rtl`, `_mobile`, `_responsive`, `_dark-style`, and per-color theme files (`_green-style`, `_red-style`, `_orange-style`, `_yellow-style`, `_pink-style`, `_violet-style`)
- **`public/plugins/`** — Third-party jQuery/CSS plugins (nicescroll, owl-carousel, tooltip, fontawesome, animate.css, etc.)

### Theming System

Theme colors work via CSS custom properties. In `app.html`, the selected theme color maps Frappe's `--blue-*` variables to the chosen color's variables (e.g., `--green-*`), and sets `--primary`, `--brand-color`, etc. Each color has a corresponding SCSS partial defining its palette. The theme color, font, and layout flags are read from `Theme Settings` at page load.

### Templates

- **`templates/side-menu.html`** / **`templates/sidemenu.html`** — Side navigation menu markup
- **`templates/includes/splash_screen.html`** — Loading splash screen
- **`templates/includes/login/login.js`** — Login page client scripts
