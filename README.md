# jedi-oracle

A Claude Code companion repo for working with the JEDI bundle and surrounding
JCSDA tools. The intent is simple: clone this repo, open it in Claude Code, and
have a knowledgeable collaborator that can answer questions about any of the
JEDI repositories — what they do, how they fit together, what changed
recently, and how to use them.

This repo itself does not contain JEDI source code. Instead, it ships:

- **`CLAUDE.md`** — instructions that turn Claude into the JEDI oracle.
- **`jedi-knowledge/`** — one knowledge brief per JEDI repository, plus shared
  tips (`jedi-tips.md`) and a workflow overview (`workflow.md`).
- **`.claude/skills/`** — slash commands that fetch code, keep it current, and
  let you grow the knowledge base over time.

On first use, Claude will offer to clone `jedi-bundle`, `jedi-docs`,
`jedi-tools`, and `jedi-workflow` (containing skylab, ewok, simobs, r2d2) into
this directory. Those clones are gitignored — they are working checkouts, not
part of the oracle repo.

## Quick start

```bash
git clone https://github.com/<your-org>/jedi-oracle.git
cd jedi-oracle
claude
```

On first interaction Claude will walk you through the init flow and tell you
about the available skills.

## Available skills

- **`/jedi-oracle`** — entry point. Reads `CLAUDE.md` and gives you a
  description of every available skill. Type this any time you want a
  refresher on what the oracle can do.
- **`/jedi-getCode`** — clone repositories listed in `jedi-bundle/CMakeLists.txt`,
  optionally enabling optional packages (RTTOV, OASIM, ROPP, GSIBEC,
  IODA-converters, PyIRI).
- **`/jedi-updateRepos`** — pull `develop` on every cloned repo, summarize the
  diffs, and (optionally) refresh the knowledge base to match.
- **`/jedi-addKnowledgeBase`** — extend the oracle with an external resource:
  another GitHub repo, a web page, a paper, etc.
- **`/jedi-getProject`** — load your personal project memories from previous
  sessions so the oracle picks up active work without you re-explaining.
- **`/jedi-investigateIssue`** — search GitHub issues you're involved in by
  keyword, pick one, then pull the full issue with `gh` and produce a
  research report covering the problem and possible solutions.

## Contributing

`jedi-knowledge/jedi-tips.md` is meant to be team-curated. Add commands,
gotchas, and recipes you've found useful. PRs welcome.

## License

Apache 2.0 (same as the JEDI bundle).
