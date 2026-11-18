# Code Review Report: Spendly

**Date:** 2026-11-18
**Scope:** `app.py`, `database/`, `templates/`, `static/css/`, `static/js/`, `requirements.txt`, `.gitignore`, git state

## Summary

Spendly is at an early stage. The public pages (landing, login, register, terms, privacy) render, but there is no database, no authentication and no expense features yet. The highest-priority issue is that the login and register forms cannot submit: they POST to routes that only accept GET. The rest of the findings are either planned work (the stubs) or minor cleanup.

## Findings

### High

**1. Login and register forms cannot be submitted**
- **Where:** `templates/login.html` (`<form method="POST" action="/login">`), `templates/register.html` (`<form method="POST" action="/register">`), `app.py` (`@app.route("/login")`, `@app.route("/register")`)
- **Problem:** Both routes only handle GET, so a form submit returns HTTP 405 Method Not Allowed.
- **Fix:** When these features are built, add `methods=["GET", "POST"]` and handle the POST. Until then, the forms are non-functional.

**2. No database layer**
- **Where:** `database/db.py`, `database/__init__.py`
- **Problem:** Both files contain only comments. There is no `get_db()`, `init_db()` or `seed_db()`, so nothing can be stored.
- **Fix:** Implement Step 1 (`Spec_01_database_setup_prompt.md`).

### Medium

**3. Debug mode is hard-coded**
- **Where:** `app.py:65`, `app.run(debug=True, port=5001)`
- **Problem:** The Werkzeug debugger allows arbitrary code execution from the browser when reachable. This is fine locally but must not ship.
- **Fix:** Read the debug flag from an environment variable, defaulting to off.

**4. No secret key or session handling yet**
- **Where:** `app.py`
- **Problem:** Login (Step 3) will need `app.secret_key` and a session. Nothing is configured.
- **Fix:** Load the key from an environment variable when Step 3 is built. Do not hard-code it.

**5. Privacy Policy describes behaviour the app does not have yet**
- **Where:** `templates/privacy.html`
- **Problem:** The text says passwords are "stored only in hashed form" and that users can delete their expenses. Neither exists yet. The contact address `support@spendly.example` is a placeholder.
- **Fix:** Keep the policy in line with what is actually built, and replace the placeholder email before launch.

**6. Terms and Privacy pages are not a legal review**
- **Where:** `templates/terms.html`, `templates/privacy.html`
- **Problem:** The wording is generic and written by the assistant, not reviewed.
- **Fix:** Have a qualified person review both pages before publishing.

### Low

**7. Unused CSS from the old hero**
- **Where:** `static/css/style.css`, roughly lines 123–260 and the `@media` rules that target `.hero*`
- **Problem:** The hero now uses the `hs-*` classes in `static/css/landing.css`. The `.hero*`, `.mock-*` and `.hero-visual` rules are no longer referenced by any template.
- **Fix:** Delete them after checking that no other page uses them.

**8. Landing styles load only on the landing page, but the rest of the design is split**
- **Where:** `static/css/landing.css`, `templates/landing.html`
- **Problem:** The hero uses its own stylesheet and hard-coded colours, not the `:root` tokens in `style.css` (`--ink`, `--accent`, and so on). Future changes to the theme will not reach the hero.
- **Fix:** Move the hero colours to the existing tokens, or add tokens for the new ones.

**9. Modal has no focus trap**
- **Where:** `templates/landing.html`, video modal script
- **Problem:** While the modal is open, Tab can still move focus to the page behind it. Close and Escape both work, and focus returns to the button.
- **Fix:** Keep focus inside the dialog while it is open, or set `inert` on the rest of the page.

**10. Placeholder YouTube video ID**
- **Where:** `templates/landing.html`, `data-src` on `#video-modal-iframe`
- **Problem:** The ID `M7lc1UVf-VE` is a placeholder and is noted in the summary to the user.
- **Fix:** Replace it before release.

**11. Unfinished placeholder routes**
- **Where:** `app.py`, `/logout`, `/profile`, `/expenses/add`, `/expenses/<id>/edit`, `/expenses/<id>/delete`
- **Problem:** They return plain text. This is intended as scaffolding.
- **Fix:** Replace each one when its step is built.

**12. No tests**
- **Where:** `requirements.txt` lists pytest and pytest-flask, but there is no `tests/` directory.
- **Fix:** Add tests alongside Step 1 and each later step.

**13. Line-ending warnings**
- **Where:** git output for `static/css/style.css`, `templates/privacy.html`, `templates/terms.html`, `templates/landing.html`, `.gitignore`
- **Problem:** The repo has no `.gitattributes`, so git warns about LF/CRLF conversion on Windows.
- **Fix:** Add a `.gitattributes` file to set one convention.

**14. Large image in the working tree**
- **Where:** `spendly.png` (2.8 MB, untracked)
- **Problem:** A design reference image sits in the project root. It is not committed.
- **Fix:** Move it to a `docs/` or `design/` folder if it should be kept in the repo, or add it to `.gitignore`.

### Informational

- **Routes:** Only the landing, login, register, terms and privacy routes are real. All others are placeholders.
- **Dependencies:** `requirements.txt` pins Flask 3.1.3, Werkzeug 3.1.6, pytest 8.3.5 and pytest-flask 1.3.0. Pinned versions are good for a teaching project.
- **Git state:** `templates/landing.html` is modified, and `static/css/landing.css` and `spendly.png` are untracked. None of these changes are committed.
- **Footer:** The Terms and Privacy links are on every page, because they are in `base.html`.

## Recommended order

1. Implement the database (Finding 2).
2. Make login and register work end to end (Finding 1), with hashing, sessions and a secret key (Findings 4 and 5).
3. Turn off debug mode for anything beyond local use (Finding 3).
4. Clean up dead CSS and tidy the stylesheets (Findings 7 and 8).
5. Add tests as each step lands (Finding 12).
