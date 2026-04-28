---
name: getProject
description: Load a user's personal project memories into the oracle's context. Lists all project-type memory entries from the user's Claude memory files, then asks which ones to read in full so the oracle is aware of the active work.
---

# /getProject

Brings a user's personal project context into the current conversation.
The oracle's memory system stores project snapshots from previous sessions
(open PRs, active investigations, ongoing issues). This skill surfaces those
snapshots so the user doesn't have to re-explain what they're working on.

## Flow

1. **Locate the memory directory.** The user's project-scoped memory lives at:
   ```
   ~/.claude/projects/-home-<username>-jedi-oracle/memory/
   ```
   Read `MEMORY.md` in that directory. It is the index — each line maps a
   memory title to a filename and a one-line description.

2. **Filter for project memories.** Read each memory file that is referenced
   in `MEMORY.md`. Check the `type:` field in the frontmatter. Collect only
   the files where `type: project`.

3. **Display the project list.** Present the projects as a numbered list.
   For each entry show the title and the one-line description from `MEMORY.md`
   (or the `description:` field in the file's frontmatter — whichever is more
   informative). Example:

   ```
   1. CADRE ABI all-sky radiance DA issue — OU MAP lab, ufo#4120; |OMA|>|OMB| over convective clouds
   2. ...
   ```

   If no project memories exist, say so and suggest the user describe their
   current work so the oracle can save it for future sessions.

4. **Ask which to load.** Prompt: *"Which of these would you like me to load
   into context? You can pick one, several, or all."* Accept a number, a list
   of numbers, a name, or "all".

5. **Read the selected files in full** and confirm: *"I've loaded [project
   names] — ask me anything about them."*

## Notes

- Do not summarize or paraphrase project memories unprompted — read the full
  file so no detail is lost.
- If the user selects a project and the memory file references an external
  document (e.g. a working doc path or a GitHub issue URL), mention that
  reference so the user knows it exists.
- This skill only reads; it does not write or update memories. If the user
  wants to update a project memory after discussing it, offer to do so at the
  end.
- Memory files can become stale. If a loaded project memory mentions specific
  file paths, PRs, or code behavior, note that it was recorded at a point in
  time and offer to verify against current repo state if needed.
