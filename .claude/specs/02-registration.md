# Spec: Registration

## Overview
Implement user registration so visitors can create a Spendly account. This step upgrades the existing `GET /register` stub into a fully working form that accepts a name, email, and password, validates the input, hashes the password, inserts the new user into the database, and redirects to the login page on success. It also introduces Flask sessions and `secret_key` — both required by all subsequent authenticated steps.

## Depends on
- Step 1 — Database setup (`get_db()`, `users` table, `init_db()`, `seed_db()` must be complete)

## Routes
- `GET /register` — render the registration form — public (already exists, keep as-is)
- `POST /register` — process form submission, insert user, redirect on success — public

## Database changes
No new tables or columns. The `users` table from Step 1 covers everything needed.

## Templates
- **Modify:** `templates/register.html`
  - Change `action="/register"` to `action="{{ url_for('register') }}"`
  - Re-render form with previously entered `name` and `email` values on validation error (sticky fields)

## Files to change
- `app.py` — add `POST /register` handler, import `create_user` and `get_user_by_email` from `db.py`, add `app.secret_key`
- `database/db.py` — add `create_user(name, email, password_hash)` and `get_user_by_email(email)` helpers
- `templates/register.html` — fix hardcoded action URL, add sticky field values

## Files to create
None.

## New dependencies
No new pip packages. Uses:
- `werkzeug.security.generate_password_hash` (already installed)
- `flask.session`, `flask.redirect`, `flask.url_for`, `flask.request`, `flask.flash` (all in Flask)

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only — never f-strings in SQL
- Passwords hashed with `werkzeug.security.generate_password_hash` before insert
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- `app.secret_key` must be set before any session/flash usage — use a hardcoded dev string for now (e.g. `"dev-secret-change-in-prod"`)
- `create_user` and `get_user_by_email` must live in `database/db.py`, not inline in the route
- The route function's only job: validate input, call db helpers, redirect or re-render
- On duplicate email, re-render the form with the error message — do not raise an unhandled exception
- Minimum password length: 8 characters (validate server-side)
- On success: redirect to `url_for('login')` — do not redirect to a dashboard that doesn't exist yet

## Definition of done
- [ ] Submitting the form with valid data creates a new row in `users` with a hashed password
- [ ] Submitting with an email that already exists re-renders the form with an error: "An account with that email already exists."
- [ ] Submitting with a password shorter than 8 characters re-renders the form with an error: "Password must be at least 8 characters."
- [ ] Submitting with any field empty is blocked by the `required` attribute (HTML) and re-renders the form server-side if bypassed
- [ ] Name and email fields are sticky — values are preserved on validation error
- [ ] On success, the user is redirected to `/login`
- [ ] The form action uses `url_for('register')` — no hardcoded URLs
- [ ] App starts without errors after changes
