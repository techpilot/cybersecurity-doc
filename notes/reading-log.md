# Reading Log

Short, dated. One takeaway per entry — not a summary of the article, the thing I'll actually remember.

---

**2026-09-07** — Krebs on Security (USPS API, 2018) + Capital One SSRF post-mortems.
Takeaway: both breaches trace to the same root pattern — a check that exists ("is this request authenticated / can this server make requests") standing in for a check that doesn't ("is this specific object/destination allowed"). Access control bugs are rarely about _breaking_ anything technically — they're about a missing question in the code.

**2026-09-10** — Published a LinkedIn post on top 3 2026 cyberattacks - **Cybersecurity Hall of Shame**. Researching it for the post doubled as today's reading. Takeaway: <the one thing that changed how you think about it>
