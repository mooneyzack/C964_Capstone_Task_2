# Integrated Content Pipeline: MushroomOracle + mushroom-oracle

**Date:** 2026-05-01  
**Status:** Approved  
**Priority order:** Generation → Quality → Pipeline

## Summary

Combine the MushroomOracle Obsidian vault (curated content) with the mushroom-oracle MCP server (research evidence) into an integrated content pipeline. Keep repos separate but ingest the vault into the MCP server so it can cross-reference curated claims against published research. Enables evidence-backed content generation, quality auditing, and one-way CMS sync to ShroomSpy's Payload CMS.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Claude Code (Authoring)                  │
│  Slash commands: /new-species, /audit, /gaps, /sync         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────┐          ┌──────────────────────┐    │
│  │  MushroomOracle  │ ingest   │   mushroom-oracle    │    │
│  │  (Obsidian Vault)│─────────▶│   (MCP Server)       │    │
│  │  148 .md files   │          │   SQLite + FTS5 +    │    │
│  │                  │◀─────────│   sqlite-vec         │    │
│  │                  │ generate │   68 research sources │    │
│  └──────────────────┘          │   + vault as source   │    │
│           │                    └──────────┬───────────┘    │
│           │                               │                 │
│           │         validate              │                 │
│           │◀──────────────────────────────┘                 │
│           │                                                 │
│           ▼                                                 │
│  ┌──────────────────┐                                      │
│  │  Payload CMS     │  (one-way sync: vault → CMS)         │
│  │  (ShroomSpy)     │                                      │
│  └──────────────────┘                                      │
└─────────────────────────────────────────────────────────────┘
```

## Section 1: Vault Ingestion

Add a markdown extractor to mushroom-oracle's ingestion pipeline.

### Requirements

- **Source type tag** — chunks from the vault are marked `source_type: "vault"` so tools can distinguish curated claims from research evidence
- **Re-ingestion command** — `pnpm ingest:vault /path/to/MushroomOracle` to run after vault changes
- **Heading-aware chunking** — split on `##` headings so each chunk is a coherent section. Fallback to character-based splitting (1000 chars, 200 overlap) for long sections.
- **Frontmatter extraction** — pull species names, tags, aliases from YAML frontmatter into chunk metadata
- **Incremental** — skip files unchanged since last ingestion (by mtime or hash)

### Changes to mushroom-oracle

- New file: `src/ingest/extractors/markdown.ts`
- Update: `src/ingest/index.ts` — register markdown extractor, add `--vault` flag
- Update: `src/db/schema.ts` — add `source_type` column to `sources` table
- New CLI command: `bin/ingest.ts` gains `vault` subcommand

## Section 2: MCP Tools for Generation

### Updated tools

- **`content_gap_analysis`** — cross-references vault coverage against taxon reference (1,333 species). Returns species with strong research evidence but no vault page.
- **`draft_species_page`** — generates markdown matching vault template structure, pre-filled with cited claims. Outputs directly to vault directory.
- **`suggest_blog_topics`** — factors in existing vault + CMS content to avoid duplication.

### New tools

- **`expand_section`** — given a vault page path and section heading, pulls additional evidence and returns expanded text with citations.
- **`validate_draft`** — runs a page against the evidence base. Flags unsupported claims, missing look-alikes, contradictions with existing vault content. Returns issues with severity levels.

### Authoring workflow

1. `content_gap_analysis` → pick a species
2. `draft_species_page` → writes markdown to vault
3. Review/edit in Claude Code
4. `validate_draft` → fix any flags
5. Commit

## Section 3: Quality Auditing

### Batch audit

- CLI command: `pnpm audit:vault`
- Iterates through all vault species pages
- Runs `audit_species` on each
- Outputs markdown report grouped by severity

### Single-page audit

- Interactive, called during authoring via MCP tool
- Same `audit_species` / `audit_article` tools already in the server

### Drift detection

- Compares vault claims against CMS content for the same species
- Flags divergences (vault updated but CMS wasn't, or vice versa)

### Severity levels

- `error` — factual inaccuracy, missing dangerous look-alike (safety-critical)
- `warning` — unsupported claim, outdated taxonomy
- `suggestion` — additional evidence available, section could be expanded

### Automation

Batch audit runs on a schedule (nightly or weekly), produces a report for review. Infrastructure TBD (local scheduled task or GitHub Actions).

## Section 4: CMS Sync (One-Way, Phase 1)

One-directional push: vault → Payload CMS.

### Components

- **Sync script** — `pnpm sync:cms` reads vault species pages, maps to Payload schema, upserts via REST API
- **Field mapping config** — JSON file defining vault frontmatter/sections → CMS fields (e.g., `## Identification` → `characteristics`)
- **Pre-sync validation** — runs `validate_draft` on each page before pushing. Pages with `error`-level issues are skipped and logged.
- **Diff mode** — only syncs pages changed since last sync (tracks timestamps). `--force` for full resync.
- **Dry run** — `pnpm sync:cms --dry-run` previews changes without touching CMS

### Not included (deferred to Approach 3)

- CMS → vault sync
- Conflict resolution
- Real-time sync

## Section 5: Claude Code Authoring UX

### MCP configuration

Add mushroom-oracle as MCP server in `.mcp.json`:

```json
{
  "mcpServers": {
    "mushroom-oracle": {
      "command": "pnpm",
      "args": ["serve"],
      "cwd": "C:\\Users\\Moonman\\mushroom-oracle"
    }
  }
}
```

### Slash commands (MushroomOracle repo)

| Command | Action |
|---------|--------|
| `/new-species <name>` | Check gaps → draft → validate → write to vault |
| `/audit <path>` | Single-page audit against evidence |
| `/audit-all` | Batch audit, output report |
| `/sync` | CMS sync with dry-run preview + confirmation |
| `/gaps` | Show top species/topics with evidence but no coverage |

### Optional hook

`PostToolUse` hook on `.md` file writes in the vault — reminds to run validation before committing.

## Future: Approach 3 — Bidirectional Sync Engine

Deferred to a future phase. Full implementation includes:

- **CMS → vault sync** — changes made in Payload CMS are pulled back into the vault as markdown
- **Conflict resolution** — when vault and CMS diverge, present diffs and let the user choose
- **Real-time sync** — webhook-driven updates in both directions (Payload webhook → vault writer, file watcher → CMS push)
- **MCP server as validation middleware** — every sync in either direction passes through audit tools
- **Version tracking** — content versioning with audit trail across both systems
- **Multi-author support** — handle concurrent edits from different CMS users and vault authors

This becomes relevant once generation and quality workflows are proven and content volume justifies the infrastructure investment.

## Implementation Order

1. Vault ingestion (markdown extractor + source_type tagging)
2. Generation tools (content_gap_analysis awareness, draft_species_page, validate_draft)
3. Authoring UX (MCP config, slash commands)
4. Quality auditing (batch audit, drift detection)
5. CMS sync (one-way push with validation)
