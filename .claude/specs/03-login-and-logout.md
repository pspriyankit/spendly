# Spec: Login and Logout

## Overview
Implement user login and logout so registered users can authenticate into Spendly and end their session. This step upgrades the existing `GET /login` stub into a fully working form that accepts an email and password, verifies credentials against the database, writes the user's id and name into the Flask session on success, and redirects to the dashboard (stub placeholder for now). Logout clears the session and redirects back to the landing page. After this step, the app has a complete authentication boundary that all subsequent steps (profile, expenses) can gate behind.

## Depends on
- Step 1 — Database setup (`get_db()`, `users` table, `init_db()`)
- Step 2 — Registration (`create_user`, `get_user_by_email`, `app.secret_key` set)

## Routes
- `GET /login` — render the login form — public
- `POST /login` — validate credentials, write session, redirect on success — public
- `GET /logout` — clear session, redirect to landing — logged-in (safe to call even when logged out)

## Database changes
No database changes. The `users` table from Step 1 already stores `email` and `password_hash`.

## Templates
- **Create:** `templates/login.html`
  - Form with `email` and `password` fields
  - Action: `{{ url_for('login') }}` method POST
  - Inline error message block (shown only when `error` is passed)
  - Email field is sticky (re-populated on error)
  - Extends `base.html`
- **Modify:** `templates/base.html`
  - Add conditional nav links: show "Logout" when `session.user_id` is set, show "Login" and "Register" otherwise

## Files to change
- `app.py`
  - Import `session` and `check_password_hash` (or use `werkzeug.security.check_password_hash`)
  - Upgrade `GET /login` stub to a full `GET /POST /login` route
  - Implement `GET /logout` route — clear session, redirect to `url_for('landing')`
- `templates/base.html` — add session-aware nav links
- `templates/login.html` — build out the login form (file may already exist as a stub; rewrite it)

## Files to create
- `templates/login.html` (create or replace existing stub)

## New dependencies
No new pip packages. Uses:
- `werkzeug.security.check_password_hash` (already installed with Flask)
- `flask.session` (built into Flask)

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only — never f-strings in SQL
- Password verification with `werkzeug.security.check_password_hash` — never compare plaintext
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Session keys: store `session['user_id']` (integer) and `session['user_name']` (string) on successful login
- `get_user_by_email` already exists in `database/db.py` — use it, do not duplicate
- On wrong email or wrong password: show the same generic error "Invalid email or password." — never reveal which field was wrong
- On successful login, redirect to `url_for('landing')` for now (dashboard does not exist yet)
- `logout` must call `session.clear()` then redirect to `url_for('landing')`
- Do not implement any `@login_required` decorator yet — that belongs to a later step

## Definition of done
- [ ] Submitting the form with correct email and password sets `session['user_id']` and `session['user_name']` and redirects to `/`
- [ ] Submitting with an unknown email re-renders the form with the error "Invalid email or password."
- [ ] Submitting with a correct email but wrong password re-renders the form with the same error
- [ ] Email field is sticky — value is preserved on failed login
- [ ] Visiting `/logout` clears the session and redirects to the landing page
- [ ] The nav in `base.html` shows "Login" / "Register" when not logged in, and "Logout" when logged in
- [ ] The form action uses `url_for('login')` — no hardcoded URLs
- [ ] App starts without errors after changes
