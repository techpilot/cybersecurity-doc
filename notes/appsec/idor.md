# IDOR — Insecure Direct Object Reference

**OWASP category:** A01:2021 Broken Access Control (also API1:2023 Broken Object Level Authorization under the OWASP API Security Top 10)
**Studied:** 2026-09-07

## What it is
An app exposes a direct reference to an internal object — a DB row ID, order number, filename, invoice ID — in a URL, parameter, or request body, and fails to check that the *requesting user is actually authorized to access that specific object*. The attacker doesn't need to break anything technically; they just change the ID.

## How it happens (mechanism)
Most apps correctly check **authentication** ("is this a logged-in user?") but skip **object-level authorization** ("is *this* user allowed to see *this* record?"). Two flavors:
- **Horizontal IDOR** — same-privilege user reads/modifies another same-tier user's object (user A reads user B's invoice).
- **Vertical IDOR** — lower-privilege user reaches a higher-privilege or admin-scoped object.
- **Write-based IDOR** — the tampered ID lands on a PUT/PATCH/DELETE, not just a GET — the dangerous variant most people under-test.

## Real-world example
**USPS "Informed Visibility" API, disclosed Nov 2018 (Brian Krebs).** Any logged-in usps.com user could query account details — email, username, account number, address, phone — for roughly **60 million other users**, and in some cases modify them. Root cause: the API accepted wildcard/ID-based queries and checked *that* you were logged in, never *whether* the record you asked for belonged to you. It sat unpatched for over a year until public disclosure forced a fix.

## How to test for it
1. Find an endpoint that returns an object by ID while authenticated as User A (e.g. `GET /api/orders/1042`).
2. Swap the ID for one you don't own (increment/decrement, or a second test account's known ID), same session/token.
3. Check if data comes back that shouldn't.
4. Repeat on write methods (PUT/PATCH/DELETE), not just GET.
5. Repeat across privilege tiers (regular user → admin-scoped object).

## How to fix it
- Enforce object-level authorization **server-side, on every request** — scope every query to the requesting user (`WHERE user_id = current_user`), never trust the client-supplied ID alone.
- Centralize the authorization check (middleware/policy layer) instead of re-implementing it per endpoint — that's where it gets forgotten.
- Indirect/non-guessable references (UUIDs) are defense-in-depth, not the fix — they reduce casual enumeration but don't replace an authorization check.

## Where I'm applying this
Good candidate to test hands-on against OWASP Juice Shop this week (it has intentional IDOR-style challenges). Also a useful lens to re-review SPOX's Auth0-based endpoints against — checking that IAM handles *authentication*, but object-level checks are a separate thing to verify per-endpoint.

**Sources:** Krebs on Security, "USPS Site Exposed Data on 60 Million Users" (Nov 2018); OWASP API Security Top 10 — API1:2023.
