# SSRF — Server-Side Request Forgery

**OWASP category:** A10:2021 Server-Side Request Forgery
**Studied:** 2026-09-07

## What it is
An app takes user-supplied input — a URL, hostname, or IP — and uses it to make an HTTP/network request *from the server*, without validating where that request is actually allowed to go. The attacker manipulates the input so the server fetches something it shouldn't: an internal-only service, localhost, or (the high-value target) a cloud metadata endpoint.

## How it happens (mechanism)
Any feature where the server fetches a URL on the user's behalf is a candidate: webhooks, "import from URL", avatar/image fetchers, PDF/report generators, URL preview cards. The server usually sits inside a trust boundary (VPC, internal network, cloud instance) the attacker can't reach directly — SSRF turns the server itself into a proxy into that boundary. In cloud environments specifically, the instance metadata service (e.g. AWS's `169.254.169.254`) hands out temporary IAM credentials to *anything that can reach it from the instance* — no extra auth required under the older protocol version.

## Real-world example
**Capital One breach, disclosed July 2019.** Paige Thompson found an SSRF vulnerability in a misconfigured WAF running on an AWS EC2 instance. She used it to query the AWS instance metadata service (running IMDSv1, unauthenticated), pulled temporary credentials for the WAF's over-permissioned IAM role, then used those credentials to read from 700+ S3 buckets — roughly **106 million customer records** exposed. Capital One paid an $80M OCC penalty and a $190M class-action settlement. AWS's response was IMDSv2, which requires a session token and closes exactly this path.

## How to test for it
1. Find any feature where the server fetches a URL you control (webhook target, "fetch from URL", report generator).
2. Point it at internal-only targets: `http://127.0.0.1`, `http://localhost`, internal hostnames, or (in a cloud lab) the metadata IP `169.254.169.254/latest/meta-data/`.
3. Check if the response leaks internal data back to you.
4. If nothing comes back directly (blind SSRF), watch for an outbound callback via a listener you control (e.g. webhook.site / Burp Collaborator) to confirm the request fired.

## How to fix it
- **Allowlist** destinations rather than blocklist — blocklists get bypassed via redirects, DNS rebinding, or alternate IP encodings (decimal/octal/IPv6-mapped).
- Disable unused URL schemes (`file://`, `gopher://`, etc.) in whatever library does the fetching.
- On AWS specifically: enforce IMDSv2 (`--http-tokens required`) so the metadata endpoint needs a session token — this alone would have stopped the Capital One chain at step one.
- Network-segment app servers so they can't reach internal services/metadata by default, regardless of app-layer bugs.
- Re-validate/re-resolve the destination at request time, not just at input-validation time (defeats DNS rebinding).

## Where I'm applying this
Building a tiny local Flask app with a "fetch this URL" endpoint as a `tools/` mini-project — feeling the mechanism by triggering it myself teaches it faster than reading about it. Also worth checking whether any SPOX feature fetches a user-supplied URL server-side, and if so, confirming it's allowlisted.

**Sources:** Krebs on Security / multiple incident write-ups on the Capital One breach (July 2019); OWASP Top 10 — A10:2021.
