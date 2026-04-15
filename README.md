# Documentation Skill

An agent skill for auditing, creating, and maintaining structured codebase documentation that works for both human readers and AI agents. Compatible with any agent that supports the [Agent Skills](https://github.com/vercel-labs/skills) standard.

## Install

```bash
npx skills add emland89/documentation-skill -g
```

## What it does

- **Audit** existing docs against source code to find discrepancies (wrong names, signatures, missing fields, stale paths)
- **Author** new documentation files following a consistent structure optimized for AI navigation
- **Maintain** docs over time with a recursive audit workflow that runs until all discrepancies are resolved

## Documentation structure

Single-topic features get one file at the `Docs/` root. Multi-topic features get a subfolder with an `index.md` and prefixed detail files.

```
Docs/
  index.md                  ← top-level routing table (paths + one-line covers)
  auth.md                   ← single-topic feature at root
  decisions.md              ← single-topic feature at root
  player/
    index.md                ← feature overview: description + diagram + routing
    player-video-player.md  ← detail file, prefixed with folder name
    player-rendering.md     ← detail file, prefixed with folder name
  database/
    index.md
    database-models.md
    database-query-layer.md
```

**Folder `index.md`** is a real document — not a routing table. It has: a dense feature-description paragraph, a `## Contents` routing table to its detail files, a `## Quick Reference` section with key facts and thresholds, and a `## Diagram` section with a shared Mermaid overview.

**Detail files** use progressive disclosure — most important content first so an AI can stop reading early; a human reading top-to-bottom gets full docs:

```
<!-- AI-SUMMARY: one dense paragraph — what this IS, key pattern, critical numbers.
KEY TYPES: TypeA, TypeB, TypeC -->

## Mental Model
One paragraph + optional ASCII diagram.

## Why It Exists  (optional)
Design alternatives and what breaks if someone ignores this design.

## How It Works
Full lifecycle, internals, data flow. Real type names, thresholds, timings.

## Key Types
| Type | Role |

## Common Patterns
Concrete code snippets for common call sites and extension points.

## Pitfalls
Actionable failure modes — what goes wrong, why, and how to fix it.

## Diagram  (optional)
```mermaid
...
```
```

Diagrams are inline Mermaid fenced blocks. Use `classDiagram` or `erDiagram` in folder `index.md` for the overall structure, and `sequenceDiagram` or focused `classDiagram` in detail files when a file-specific diagram adds value.

## Usage

Once installed, invoke the skill to:

- Audit all recently changed subsystems: `/documentation-audit`
- Audit a specific subsystem: `/documentation-audit player`
- Audit a specific file: `/documentation-audit Docs/player/player-rendering.md`
- Write docs for a new feature: `/documentation-audit auth`

## Templates

- `references/detail-template.md` — starting point for detail files
- `references/index-template.md` — starting point for folder `index.md` files
