---
framework_version: 1.5.0
---

# LinkedIn Outreach After Applying

Runs as Step 7 of `/apply`, after the CV and cover letter are finalized and the tracker row is written. Goal: identify the recruiting/hiring manager (and, sometimes, a strong second-degree connection) at the target company, draft a short personalized note, and send it — but only once the user has approved each specific message.

## Hard rule: no automation tooling, no scraping, no bot login

**Every LinkedIn action in this workflow happens through the Browser pane, driving the real linkedin.com UI with the user's own logged-in session — clicking, typing, reading the page — exactly as the user would do it by hand.** This is a deliberate, non-negotiable boundary, not a style preference:

- **Never** use a tool that logs into LinkedIn programmatically, drives it via Selenium/API automation, or scrapes profile data at scale (any auto-apply or scraping bot falls in this category). LinkedIn's User Agreement prohibits automated/bot access to the platform, and tools like this put the user's account at risk of being rate-limited or banned — a cost that falls entirely on the user, for a workflow whose whole point is protecting the candidacy.
- A single profile view or a single people-search per company, read the way a logged-in browser tab already renders it, is the human-equivalent action this workflow performs. It is not scraping.
- If the user ever names a specific automation/scraper repo or extension for this purpose, check what it actually does (fetch its README) before using it. Assume automation-category tools are out of bounds unless proven otherwise — never assume a tool is safe from its name alone.

## Step-by-step

### 1. Identify the hiring/recruiting manager (always attempt this)

Search LinkedIn (People search, company filter) for the most likely contact:
- A recruiter/talent-acquisition title at the target company ("Recruiting Manager", "Talent Acquisition Partner", "Head of Talent", "People Ops", etc.), or
- The actual manager the role reports to, when inferable from the posting or department (e.g. a "Chief of Staff to the CEO" posting → search for the CEO/COO by name if the posting names them, otherwise the most senior People/Ops leader at the company)

Open the top 1-2 candidate profiles. For each, record: name, title, connection degree (1st/2nd/3rd/out of network).

**Per the user's standing instruction: if a plausible hiring/recruiting manager contact is found, always draft outreach for them** — this is not offered contingently like the application-form-fields step; attempt it by default every `/apply` run. If no plausible contact can be found (locked profile, no recruiting/People presence visible, name genuinely unresolvable), say so plainly rather than guessing at a stranger.

### 2. Search for second-degree connections at the company (always attempt this)

Run a People search filtered by company name + connection degree = 2nd. This surfaces people the user has a mutual connection with, regardless of their role.

For every 2nd-degree result, open the profile and read the mutual-connections module LinkedIn displays natively on the page ("X, Y, and N others are mutual connections") — this is visible to the user's own logged-in session already, so reading it off the rendered page is not a scrape.

**Evaluate each 2nd-degree result for whether it's a "good" contact** — relevant to the role (same team/function, senior enough to matter, or the mutual connection itself is a strong signal) versus someone at the company in an unrelated function with only a weak/incidental mutual link. Do not pre-filter these out of the report; the user wants to see the finding and make the call themselves. **Only surface someone as a candidate worth messaging if they are a genuine 2nd-degree connection (a mutual exists) — do not propose reaching out to a 1st-degree stranger-equivalent or out-of-network person under this step**, since the entire value of this angle is the mutual-connection warm path.

### 3. Report findings before drafting anything

Present to the user:
- The hiring/recruiting manager candidate(s) found (name, title, connection degree, **and their LinkedIn profile URL — standing rule: always include the actual profile link when naming a person found this way, in every report and every send confirmation, not just the first mention**)
- Every 2nd-degree connection found at the company, with their mutual connections listed, their profile URL, and a plain-language read on whether it looks like a good one to use ("relevant — same function, VP-level, mutual is a former colleague" vs. "weak — different department, mutual is only a shared alumni connection")

Do not draft messages yet — the user picks who (beyond the default hiring-manager contact) is worth a note.

### 4. Draft the outreach note(s)

For each approved contact:
- Tailor the note to **this specific application** — pull the actual "why I'm a fit" reasoning from the cover letter/CV just written for this company, never a generic template.
- **Check the actual connection state on their profile before drafting length.** Not-yet-connected contacts get a **connection request note, which LinkedIn caps at ~300 characters** — short and specific, not the full paragraph style. Already-connected or InMail-eligible contacts can get a longer message. Confirm which applies by looking at the profile's actual button state (Connect vs. Message) rather than assuming.
- **For the 300-char case specifically, the "requirement → proof" compression is the fastest way to hit that limit without sounding cut off:** pick the single strongest requirement-to-your-evidence pair from the CV/cover letter just written, state it in one clause, done. Don't try to fit a full pitch.
- Keep the tone and content consistent with `03-writing-style.md` (the user's voice, no AI-sounding constructions, no cliches) — the same bar as the CV and cover letter.

Show every drafted note to the user before sending anything.

### 5. Send only on explicit per-message approval

**This step never sends on its own, no matter how many times the user has approved this workflow generally.** Confirm each specific note in chat, then send it by driving the real LinkedIn UI in the Browser pane (click Connect/Message, type the note, click Send) — one message, one confirmation, one send. Never batch-send multiple notes off a single approval.

### 6. Log what was sent

After a send, record it so a later `/apply` or a rerun of this step doesn't duplicate outreach:
- Update the `contact_person` and `notes` columns of the matching `job_search_tracker.csv` row: `contact_person` gets the name(s) reached out to; `notes` gets a dated line, e.g. `YYYY-MM-DD: LinkedIn outreach sent to Jane Doe (Head of Talent)`.
- If a good 2nd-degree contact was found but the user chose not to reach out, note that too (undecided/declined), so it isn't re-surfaced as new next time.
