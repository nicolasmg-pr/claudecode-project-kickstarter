---
name: fresh-docs
description: Verify current official documentation before any framework-specific work. Use before coding against Next.js, React, Supabase, Stripe, Tailwind, or any fast-moving library, or when the user asks about a framework API.
---

# Fresh Docs

Rule: **never assume training knowledge is current for anything framework-related.** Training is a snapshot; the library is a moving target. Applies to Next.js, React, Supabase, Stripe, Tailwind CSS, and anything that releases regularly.

## The loop: search → paste → cite

1. Pin the version: read `package.json` / lockfile / `requirements.txt` to know exactly what the project uses.
2. WebSearch for the official docs page covering the task; WebFetch that page. Official docs over blog posts.
3. Paste the relevant snippet into the working context and cite the source URL.
4. Durable snippet (needed across sessions)? Save it to `docs/` as a small scoped file: one topic, source URL, date fetched.
5. Write code that matches the fetched docs, not memory. If the docs describe a newer version than the project uses, flag the mismatch before coding.

Do not skip the fetch because the API "feels familiar" — that feeling is exactly the failure mode this skill prevents.
