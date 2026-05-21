---
name: jedi-addKnowledgeBase
description: Add an external resource (GitHub repo, web page, local PDF, etc.) to the oracle's knowledge base. Writes a per-resource brief to jedi-knowledge/external/ and indexes it in externalKnowledge.md.
---

# /jedi-addKnowledgeBase

Extend the oracle's knowledge base with an external resource so future
questions can draw on it. Output goes to:

- `jedi-knowledge/external/<name>.md` — the per-resource brief (gitignored)
- `externalKnowledge.md` — index of external resources at the oracle
  root (gitignored, created on first use)

Both are gitignored: external knowledge is per-user, not part of the
shared oracle repo.

## Flow

1. **Ask what to add.** Offer common types but accept anything:

   > *"What would you like to add to the knowledge base? Common options:
   > another GitHub repo, a web page or doc URL, a local PDF or paper, a
   > Slack/Discord channel, internal docs. Anything else? Just describe
   > it."*

2. **Get the resource details** based on type:
   - **GitHub repo** — URL and branch (default `develop` or `main`).
   - **Web page** — URL.
   - **Local PDF / paper** — absolute file path.
   - **Other** — ask the user how the oracle should ingest it. If it's
     something Claude can't fetch directly (e.g. a Slack channel), record
     where to find it and what topics it covers — that pointer alone is
     useful.

3. **Pick a short slug** for the file name (kebab-case, e.g.
   `mpas-atmosphere-paper`, `ecmwf-blog-bias-correction`). Confirm with
   the user if ambiguous.

4. **Ingest the resource:**
   - **GitHub repo** — `git clone --depth=1` into a temp dir, scan
     README + top-level dirs + key source files, then delete the temp
     clone. Don't keep the clone around — the brief is what we're
     building.
   - **Web page** — fetch with WebFetch.
   - **Local PDF** — read with the Read tool (use the `pages` parameter
     for large PDFs).
   - **Other** — record the pointer and any context the user gave.

5. **Write `jedi-knowledge/external/<slug>.md`** with this structure:

   ```markdown
   # <Title>

   **Source:** <URL or path>
   **Type:** <github-repo | web-page | pdf | other>
   **Added:** <ISO date>

   ## What this is
   <1–3 sentence description>

   ## Why it's useful for JEDI work
   <how it relates to the bundle, workflow, or domain>

   ## Key takeaways / structure
   <bullets — what's in it, where to look for what>

   ## When to consult this
   <questions or scenarios where the oracle should reach for this>
   ```

6. **Update `externalKnowledge.md`** at the oracle root. Create it on
   first use with this header:

   ```markdown
   # External knowledge base

   Per-user index of external resources added via /jedi-addKnowledgeBase.
   Gitignored — these are local to your checkout.

   ## Resources

   - [<Title>](jedi-knowledge/external/<slug>.md) — <one-line hook>
   ```

   On subsequent runs, append to the `## Resources` list. Keep entries
   to one line each.

7. **Confirm to the user**: print the brief's path and the one-line
   index entry. Tell them the oracle will now consult this resource for
   relevant questions (CLAUDE.md instructs Claude to check
   `externalKnowledge.md` when present).

## Notes

- These files are gitignored intentionally — different users will care
  about different external resources, and some may be private.
- If the user wants an external resource shared with the team, they
  should add it to `jedi-knowledge/jedi-tips.md` (tracked) instead, or
  open a PR to add a tracked brief.
- For very large resources (a 500-page paper, an entire codebase),
  ask the user to scope: which chapters? which subsystem? Don't try to
  ingest everything.
