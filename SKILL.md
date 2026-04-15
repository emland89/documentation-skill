---
name: documentation-audit
description: Audit, create, and maintain structured codebase documentation that works for both human readers and AI agents. Use when asked to audit docs against source code, write new subsystem documentation, update stale docs after a refactor, or set up a Docs/ structure for a project.
---

# Documentation Audit & Authoring

## Documentation Format

### File vs folder layout

**Default: one file per feature at `Docs/` root.** Only create a subfolder when a single file would cover multiple independently-navigable topics.

```
Docs/
  index.md                  ← top-level routing table
  auth.md                   ← single-topic feature at root
  decisions.md              ← single-topic feature at root
  player/
    index.md                ← feature overview + shared diagram + routing
    player-video-player.md  ← detail file, prefixed with folder name
    player-rendering.md     ← detail file, prefixed with folder name
  database/
    index.md
    database-models.md
    database-query-layer.md
```

**Files in a subfolder are prefixed with the folder name** (`player/player-video-player.md`, not `player/video-player.md`). This keeps files identifiable when opened in tabs and searchable without folder context.

Use a folder when the topics are independently navigable — someone working on rendering doesn't need to read the player facade. Use a root file when the topics are tightly coupled.

### index.md (top-level)

Routing only: a `| Path | Covers |` table pointing to every root file and every subfolder `index.md`. No narrative, no diagrams.

### index.md (folder)

Not a routing table — a real document. Structure:
1. **Heading + one dense paragraph** — what this feature IS, the key pattern, and what the detail files cover
2. **`## Contents`** — `| File | Covers |` routing table for detail files in this folder
3. **`## Quick Reference`** — key facts, thresholds, enum values; structured for scanning
4. **`## Diagram`** — one shared Mermaid diagram showing the feature's overall structure (class hierarchy, ER, component graph)

### Detail file structure

Progressive disclosure: most important content first so an AI can stop reading early; a human reading top-to-bottom gets full docs.

```
<!-- AI-SUMMARY: one dense paragraph — what this IS, key pattern, critical numbers.
KEY TYPES: TypeA, TypeB, TypeC -->

## [What this is] / Mental Model
One paragraph + optional ASCII diagram. Answers: what is this thing and how does it fit?

## Why It Exists  (optional — include when the design is non-obvious or refactor-prone)
What alternatives exist. What breaks if someone ignores this design.

## How It Works
Full lifecycle, internals, data flow. Actual type names, thresholds, timings from source.
Sub-sections per major phase or component.

## Key Types
| Type | Role |

## Common Patterns
Concrete code snippets for the most frequent call sites and extension points.

## Pitfalls
Actionable failure modes. Never "be careful with X" without the failure mode and fix.

## Diagram  (optional — if the file warrants its own diagram beyond what's in the folder index)
```mermaid
...
```
```

**AI-SUMMARY purpose:** lets an agent decide whether to read the full file without reading it. Must be accurate, dense, and list all key types. Write it last — verify it against the finished file.

See `references/detail-template.md` and `references/index-template.md` for copy-paste starting points.

---

## Audit Workflow

Run this when docs may be stale. Recurse until the discrepancy list is empty.

### Step 1 — Scope

Read `Docs/index.md` (or `Docs/INDEX.md`) to list all features. Check `git log --oneline -20` to see which subsystems had recent source changes. Queue only those for audit unless a full audit was requested.

### Step 2 — Parallel agent audit

Spawn one agent per feature (root file or subfolder). Each agent:

1. Reads the feature's file(s): the root `.md` or the subfolder `index.md` + all prefixed detail files
2. Reads the actual source files for that feature
3. Produces a discrepancy list in this exact format:

```
WRONG:   docs say X
ACTUAL:  source says Y
FILE:    Docs/feature/file.md line N
SOURCE:  src/Feature/SourceFile.swift line N
```

### Step 3 — Fix and verify

Fix each discrepancy in the docs. After fixing, re-read the changed sections and verify the fix is correct. Do not modify source code during a doc audit.

### Step 4 — Recurse

Re-run the audit on any feature whose docs were just changed. Stop when the discrepancy list is empty.

### Common discrepancy categories

- Wrong property or method name (renamed after refactor)
- Wrong method signature (parameter labels or types changed)
- Wrong enum cases (added, removed, or renamed)
- Wrong capability (service returns `[]` but docs say it's supported)
- Structural mismatch (hierarchy described no longer exists)
- Missing fields (diagram or property table omits new columns)
- Wrong file path (source moved, docs still reference old location)

---

## Authoring Workflow

### Writing a root-level single file

Use this for features that fit in one document.

1. **Read all source files first** — extract every public type, protocol + implementations, full lifecycle, thresholds/timings, concurrency model, non-obvious decisions
2. **Write AI-SUMMARY** — one dense paragraph covering: what this IS, key pattern, critical numbers. Then `KEY TYPES:` list. Test it: could an agent route to this file from the summary alone?
3. **Write the opening section** — one paragraph mental model + optional ASCII diagram
4. **Why It Exists** — only if the design is non-obvious or refactor-prone
5. **How It Works** — bulk of the file; use actual type names, real numbers, describe the sequence not just the components
6. **Key Types table** — only types that appear in How It Works
7. **Common Patterns** — actual code from the project, not pseudocode; the most common call site + how to extend
8. **Pitfalls** — each one: what goes wrong + why + how to avoid
9. **Diagram** — Mermaid if the file benefits from one (sequence, class hierarchy, data flow)

### Writing a subfolder

Use this for features with multiple independently-navigable topics.

1. **Plan the split** — identify which topics are independently navigable; each topic becomes one prefixed detail file
2. **Write `index.md` first** — feature description paragraph, routing table, quick reference, shared diagram (the overview-level diagram for the whole feature)
3. **Write each detail file** — follow the single-file authoring steps above; the detail file's AI-SUMMARY should be specific to that file's scope, not repeat what the index says
4. **Name detail files with the folder prefix** — `player/player-video-player.md`, not `player/video-player.md`

### Quality Checklist

Before marking any doc done:

- [ ] AI-SUMMARY is accurate, dense, and includes all KEY TYPES
- [ ] Every type in Key Types appears in How It Works
- [ ] Every threshold/number was verified in source (not from memory)
- [ ] Pitfalls are grounded in real failure modes, not speculation
- [ ] No section is empty or says "TBD"
- [ ] All file paths referenced exist in their current form
- [ ] Method signatures match actual source (parameter labels, return types)
- [ ] For folder `index.md`: diagram reflects current structure, contents table lists current filenames with prefix

---

## Arguments

If `$ARGUMENTS` is provided, treat it as the feature name or file to audit/write. Examples:
- `player` → audit or write `Docs/player/` (subfolder + all detail files)
- `search` → audit or write `Docs/search.md` (single root file)
- `Docs/player/player-rendering.md` → audit or rewrite that specific file
- _(empty)_ → audit all features with recent git changes

ARGUMENTS: $ARGUMENTS
