# ListingHive Child — review-only scaffold

This directory contains the minimal, **inactive** child theme for Lagina.lt.

## Structure

- `style.css`: WordPress child theme metadata, with the correct `Template: listinghive`.
- `functions.php`: direct-access guard only; **no CSS/JS enqueueing**.
- `assets/css/.gitkeep` and `assets/js/.gitkeep`: placeholder directories.

## Why no parent stylesheet enqueue?

ListingHive loads its frontend assets through the HiveTheme framework. Its main handle includes `hivetheme-parent-frontend`; blindly adding `listinghive-style` for the parent theme risks duplicate styling. This skeleton intentionally leaves the parent theme's asset pipeline untouched. Any future child stylesheet enqueue must be tested against the installed ListingHive 1.2.13 and the current custom CSS on **protected staging**.

## Deployment gate — DO NOT ACTIVATE YET

1. Document existing Customizer/Additional CSS, snippets, menu/widget assignments, and current plugin/theme versions.
2. Confirm staging backup and a reversible rollback (return to the original ListingHive theme).
3. Install the child theme **only on protected staging**, without activating it initially.
4. Compare parent-theme and child-theme preview for desktop/mobile appearance, layouts, fonts, navigation, filters, registration, login, listing submission, messages, and theme toggle, using disposable test accounts.
5. Verify **no duplicate parent stylesheets** and check browser console errors.
6. Activate on staging only if owner separately authorizes it and a rollback is ready.
7. Keep Montonio inactive, payments disabled, and production unchanged.

## Status

Code review pending; no PHP runtime or WordPress integration verification is claimed. This branch is **not production-ready**.
