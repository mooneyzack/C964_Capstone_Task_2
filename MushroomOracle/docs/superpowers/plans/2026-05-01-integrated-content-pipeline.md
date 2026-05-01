# Integrated Content Pipeline Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Connect the MushroomOracle Obsidian vault with the mushroom-oracle MCP server so the MCP server can ingest vault content, generate new evidence-backed pages, audit existing content, and sync to ShroomSpy's Payload CMS.

**Architecture:** Add a markdown extractor to mushroom-oracle's ingestion pipeline with heading-aware chunking and vault source_type tagging. Extend existing MCP tools (content_gap_analysis, draft_species_page) with vault awareness. Add new tools (expand_section, validate_draft). Build slash commands in MushroomOracle for authoring UX. Add batch audit CLI and one-way CMS sync script.

**Tech Stack:** TypeScript, Vitest, better-sqlite3, OpenAI API, MCP SDK, Zod, gray-matter (frontmatter parsing)

---

## File Structure

### mushroom-oracle (MCP server repo)

| File | Responsibility |
|------|---------------|
| `src/ingest/extractors/markdown.ts` | Extract text + metadata from .md files with frontmatter parsing and heading-aware sectioning |
| `src/ingest/index.ts` (modify) | Register markdown extractor, handle `source_type` |
| `src/db/schema.ts` (modify) | Add `source_type` column to `sources` table |
| `src/db/store.ts` (modify) | Accept `source_type` in `insertSource`, add `getVaultSources()` |
| `src/types.ts` (modify) | Add `source_type` to `Source` interface and `InsertSourceParams` |
| `src/tools/contentGapAnalysis.ts` (modify) | Auto-read vault species from DB instead of requiring `existing_species` param |
| `src/tools/expandSection.ts` | New tool: expand a vault page section with additional evidence |
| `src/tools/validateDraft.ts` | New tool: validate a draft page against evidence base |
| `src/mcp/server.ts` (modify) | Register new tools (expand_section, validate_draft) |
| `bin/ingest-vault.ts` | CLI entry point for vault ingestion |
| `bin/audit-vault.ts` | CLI entry point for batch vault auditing |
| `bin/sync-cms.ts` | CLI entry point for one-way CMS sync |
| `src/sync/fieldMapping.ts` | Map vault frontmatter/sections to Payload CMS fields |
| `src/sync/payloadClient.ts` | Payload CMS REST API client for upsert operations |
| `data/field_mapping.json` | Config: vault sections → CMS schema mapping |
| `tests/ingest/extractors/markdown.test.ts` | Tests for markdown extractor |
| `tests/ingest/vault-ingestion.test.ts` | Integration tests for vault ingestion flow |
| `tests/tools/expandSection.test.ts` | Tests for expand_section tool |
| `tests/tools/validateDraft.test.ts` | Tests for validate_draft tool |
| `tests/sync/fieldMapping.test.ts` | Tests for field mapping logic |

### MushroomOracle (Obsidian vault repo)

| File | Responsibility |
|------|---------------|
| `.mcp.json` | MCP server configuration pointing to mushroom-oracle |
| `.claude/commands/new-species.md` | Slash command: generate new species page |
| `.claude/commands/audit.md` | Slash command: audit a single page |
| `.claude/commands/audit-all.md` | Slash command: batch audit all pages |
| `.claude/commands/sync.md` | Slash command: CMS sync with dry-run |
| `.claude/commands/gaps.md` | Slash command: show content gaps |

---

## Task 1: Add `source_type` Column to Database Schema

**Files:**
- Modify: `C:\Users\Moonman\mushroom-oracle\src\db\schema.ts`
- Modify: `C:\Users\Moonman\mushroom-oracle\src\db\store.ts`
- Modify: `C:\Users\Moonman\mushroom-oracle\src\types.ts`

- [ ] **Step 1: Add `source_type` to the `Source` interface and `InsertSourceParams`**

In `src/types.ts`, add `source_type` field:

```typescript
/** Source record in the DB */
export interface Source {
  id: string
  file_path: string
  title: string | null
  authors: string | null
  year: number | null
  publisher_or_journal: string | null
  ingested_at: string
  chunk_count: number
  source_type: 'research' | 'vault'
}
```

In `src/db/store.ts`, add to `InsertSourceParams`:

```typescript
export interface InsertSourceParams {
  id: string
  file_path: string
  title?: string | null
  authors?: string | null
  year?: number | null
  publisher_or_journal?: string | null
  source_type?: 'research' | 'vault'
}
```

- [ ] **Step 2: Update schema to include `source_type` column**

In `src/db/schema.ts`, update the `sources` CREATE TABLE:

```typescript
db.exec(`
  CREATE TABLE IF NOT EXISTS sources (
    id TEXT PRIMARY KEY,
    file_path TEXT NOT NULL,
    title TEXT,
    authors TEXT,
    year INTEGER,
    publisher_or_journal TEXT,
    source_type TEXT NOT NULL DEFAULT 'research',
    ingested_at TEXT NOT NULL,
    chunk_count INTEGER DEFAULT 0
  );

  CREATE TABLE IF NOT EXISTS chunks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    source_id TEXT NOT NULL REFERENCES sources(id),
    chunk_index INTEGER NOT NULL,
    page_range TEXT,
    section TEXT,
    taxon_names TEXT,
    common_names TEXT,
    accepted_name TEXT,
    synonyms TEXT,
    family TEXT,
    topics TEXT,
    region TEXT,
    edibility_claims TEXT,
    toxicity_claims TEXT,
    lookalikes TEXT,
    diagnostic_features TEXT,
    cultivation_notes TEXT,
    text TEXT NOT NULL,
    UNIQUE(source_id, chunk_index)
  );
`)
```

Add a migration after the CREATE TABLE block to handle existing databases:

```typescript
const hasSourceType = db
  .prepare("SELECT 1 FROM pragma_table_info('sources') WHERE name = 'source_type'")
  .get()
if (!hasSourceType) {
  db.exec("ALTER TABLE sources ADD COLUMN source_type TEXT NOT NULL DEFAULT 'research'")
}
```

- [ ] **Step 3: Update `insertSource` to accept and store `source_type`**

In `src/db/store.ts`:

```typescript
export function insertSource(db: Database.Database, params: InsertSourceParams): void {
  db.prepare(
    `INSERT OR REPLACE INTO sources (id, file_path, title, authors, year, publisher_or_journal, source_type, ingested_at, chunk_count)
     VALUES (?, ?, ?, ?, ?, ?, ?, datetime('now'), 0)`,
  ).run(
    params.id,
    params.file_path,
    params.title ?? null,
    params.authors ?? null,
    params.year ?? null,
    params.publisher_or_journal ?? null,
    params.source_type ?? 'research',
  )
}
```

- [ ] **Step 4: Add `getVaultSources` helper**

In `src/db/store.ts`:

```typescript
export function getVaultSources(db: Database.Database): Source[] {
  return db
    .prepare("SELECT * FROM sources WHERE source_type = 'vault' ORDER BY title")
    .all() as Source[]
}

export function getVaultSpeciesNames(db: Database.Database): string[] {
  const rows = db
    .prepare(
      `SELECT DISTINCT accepted_name FROM chunks c
       JOIN sources s ON c.source_id = s.id
       WHERE s.source_type = 'vault' AND c.accepted_name IS NOT NULL`,
    )
    .all() as Array<{ accepted_name: string }>
  return rows.map((r) => r.accepted_name)
}
```

- [ ] **Step 5: Run tests to verify nothing is broken**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm test`
Expected: All existing tests pass (schema changes are backwards-compatible via DEFAULT and ALTER migration).

- [ ] **Step 6: Commit**

```bash
cd C:\Users\Moonman\mushroom-oracle
git add src/db/schema.ts src/db/store.ts src/types.ts
git commit -m "feat: add source_type column to sources table for vault/research distinction"
```

---

## Task 2: Markdown Extractor

**Files:**
- Create: `C:\Users\Moonman\mushroom-oracle\src\ingest\extractors\markdown.ts`
- Create: `C:\Users\Moonman\mushroom-oracle\tests\ingest\extractors\markdown.test.ts`

- [ ] **Step 1: Install gray-matter for frontmatter parsing**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm add gray-matter && pnpm add -D @types/gray-matter`

- [ ] **Step 2: Write the failing test**

Create `tests/ingest/extractors/markdown.test.ts`:

```typescript
import { describe, it, expect } from 'vitest'
import { extractMarkdown } from '../../../src/ingest/extractors/markdown.js'

const SAMPLE_MD = `---
common_name: Lion's Mane
scientific_name: Hericium erinaceus
phylum: Basidiomycota
class: Agaricomycetes
order: Russulales
family: Hericiaceae
tags:
  - gourmet
  - medicinal
edibility: edible
habitat: On dead or dying hardwood trees
distribution: North America, Europe, Asia
last_verified: 2026-05-01
---

## Identification

Forms a single clump of cascading white spines.

## Biology

White-rot saprotroph. Causes white heart rot in host trees.

## Cultivation

Grows on hardwood sawdust blocks. Fruiting at 15-22°C.
`

describe('extractMarkdown', () => {
  it('extracts frontmatter as metadata', () => {
    const result = extractMarkdown(SAMPLE_MD, 'Hericium erinaceus.md')
    expect(result.metadata.title).toBe('Hericium erinaceus')
    expect(result.metadata.authors).toBeUndefined()
  })

  it('splits pages by ## headings', () => {
    const result = extractMarkdown(SAMPLE_MD, 'Hericium erinaceus.md')
    expect(result.pages.length).toBe(3)
    expect(result.pages[0].section).toBe('Identification')
    expect(result.pages[1].section).toBe('Biology')
    expect(result.pages[2].section).toBe('Cultivation')
  })

  it('includes section text without the heading itself', () => {
    const result = extractMarkdown(SAMPLE_MD, 'Hericium erinaceus.md')
    expect(result.pages[0].text).toContain('cascading white spines')
    expect(result.pages[0].text).not.toContain('## Identification')
  })

  it('extracts frontmatter fields into metadata', () => {
    const result = extractMarkdown(SAMPLE_MD, 'Hericium erinaceus.md')
    expect(result.frontmatter.scientific_name).toBe('Hericium erinaceus')
    expect(result.frontmatter.common_name).toBe("Lion's Mane")
    expect(result.frontmatter.family).toBe('Hericiaceae')
    expect(result.frontmatter.tags).toContain('medicinal')
  })

  it('uses scientific_name as title, falls back to filename', () => {
    const noName = `---\ntags: []\n---\n\n## Section\n\nContent here.`
    const result = extractMarkdown(noName, 'Some File.md')
    expect(result.metadata.title).toBe('Some File')
  })
})
```

- [ ] **Step 3: Run test to verify it fails**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm vitest run tests/ingest/extractors/markdown.test.ts`
Expected: FAIL — module not found.

- [ ] **Step 4: Implement the markdown extractor**

Create `src/ingest/extractors/markdown.ts`:

```typescript
import matter from 'gray-matter'
import { basename } from 'node:path'
import type { ExtractedDoc, ExtractedPage } from '../../types.js'

export interface MarkdownFrontmatter {
  scientific_name?: string
  common_name?: string
  phylum?: string
  class?: string
  order?: string
  family?: string
  tags?: string[]
  edibility?: string
  habitat?: string
  distribution?: string
  [key: string]: unknown
}

export interface ExtractedMarkdown extends ExtractedDoc {
  frontmatter: MarkdownFrontmatter
}

export function extractMarkdown(content: string, fileName: string): ExtractedMarkdown {
  const { data, content: body } = matter(content)
  const frontmatter = data as MarkdownFrontmatter

  const title =
    frontmatter.scientific_name || basename(fileName, '.md')

  const sections = splitBySections(body)

  const pages: ExtractedPage[] = sections.map((s) => ({
    text: s.text.trim(),
    section: s.heading,
  }))

  return {
    pages: pages.filter((p) => p.text.length > 0),
    metadata: {
      title,
      authors: undefined,
      year: undefined,
      publisher_or_journal: undefined,
    },
    frontmatter,
  }
}

interface Section {
  heading: string | undefined
  text: string
}

function splitBySections(body: string): Section[] {
  const lines = body.split('\n')
  const sections: Section[] = []
  let currentHeading: string | undefined = undefined
  let currentLines: string[] = []

  for (const line of lines) {
    const headingMatch = line.match(/^##\s+(.+)$/)
    if (headingMatch) {
      if (currentLines.length > 0 || currentHeading !== undefined) {
        sections.push({ heading: currentHeading, text: currentLines.join('\n') })
      }
      currentHeading = headingMatch[1].trim()
      currentLines = []
    } else {
      currentLines.push(line)
    }
  }

  if (currentLines.length > 0 || currentHeading !== undefined) {
    sections.push({ heading: currentHeading, text: currentLines.join('\n') })
  }

  return sections
}
```

- [ ] **Step 5: Run test to verify it passes**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm vitest run tests/ingest/extractors/markdown.test.ts`
Expected: All 5 tests PASS.

- [ ] **Step 6: Commit**

```bash
cd C:\Users\Moonman\mushroom-oracle
git add src/ingest/extractors/markdown.ts tests/ingest/extractors/markdown.test.ts package.json pnpm-lock.yaml
git commit -m "feat: add markdown extractor with heading-aware sectioning and frontmatter parsing"
```

---

## Task 3: Vault Ingestion Orchestrator

**Files:**
- Modify: `C:\Users\Moonman\mushroom-oracle\src\ingest\index.ts`
- Create: `C:\Users\Moonman\mushroom-oracle\bin\ingest-vault.ts`
- Create: `C:\Users\Moonman\mushroom-oracle\tests\ingest\vault-ingestion.test.ts`

- [ ] **Step 1: Write the failing test for vault-specific discovery and ingestion**

Create `tests/ingest/vault-ingestion.test.ts`:

```typescript
import { describe, it, expect } from 'vitest'
import { discoverMarkdownFiles, routeExtractor } from '../../src/ingest/index.js'

describe('discoverMarkdownFiles', () => {
  it('finds .md files recursively', () => {
    // Uses the MushroomOracle vault path for integration testing
    // This test verifies the discovery function works with a real directory
    const files = discoverMarkdownFiles('C:\\Users\\Moonman\\MushroomOracle\\species')
    expect(files.length).toBeGreaterThan(0)
    expect(files.every((f) => f.endsWith('.md'))).toBe(true)
  })
})

describe('routeExtractor with markdown', () => {
  it('returns markdown for .md files', () => {
    expect(routeExtractor('species/Hericium erinaceus.md')).toBe('markdown')
  })

  it('still returns pdf for .pdf files', () => {
    expect(routeExtractor('research/paper.pdf')).toBe('pdf')
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm vitest run tests/ingest/vault-ingestion.test.ts`
Expected: FAIL — `discoverMarkdownFiles` not exported, `routeExtractor` doesn't handle `.md`.

- [ ] **Step 3: Update `src/ingest/index.ts` to support markdown files**

Add markdown discovery function and update `routeExtractor`:

```typescript
import { extractMarkdown } from './extractors/markdown.js'
import { readFileSync } from 'node:fs'

/** Recursively discover .md files in a directory, sorted alphabetically */
export function discoverMarkdownFiles(dirPath: string): string[] {
  const results: string[] = []
  walkSyncMd(dirPath, results)
  return results.sort()
}

function walkSyncMd(dir: string, results: string[]): void {
  const entries = readdirSync(dir, { withFileTypes: true }) as Array<{
    name: string
    isDirectory: () => boolean
    isFile: () => boolean
  }>

  for (const entry of entries) {
    const fullPath = resolve(dir, entry.name)
    if (entry.isDirectory()) {
      walkSyncMd(fullPath, results)
    } else if (entry.isFile()) {
      const ext = extname(entry.name).toLowerCase()
      if (ext === '.md') {
        results.push(fullPath)
      }
    }
  }
}
```

Update `routeExtractor`:

```typescript
export function routeExtractor(filePath: string): 'pdf' | 'epub' | 'docx' | 'markdown' | null {
  const ext = extname(filePath).toLowerCase()
  switch (ext) {
    case '.pdf':
      return 'pdf'
    case '.epub':
      return 'epub'
    case '.docx':
      return 'docx'
    case '.md':
      return 'markdown'
    default:
      return null
  }
}
```

- [ ] **Step 4: Add `ingestVault` function to the ingestion orchestrator**

Add to `src/ingest/index.ts`:

```typescript
export interface VaultIngestOptions {
  db: Database.Database
  embedder: Embedder
  enricher: Enricher
  config: OracleConfig
  force?: boolean
}

export async function ingestVault(
  vaultPath: string,
  options: VaultIngestOptions,
): Promise<IngestResult> {
  const { db, embedder, enricher, config, force = false } = options

  const result: IngestResult = {
    processed: 0,
    skipped: 0,
    failed: [],
    skippedScanned: [],
  }

  const files = discoverMarkdownFiles(vaultPath)
  console.log(`[ORACLE] Discovered ${files.length} markdown files in vault`)

  for (let i = 0; i < files.length; i++) {
    const filePath = files[i]
    const fileName = basename(filePath)
    const progress = `[${i + 1}/${files.length}]`

    try {
      const hash = await computeFileHash(filePath)

      if (!force && sourceExists(db, hash)) {
        console.log(`[ORACLE] ${progress} Skipping "${fileName}" (already ingested)`)
        result.skipped++
        continue
      }

      if (sourceExists(db, hash)) {
        deleteSourceAndChunks(db, hash)
      }

      const content = readFileSync(filePath, 'utf-8')
      const extracted = extractMarkdown(content, fileName)

      if (extracted.pages.length === 0) {
        console.log(`[ORACLE] ${progress} Skipping "${fileName}" (no content)`)
        result.skipped++
        continue
      }

      const chunks = chunkText(extracted.pages, {
        chunkSize: config.chunkSize * 4,
        chunkOverlap: config.chunkOverlap * 4,
      })

      if (chunks.length === 0) {
        result.failed.push(fileName)
        continue
      }

      console.log(`[ORACLE] ${progress} Embedding ${chunks.length} chunks from "${fileName}"...`)
      const embeddings = await embedder.embedBatch(chunks.map((c) => c.text))

      const sourceTitle = extracted.metadata.title ?? fileName
      const metadatas = await enricher.enrichChunks(
        chunks.map((c) => ({ text: c.text, sourceTitle })),
      )

      const insertAll = db.transaction(() => {
        insertSource(db, {
          id: hash,
          file_path: filePath,
          title: sourceTitle,
          authors: null,
          year: null,
          publisher_or_journal: null,
          source_type: 'vault',
        })

        for (let j = 0; j < chunks.length; j++) {
          insertChunk(db, {
            sourceId: hash,
            chunkIndex: chunks[j].chunkIndex,
            pageRange: null,
            section: chunks[j].section ?? null,
            text: chunks[j].text,
            embedding: embeddings[j],
            metadata: metadatas[j],
          })
        }
      })

      insertAll()
      console.log(`[ORACLE] ${progress} Ingested "${fileName}" — ${chunks.length} chunks`)
      result.processed++
    } catch (error) {
      const message = error instanceof Error ? error.message : String(error)
      console.error(`[ORACLE] ${progress} Failed "${fileName}": ${message}`)
      result.failed.push(fileName)
    }
  }

  console.log(
    `[ORACLE] Vault ingestion done. Processed: ${result.processed}, Skipped: ${result.skipped}, Failed: ${result.failed.length}`,
  )
  return result
}
```

- [ ] **Step 5: Create the `bin/ingest-vault.ts` CLI entry point**

Create `bin/ingest-vault.ts`:

```typescript
#!/usr/bin/env tsx
import { resolve } from 'node:path'
import OpenAI from 'openai'
import { loadConfig } from '../src/config.js'
import { initDb } from '../src/db/schema.js'
import { getStats } from '../src/db/store.js'
import { createEmbedder } from '../src/ingest/embedder.js'
import { createEnricher } from '../src/ingest/enricher.js'
import { ingestVault } from '../src/ingest/index.js'
import { buildTaxonReference } from '../src/ingest/taxonReference.js'

const args = process.argv.slice(2)
let vaultPath: string | undefined
let force = false

for (const arg of args) {
  if (arg === '--force') {
    force = true
  } else if (!arg.startsWith('--')) {
    vaultPath = arg
  }
}

if (!vaultPath) {
  console.error('Usage: pnpm ingest:vault <vault-directory> [--force]')
  process.exit(1)
}

const config = loadConfig()

if (!config.openaiApiKey) {
  console.error('[ORACLE] Error: OPENAI_API_KEY is not set.')
  process.exit(1)
}

const resolvedPath = resolve(vaultPath)
console.log(`[ORACLE] Ingesting vault from: ${resolvedPath}`)

const openaiClient = new OpenAI({ apiKey: config.openaiApiKey })
const db = initDb(config.dbPath)
const embedder = createEmbedder({ openaiClient, model: config.embeddingModel })
const enricher = createEnricher({ openaiClient, model: config.enrichmentModel })

try {
  const result = await ingestVault(resolvedPath, { db, embedder, enricher, config, force })

  console.log('\n[ORACLE] === Vault Ingestion Summary ===')
  console.log(`  Processed: ${result.processed}`)
  console.log(`  Skipped:   ${result.skipped}`)
  console.log(`  Failed:    ${result.failed.length}`)
  if (result.failed.length > 0) {
    console.log(`  Failed files: ${result.failed.join(', ')}`)
  }

  console.log('\n[ORACLE] Rebuilding taxon reference...')
  const taxonPath = resolve(config.projectRoot, 'data', 'taxon_reference.json')
  const taxonRef = buildTaxonReference(db, taxonPath)
  console.log(`[ORACLE] Taxon reference: ${taxonRef.size} species`)

  const stats = getStats(db)
  console.log(`\n[ORACLE] === Database Stats ===`)
  console.log(`  Sources: ${stats.sourceCount}`)
  console.log(`  Chunks:  ${stats.chunkCount}`)
} finally {
  db.close()
}
```

- [ ] **Step 6: Add `ingest:vault` script to package.json**

Add to `scripts` in `package.json`:

```json
"ingest:vault": "tsx bin/ingest-vault.ts"
```

- [ ] **Step 7: Run tests to verify**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm vitest run tests/ingest/vault-ingestion.test.ts`
Expected: PASS.

- [ ] **Step 8: Commit**

```bash
cd C:\Users\Moonman\mushroom-oracle
git add src/ingest/index.ts bin/ingest-vault.ts tests/ingest/vault-ingestion.test.ts package.json
git commit -m "feat: vault ingestion pipeline with markdown discovery and source_type tagging"
```

---

## Task 4: `expand_section` Tool

**Files:**
- Create: `C:\Users\Moonman\mushroom-oracle\src\tools\expandSection.ts`
- Create: `C:\Users\Moonman\mushroom-oracle\tests\tools\expandSection.test.ts`
- Modify: `C:\Users\Moonman\mushroom-oracle\src\types.ts`

- [ ] **Step 1: Add types for expand_section**

Add to `src/types.ts`:

```typescript
// ── Expand Section ──────────────────────────────────────────
export interface ExpandSectionParams {
  species_name: string
  section_heading: string
  current_text: string
  embedFn: (text: string) => Promise<Float32Array>
  chatFn: (params: any) => Promise<any>
  topK?: number
}

export interface ExpandSectionResult {
  expanded_text: string
  new_claims: Array<{
    claim: string
    source: string
    authors: string[]
    page: string | null
  }>
  sources: SupportingPassage[]
}
```

- [ ] **Step 2: Write the failing test**

Create `tests/tools/expandSection.test.ts`:

```typescript
import { describe, it, expect, vi } from 'vitest'
import { expandSection } from '../../src/tools/expandSection.js'
import Database from 'better-sqlite3'
import { initDb } from '../../src/db/schema.js'
import { insertSource, insertChunk } from '../../src/db/store.js'
import type { EnrichedMetadata } from '../../src/types.js'

function makeMetadata(overrides: Partial<EnrichedMetadata> = {}): EnrichedMetadata {
  return {
    taxon_names: ['Hericium erinaceus'],
    common_names: ['Lion\'s Mane'],
    accepted_name: 'Hericium erinaceus',
    synonyms: [],
    family: 'Hericiaceae',
    topics: ['medicinal'],
    region: null,
    edibility_claims: [],
    toxicity_claims: [],
    lookalikes: [],
    diagnostic_features: [],
    cultivation_notes: [],
    ...overrides,
  }
}

describe('expandSection', () => {
  it('returns expanded text with source citations', async () => {
    const db = initDb(':memory:')

    insertSource(db, {
      id: 'src1',
      file_path: '/test/paper.pdf',
      title: 'NGF Research Paper',
      authors: JSON.stringify(['Smith J']),
      year: 2020,
      source_type: 'research',
    })

    const embedding = new Float32Array(1536).fill(0.1)
    insertChunk(db, {
      sourceId: 'src1',
      chunkIndex: 0,
      pageRange: 'p. 42',
      section: null,
      text: 'Hericium erinaceus erinacines stimulate NGF synthesis in astrocytes at concentrations of 10μM.',
      embedding,
      metadata: makeMetadata({ topics: ['medicinal', 'neuroscience'] }),
    })

    const mockEmbedFn = vi.fn().mockResolvedValue(new Float32Array(1536).fill(0.1))
    const mockChatFn = vi.fn().mockResolvedValue({
      choices: [{
        message: {
          content: JSON.stringify({
            expanded_text: 'Erinacines stimulate NGF synthesis in astrocytes. Studies show concentrations of 10μM are effective [1].',
            new_claims: [{
              claim: 'Erinacines stimulate NGF at 10μM',
              source: 'NGF Research Paper',
              authors: ['Smith J'],
              page: 'p. 42',
            }],
          }),
        },
      }],
    })

    const result = await expandSection(db, {
      species_name: 'Hericium erinaceus',
      section_heading: 'Bioactive Compounds',
      current_text: 'Contains hericenones and erinacines.',
      embedFn: mockEmbedFn,
      chatFn: mockChatFn,
    })

    expect(result.expanded_text).toContain('NGF')
    expect(result.new_claims.length).toBeGreaterThan(0)
    expect(mockChatFn).toHaveBeenCalledOnce()
  })
})
```

- [ ] **Step 3: Run test to verify it fails**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm vitest run tests/tools/expandSection.test.ts`
Expected: FAIL — module not found.

- [ ] **Step 4: Implement `expandSection`**

Create `src/tools/expandSection.ts`:

```typescript
import type Database from 'better-sqlite3'
import type { ExpandSectionParams, ExpandSectionResult, SupportingPassage } from '../types.js'
import { searchEvidence } from './searchEvidence.js'

const EXPAND_PROMPT = `You are a mycology expert expanding a section of a species page with additional evidence.

Given the current section text and new evidence passages, write an expanded version that:
1. Preserves all existing accurate content
2. Adds new claims supported by the evidence passages
3. Cites passage numbers [1], [2], etc.
4. Maintains the same writing style and depth

Return ONLY valid JSON:
{
  "expanded_text": "The expanded section text with [1] citations",
  "new_claims": [
    {
      "claim": "The specific new claim added",
      "source": "Source title",
      "authors": ["Author names"],
      "page": "p. X or null"
    }
  ]
}`

export async function expandSection(
  db: Database.Database,
  params: ExpandSectionParams,
): Promise<ExpandSectionResult> {
  const { species_name, section_heading, current_text, embedFn, chatFn, topK = 10 } = params

  const query = `${species_name} ${section_heading} ${current_text.slice(0, 200)}`

  const evidence = await searchEvidence(db, {
    query,
    embedFn,
    topK,
    taxonFilter: species_name,
  })

  if (evidence.chunks.length === 0) {
    return {
      expanded_text: current_text,
      new_claims: [],
      sources: [],
    }
  }

  const passageContext = evidence.chunks
    .map((c, i) => `[${i + 1}] (${c.source.title ?? 'Unknown'}, ${c.source.page ?? '?'})\n${c.text}`)
    .join('\n\n')

  const response = await chatFn({
    model: 'gpt-4o-mini',
    messages: [
      { role: 'system', content: EXPAND_PROMPT },
      {
        role: 'user',
        content: `Species: ${species_name}\nSection: ${section_heading}\n\nCurrent text:\n${current_text}\n\nNew evidence passages:\n\n${passageContext}`,
      },
    ],
    temperature: 0.1,
    response_format: { type: 'json_object' },
  })

  const content = response.choices?.[0]?.message?.content ?? '{}'
  let parsed: any
  try {
    parsed = JSON.parse(content)
  } catch {
    parsed = { expanded_text: current_text, new_claims: [] }
  }

  const sources: SupportingPassage[] = evidence.chunks.map((c) => ({
    text: c.text,
    source: c.source.title ?? 'Unknown',
    authors: c.source.authors,
    page: c.source.page,
  }))

  return {
    expanded_text: parsed.expanded_text ?? current_text,
    new_claims: parsed.new_claims ?? [],
    sources,
  }
}
```

- [ ] **Step 5: Run test to verify it passes**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm vitest run tests/tools/expandSection.test.ts`
Expected: PASS.

- [ ] **Step 6: Commit**

```bash
cd C:\Users\Moonman\mushroom-oracle
git add src/tools/expandSection.ts tests/tools/expandSection.test.ts src/types.ts
git commit -m "feat: add expand_section tool for enriching vault page sections with evidence"
```

---

## Task 5: `validate_draft` Tool

**Files:**
- Create: `C:\Users\Moonman\mushroom-oracle\src\tools\validateDraft.ts`
- Create: `C:\Users\Moonman\mushroom-oracle\tests\tools\validateDraft.test.ts`
- Modify: `C:\Users\Moonman\mushroom-oracle\src\types.ts`

- [ ] **Step 1: Add types for validate_draft**

Add to `src/types.ts`:

```typescript
// ── Validate Draft ──────────────────────────────────────────
export interface ValidateDraftParams {
  species_name: string
  draft_text: string
  embedFn: (text: string) => Promise<Float32Array>
  chatFn: (params: any) => Promise<any>
  topK?: number
}

export interface ValidationIssue {
  severity: 'error' | 'warning' | 'suggestion'
  type: 'unsupported_claim' | 'contradiction' | 'missing_safety' | 'missing_context'
  claim: string
  detail: string
  sources: Array<{ title: string; authors: string[]; page: string | null }>
}

export interface ValidateDraftResult {
  species_name: string
  issues: ValidationIssue[]
  summary: {
    total_claims_checked: number
    errors: number
    warnings: number
    suggestions: number
  }
  verdict: 'pass' | 'needs_revision' | 'major_issues'
}
```

- [ ] **Step 2: Write the failing test**

Create `tests/tools/validateDraft.test.ts`:

```typescript
import { describe, it, expect, vi } from 'vitest'
import { validateDraft } from '../../src/tools/validateDraft.js'
import { initDb } from '../../src/db/schema.js'
import { insertSource, insertChunk } from '../../src/db/store.js'
import type { EnrichedMetadata } from '../../src/types.js'

function makeMetadata(overrides: Partial<EnrichedMetadata> = {}): EnrichedMetadata {
  return {
    taxon_names: ['Hericium erinaceus'],
    common_names: ['Lion\'s Mane'],
    accepted_name: 'Hericium erinaceus',
    synonyms: [],
    family: 'Hericiaceae',
    topics: ['medicinal'],
    region: null,
    edibility_claims: [],
    toxicity_claims: [],
    lookalikes: [],
    diagnostic_features: [],
    cultivation_notes: [],
    ...overrides,
  }
}

describe('validateDraft', () => {
  it('returns issues for unsupported claims', async () => {
    const db = initDb(':memory:')

    insertSource(db, {
      id: 'src1',
      file_path: '/test/paper.pdf',
      title: 'Lion\'s Mane Review',
      authors: JSON.stringify(['Jones A']),
      year: 2021,
      source_type: 'research',
    })

    const embedding = new Float32Array(1536).fill(0.1)
    insertChunk(db, {
      sourceId: 'src1',
      chunkIndex: 0,
      pageRange: 'p. 15',
      section: null,
      text: 'Hericium erinaceus shows promise for cognitive support but clinical evidence is limited to small trials.',
      embedding,
      metadata: makeMetadata(),
    })

    const mockEmbedFn = vi.fn().mockResolvedValue(new Float32Array(1536).fill(0.1))
    const mockChatFn = vi.fn().mockResolvedValue({
      choices: [{
        message: {
          content: JSON.stringify({
            issues: [{
              severity: 'warning',
              type: 'unsupported_claim',
              claim: 'Lion\'s Mane cures Alzheimer\'s disease',
              detail: 'Evidence supports cognitive support but not a cure for Alzheimer\'s',
              sources: [{ title: 'Lion\'s Mane Review', authors: ['Jones A'], page: 'p. 15' }],
            }],
            total_claims_checked: 3,
          }),
        },
      }],
    })

    const result = await validateDraft(db, {
      species_name: 'Hericium erinaceus',
      draft_text: 'Lion\'s Mane cures Alzheimer\'s disease and regenerates brain cells completely.',
      embedFn: mockEmbedFn,
      chatFn: mockChatFn,
    })

    expect(result.issues.length).toBeGreaterThan(0)
    expect(result.issues[0].severity).toBe('warning')
    expect(result.verdict).not.toBe('pass')
  })

  it('returns pass verdict when no issues found', async () => {
    const db = initDb(':memory:')

    const mockEmbedFn = vi.fn().mockResolvedValue(new Float32Array(1536).fill(0.1))
    const mockChatFn = vi.fn().mockResolvedValue({
      choices: [{
        message: {
          content: JSON.stringify({
            issues: [],
            total_claims_checked: 2,
          }),
        },
      }],
    })

    const result = await validateDraft(db, {
      species_name: 'Hericium erinaceus',
      draft_text: 'Lion\'s Mane is an edible fungus in the family Hericiaceae.',
      embedFn: mockEmbedFn,
      chatFn: mockChatFn,
    })

    expect(result.issues).toHaveLength(0)
    expect(result.verdict).toBe('pass')
  })
})
```

- [ ] **Step 3: Run test to verify it fails**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm vitest run tests/tools/validateDraft.test.ts`
Expected: FAIL �� module not found.

- [ ] **Step 4: Implement `validateDraft`**

Create `src/tools/validateDraft.ts`:

```typescript
import type Database from 'better-sqlite3'
import type {
  ValidateDraftParams,
  ValidateDraftResult,
  ValidationIssue,
  SupportingPassage,
} from '../types.js'
import { searchEvidence } from './searchEvidence.js'

const VALIDATE_PROMPT = `You are a rigorous mycology fact-checker. Given a draft species page and evidence passages from published research, identify any issues.

Check for:
1. Claims not supported by the evidence (unsupported_claim)
2. Claims that contradict the evidence (contradiction)
3. Missing safety information — dangerous look-alikes not mentioned (missing_safety)
4. Important context that is missing and could mislead (missing_context)

Return ONLY valid JSON:
{
  "issues": [
    {
      "severity": "error | warning | suggestion",
      "type": "unsupported_claim | contradiction | missing_safety | missing_context",
      "claim": "The specific claim in the draft",
      "detail": "What the evidence actually says",
      "sources": [{"title": "Source title", "authors": ["Author"], "page": "p. X or null"}]
    }
  ],
  "total_claims_checked": <number>
}

Severity guide:
- error: factual inaccuracy, dangerous omission (missing toxic look-alike)
- warning: unsupported or overstated claim
- suggestion: could be improved with additional context

If no issues are found, return {"issues": [], "total_claims_checked": <number>}.`

export async function validateDraft(
  db: Database.Database,
  params: ValidateDraftParams,
): Promise<ValidateDraftResult> {
  const { species_name, draft_text, embedFn, chatFn, topK = 15 } = params

  const evidence = await searchEvidence(db, {
    query: `${species_name} ${draft_text.slice(0, 300)}`,
    embedFn,
    topK,
    taxonFilter: species_name,
  })

  const passageContext =
    evidence.chunks.length > 0
      ? evidence.chunks
          .map((c, i) => `[${i + 1}] (${c.source.title ?? 'Unknown'}, ${c.source.page ?? '?'})\n${c.text}`)
          .join('\n\n')
      : 'No evidence found for this species.'

  const response = await chatFn({
    model: 'gpt-4o-mini',
    messages: [
      { role: 'system', content: VALIDATE_PROMPT },
      {
        role: 'user',
        content: `Species: ${species_name}\n\nDraft text:\n${draft_text}\n\nEvidence passages:\n\n${passageContext}`,
      },
    ],
    temperature: 0.0,
    response_format: { type: 'json_object' },
  })

  const content = response.choices?.[0]?.message?.content ?? '{}'
  let parsed: any
  try {
    parsed = JSON.parse(content)
  } catch {
    parsed = { issues: [], total_claims_checked: 0 }
  }

  const issues: ValidationIssue[] = (parsed.issues ?? []).map((issue: any) => ({
    severity: issue.severity ?? 'warning',
    type: issue.type ?? 'unsupported_claim',
    claim: issue.claim ?? '',
    detail: issue.detail ?? '',
    sources: issue.sources ?? [],
  }))

  const errors = issues.filter((i) => i.severity === 'error').length
  const warnings = issues.filter((i) => i.severity === 'warning').length
  const suggestions = issues.filter((i) => i.severity === 'suggestion').length

  let verdict: 'pass' | 'needs_revision' | 'major_issues'
  if (errors > 0) {
    verdict = 'major_issues'
  } else if (warnings > 0) {
    verdict = 'needs_revision'
  } else {
    verdict = 'pass'
  }

  return {
    species_name,
    issues,
    summary: {
      total_claims_checked: parsed.total_claims_checked ?? 0,
      errors,
      warnings,
      suggestions,
    },
    verdict,
  }
}
```

- [ ] **Step 5: Run test to verify it passes**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm vitest run tests/tools/validateDraft.test.ts`
Expected: PASS.

- [ ] **Step 6: Commit**

```bash
cd C:\Users\Moonman\mushroom-oracle
git add src/tools/validateDraft.ts tests/tools/validateDraft.test.ts src/types.ts
git commit -m "feat: add validate_draft tool for checking vault pages against evidence"
```

---

## Task 6: Register New Tools in MCP Server

**Files:**
- Modify: `C:\Users\Moonman\mushroom-oracle\src\mcp\server.ts`

- [ ] **Step 1: Import the new tools**

Add imports at top of `src/mcp/server.ts`:

```typescript
import { expandSection } from '../tools/expandSection.js'
import { validateDraft } from '../tools/validateDraft.js'
import { getVaultSpeciesNames } from '../db/store.js'
```

- [ ] **Step 2: Register `expand_section` tool**

Add after the existing tool registrations:

```typescript
server.tool(
  'expand_section',
  'Expand a section of a vault species page with additional evidence from the research base. Returns enriched text with source citations.',
  {
    species_name: z.string().describe('Scientific name of the species'),
    section_heading: z.string().describe('The heading of the section to expand (e.g., "Bioactive Compounds")'),
    current_text: z.string().describe('The current text content of the section'),
  },
  async ({ species_name, section_heading, current_text }) => {
    const result = await expandSection(db, {
      species_name,
      section_heading,
      current_text,
      embedFn,
      chatFn,
    })
    return { content: [{ type: 'text' as const, text: JSON.stringify(result, null, 2) }] }
  },
)
```

- [ ] **Step 3: Register `validate_draft` tool**

```typescript
server.tool(
  'validate_draft',
  'Validate a draft species page against the research evidence base. Flags unsupported claims, contradictions, missing safety information, and missing context.',
  {
    species_name: z.string().describe('Scientific name of the species'),
    draft_text: z.string().describe('The full markdown text of the draft page'),
  },
  async ({ species_name, draft_text }) => {
    const result = await validateDraft(db, {
      species_name,
      draft_text,
      embedFn,
      chatFn,
    })
    return { content: [{ type: 'text' as const, text: JSON.stringify(result, null, 2) }] }
  },
)
```

- [ ] **Step 4: Update `content_gap_analysis` to auto-read vault species**

Modify the existing `content_gap_analysis` tool registration to make `existing_species` optional and auto-populate from vault:

```typescript
server.tool(
  'content_gap_analysis',
  'Compare Oracle species coverage against existing content (vault + CMS) to find high-value species pages to create. If no existing_species provided, reads from ingested vault content.',
  {
    existing_species: z.array(z.string()).optional().describe('Species names currently covered. If omitted, reads from vault.'),
  },
  async ({ existing_species }) => {
    const species = existing_species ?? getVaultSpeciesNames(db)
    const result = contentGapAnalysis(db, { existing_species: species }, taxonReference)
    return { content: [{ type: 'text' as const, text: JSON.stringify(result, null, 2) }] }
  },
)
```

- [ ] **Step 5: Run tests**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm test`
Expected: All tests pass.

- [ ] **Step 6: Commit**

```bash
cd C:\Users\Moonman\mushroom-oracle
git add src/mcp/server.ts
git commit -m "feat: register expand_section and validate_draft tools, auto-populate vault species in gap analysis"
```

---

## Task 7: MCP Configuration in MushroomOracle

**Files:**
- Create: `C:\Users\Moonman\MushroomOracle\.mcp.json`

- [ ] **Step 1: Create MCP config**

Create `C:\Users\Moonman\MushroomOracle\.mcp.json`:

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

- [ ] **Step 2: Commit**

```bash
cd C:\Users\Moonman\MushroomOracle
git add .mcp.json
git commit -m "feat: add MCP server configuration for mushroom-oracle integration"
```

---

## Task 8: Slash Commands for Authoring UX

**Files:**
- Create: `C:\Users\Moonman\MushroomOracle\.claude\commands\new-species.md`
- Create: `C:\Users\Moonman\MushroomOracle\.claude\commands\audit.md`
- Create: `C:\Users\Moonman\MushroomOracle\.claude\commands\audit-all.md`
- Create: `C:\Users\Moonman\MushroomOracle\.claude\commands\sync.md`
- Create: `C:\Users\Moonman\MushroomOracle\.claude\commands\gaps.md`

- [ ] **Step 1: Create `/new-species` command**

Create `.claude/commands/new-species.md`:

```markdown
---
description: Generate a new evidence-backed species page for the vault
arguments:
  - name: species
    description: Scientific name of the species (e.g., "Trametes versicolor")
    required: true
---

Generate a new species page for $ARGUMENTS.species:

1. First, call the `content_gap_analysis` MCP tool (mushroom-oracle server) to verify this species doesn't already have vault coverage.
2. If coverage exists, inform me and ask if I want to proceed anyway.
3. Call `draft_species_page` with species_name="$ARGUMENTS.species" to generate the content from research evidence.
4. Format the result as a markdown file matching this template structure:
   - YAML frontmatter with: common_name, scientific_name, phylum, class, order, family, tags, edibility, habitat, distribution, last_verified
   - Sections: ## Identification, ## Biology, ## Cultivation, ## Bioactive Compounds, ## Culinary, ## Sources
5. Write the file to `species/$ARGUMENTS.species.md`
6. Call `validate_draft` with the full text to check for issues.
7. Report any validation issues and ask if I want to fix them.
```

- [ ] **Step 2: Create `/audit` command**

Create `.claude/commands/audit.md`:

```markdown
---
description: Audit a vault species page against the research evidence base
arguments:
  - name: path
    description: Path to the species file (e.g., "species/Hericium erinaceus.md")
    required: true
---

Audit the species page at $ARGUMENTS.path:

1. Read the file at $ARGUMENTS.path
2. Extract the species_name from frontmatter (scientific_name field)
3. Call the `audit_species` MCP tool (mushroom-oracle server) with the page's structured data:
   - Map frontmatter fields to the tool's input schema
   - Extract look_alikes from the content if present
4. Call `validate_draft` with the full page text for claim-level checking
5. Present findings grouped by severity (errors first, then warnings, then suggestions)
6. For each error-level finding, suggest a specific fix
```

- [ ] **Step 3: Create `/audit-all` command**

Create `.claude/commands/audit-all.md`:

```markdown
---
description: Run batch audit across all vault species pages
---

Run a batch audit of all species pages in the vault:

1. List all .md files in the `species/` directory
2. For each file, read its frontmatter to get the species_name
3. Call `validate_draft` for each page (use the mushroom-oracle MCP server)
4. Compile results into a report at `docs/audit-report-YYYY-MM-DD.md` with:
   - Summary: total pages checked, pages with errors, pages with warnings
   - Error-level findings (grouped by species)
   - Warning-level findings (grouped by species)
   - Suggestion-level findings (grouped by species)
5. Print a summary to the console
```

- [ ] **Step 4: Create `/sync` command**

Create `.claude/commands/sync.md`:

```markdown
---
description: Sync vault content to ShroomSpy Payload CMS (dry-run first)
---

Sync vault species pages to ShroomSpy's Payload CMS:

1. Run `pnpm sync:cms --dry-run` in the mushroom-oracle directory to preview changes
2. Show me what would be created/updated/skipped
3. Ask for confirmation before proceeding
4. If confirmed, run `pnpm sync:cms` to push changes
5. Report results: pages synced, pages skipped (validation errors), any failures
```

- [ ] **Step 5: Create `/gaps` command**

Create `.claude/commands/gaps.md`:

```markdown
---
description: Show species with strong research evidence but no vault coverage
---

Find content gaps — species with research evidence in the Oracle but no vault page:

1. Call `content_gap_analysis` MCP tool (mushroom-oracle server) without providing existing_species (it will auto-read from vault)
2. Present the top 10 gaps as a table:
   - Species name (scientific + common names)
   - Evidence strength (chunk count, source count)
   - Topics covered
   - Commercial potential
   - Recommendation
3. Ask which species I'd like to create a page for, then run /new-species for it
```

- [ ] **Step 6: Commit**

```bash
cd C:\Users\Moonman\MushroomOracle
git add .claude/commands/new-species.md .claude/commands/audit.md .claude/commands/audit-all.md .claude/commands/sync.md .claude/commands/gaps.md
git commit -m "feat: add slash commands for species generation, auditing, syncing, and gap analysis"
```

---

## Task 9: Batch Audit CLI

**Files:**
- Create: `C:\Users\Moonman\mushroom-oracle\bin\audit-vault.ts`

- [ ] **Step 1: Create the batch audit script**

Create `bin/audit-vault.ts`:

```typescript
#!/usr/bin/env tsx
import { resolve } from 'node:path'
import { readFileSync, writeFileSync } from 'node:fs'
import OpenAI from 'openai'
import { loadConfig } from '../src/config.js'
import { initDb } from '../src/db/schema.js'
import { createEmbedder } from '../src/ingest/embedder.js'
import { discoverMarkdownFiles } from '../src/ingest/index.js'
import { extractMarkdown } from '../src/ingest/extractors/markdown.js'
import { validateDraft } from '../src/tools/validateDraft.js'
import type { ValidateDraftResult } from '../src/types.js'

const args = process.argv.slice(2)
let vaultPath: string | undefined

for (const arg of args) {
  if (!arg.startsWith('--')) {
    vaultPath = arg
  }
}

if (!vaultPath) {
  console.error('Usage: pnpm audit:vault <vault-directory>')
  process.exit(1)
}

const config = loadConfig()
if (!config.openaiApiKey) {
  console.error('[ORACLE] Error: OPENAI_API_KEY is not set.')
  process.exit(1)
}

const resolvedPath = resolve(vaultPath)
const openaiClient = new OpenAI({ apiKey: config.openaiApiKey })
const db = initDb(config.dbPath)
const embedder = createEmbedder({ openaiClient, model: config.embeddingModel })
const embedFn = (text: string) => embedder.embedOne(text)
const chatFn = (params: any) => openaiClient.chat.completions.create(params)

const speciesDir = resolve(resolvedPath, 'species')
const files = discoverMarkdownFiles(speciesDir)

console.log(`[ORACLE] Auditing ${files.length} species pages...`)

const results: Array<{ file: string; result: ValidateDraftResult }> = []

for (let i = 0; i < files.length; i++) {
  const filePath = files[i]
  const content = readFileSync(filePath, 'utf-8')
  const extracted = extractMarkdown(content, filePath)
  const speciesName = extracted.frontmatter.scientific_name ?? extracted.metadata.title ?? 'Unknown'

  console.log(`[${i + 1}/${files.length}] Auditing ${speciesName}...`)

  try {
    const result = await validateDraft(db, {
      species_name: speciesName,
      draft_text: content,
      embedFn,
      chatFn,
    })
    results.push({ file: filePath, result })
  } catch (error) {
    const msg = error instanceof Error ? error.message : String(error)
    console.error(`  Failed: ${msg}`)
  }
}

// Generate report
const now = new Date().toISOString().split('T')[0]
const errors = results.filter((r) => r.result.verdict === 'major_issues')
const warnings = results.filter((r) => r.result.verdict === 'needs_revision')
const passing = results.filter((r) => r.result.verdict === 'pass')

let report = `# Vault Audit Report — ${now}\n\n`
report += `## Summary\n\n`
report += `- **Pages checked:** ${results.length}\n`
report += `- **Passing:** ${passing.length}\n`
report += `- **Needs revision:** ${warnings.length}\n`
report += `- **Major issues:** ${errors.length}\n\n`

if (errors.length > 0) {
  report += `## Errors (Major Issues)\n\n`
  for (const { file, result } of errors) {
    report += `### ${result.species_name}\n\n`
    for (const issue of result.issues.filter((i) => i.severity === 'error')) {
      report += `- **${issue.type}:** ${issue.claim}\n  - ${issue.detail}\n`
    }
    report += '\n'
  }
}

if (warnings.length > 0) {
  report += `## Warnings\n\n`
  for (const { file, result } of warnings) {
    report += `### ${result.species_name}\n\n`
    for (const issue of result.issues.filter((i) => i.severity === 'warning')) {
      report += `- **${issue.type}:** ${issue.claim}\n  - ${issue.detail}\n`
    }
    report += '\n'
  }
}

const reportPath = resolve(resolvedPath, 'docs', `audit-report-${now}.md`)
writeFileSync(reportPath, report, 'utf-8')
console.log(`\n[ORACLE] Report written to: ${reportPath}`)
console.log(`[ORACLE] Summary: ${passing.length} pass, ${warnings.length} warnings, ${errors.length} errors`)

db.close()
```

- [ ] **Step 2: Add script to package.json**

Add to `scripts` in `package.json`:

```json
"audit:vault": "tsx bin/audit-vault.ts"
```

- [ ] **Step 3: Commit**

```bash
cd C:\Users\Moonman\mushroom-oracle
git add bin/audit-vault.ts package.json
git commit -m "feat: add batch vault audit CLI with markdown report generation"
```

---

## Task 10: CMS Sync — Field Mapping

**Files:**
- Create: `C:\Users\Moonman\mushroom-oracle\src\sync\fieldMapping.ts`
- Create: `C:\Users\Moonman\mushroom-oracle\data\field_mapping.json`
- Create: `C:\Users\Moonman\mushroom-oracle\tests\sync\fieldMapping.test.ts`

- [ ] **Step 1: Write the failing test**

Create `tests/sync/fieldMapping.test.ts`:

```typescript
import { describe, it, expect } from 'vitest'
import { mapVaultToCms } from '../../src/sync/fieldMapping.js'

const SAMPLE_FRONTMATTER = {
  common_name: "Lion's Mane",
  scientific_name: 'Hericium erinaceus',
  phylum: 'Basidiomycota',
  class: 'Agaricomycetes',
  order: 'Russulales',
  family: 'Hericiaceae',
  tags: ['gourmet', 'medicinal'],
  edibility: 'edible',
  habitat: 'On dead or dying hardwood trees',
  distribution: 'North America, Europe, Asia',
}

const SAMPLE_SECTIONS: Record<string, string> = {
  Identification: 'Forms a single clump of cascading white spines.',
  Biology: 'White-rot saprotroph.',
  Cultivation: 'Grows on hardwood sawdust blocks.',
  'Bioactive Compounds': 'Contains hericenones and erinacines.',
}

describe('mapVaultToCms', () => {
  it('maps frontmatter to CMS taxonomy fields', () => {
    const result = mapVaultToCms(SAMPLE_FRONTMATTER, SAMPLE_SECTIONS)
    expect(result.taxonomy.family).toBe('Hericiaceae')
    expect(result.taxonomy.order).toBe('Russulales')
  })

  it('maps sections to CMS content fields', () => {
    const result = mapVaultToCms(SAMPLE_FRONTMATTER, SAMPLE_SECTIONS)
    expect(result.characteristics).toContain('cascading white spines')
    expect(result.description).toContain('White-rot saprotroph')
  })

  it('maps edibility from frontmatter', () => {
    const result = mapVaultToCms(SAMPLE_FRONTMATTER, SAMPLE_SECTIONS)
    expect(result.edibility).toBe('edible')
  })

  it('maps tags to CMS categories', () => {
    const result = mapVaultToCms(SAMPLE_FRONTMATTER, SAMPLE_SECTIONS)
    expect(result.categories).toContain('gourmet')
    expect(result.categories).toContain('medicinal')
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm vitest run tests/sync/fieldMapping.test.ts`
Expected: FAIL — module not found.

- [ ] **Step 3: Create the field mapping config**

Create `data/field_mapping.json`:

```json
{
  "frontmatter_to_cms": {
    "scientific_name": "species_name",
    "common_name": "common_name",
    "phylum": "taxonomy.phylum",
    "class": "taxonomy.class",
    "order": "taxonomy.order",
    "family": "taxonomy.family",
    "edibility": "edibility",
    "habitat": "habitat",
    "distribution": "distribution",
    "tags": "categories"
  },
  "sections_to_cms": {
    "Identification": "characteristics",
    "Biology": "description",
    "Cultivation": "cultivation_notes",
    "Bioactive Compounds": "chemistry_notes",
    "Culinary": "culinary_notes"
  }
}
```

- [ ] **Step 4: Implement field mapping**

Create `src/sync/fieldMapping.ts`:

```typescript
import type { MarkdownFrontmatter } from '../ingest/extractors/markdown.js'

export interface CmsPayload {
  species_name: string
  common_name: string
  taxonomy: {
    phylum?: string
    class?: string
    order?: string
    family?: string
    genus?: string
    species?: string
  }
  characteristics: string | null
  description: string | null
  cultivation_notes: string | null
  chemistry_notes: string | null
  culinary_notes: string | null
  edibility: string | null
  habitat: string | null
  distribution: string | null
  categories: string[]
}

export function mapVaultToCms(
  frontmatter: MarkdownFrontmatter,
  sections: Record<string, string>,
): CmsPayload {
  return {
    species_name: frontmatter.scientific_name ?? '',
    common_name: frontmatter.common_name ?? '',
    taxonomy: {
      phylum: frontmatter.phylum,
      class: frontmatter.class,
      order: frontmatter.order,
      family: frontmatter.family,
    },
    characteristics: sections['Identification'] ?? null,
    description: sections['Biology'] ?? null,
    cultivation_notes: sections['Cultivation'] ?? null,
    chemistry_notes: sections['Bioactive Compounds'] ?? null,
    culinary_notes: sections['Culinary'] ?? null,
    edibility: frontmatter.edibility ?? null,
    habitat: frontmatter.habitat ?? null,
    distribution: frontmatter.distribution ?? null,
    categories: frontmatter.tags ?? [],
  }
}
```

- [ ] **Step 5: Run test to verify it passes**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm vitest run tests/sync/fieldMapping.test.ts`
Expected: PASS.

- [ ] **Step 6: Commit**

```bash
cd C:\Users\Moonman\mushroom-oracle
git add src/sync/fieldMapping.ts data/field_mapping.json tests/sync/fieldMapping.test.ts
git commit -m "feat: add vault-to-CMS field mapping with config"
```

---

## Task 11: CMS Sync — Payload Client and CLI

**Files:**
- Create: `C:\Users\Moonman\mushroom-oracle\src\sync\payloadClient.ts`
- Create: `C:\Users\Moonman\mushroom-oracle\bin\sync-cms.ts`

- [ ] **Step 1: Implement Payload CMS REST client**

Create `src/sync/payloadClient.ts`:

```typescript
import type { CmsPayload } from './fieldMapping.js'

export interface PayloadConfig {
  baseUrl: string
  apiKey: string
  collection: string
}

export interface SyncResult {
  created: string[]
  updated: string[]
  skipped: string[]
  failed: Array<{ species: string; error: string }>
}

export async function upsertSpecies(
  config: PayloadConfig,
  payload: CmsPayload,
  dryRun: boolean,
): Promise<'created' | 'updated' | 'skipped'> {
  const { baseUrl, apiKey, collection } = config

  const existing = await fetch(
    `${baseUrl}/api/${collection}?where[species_name][equals]=${encodeURIComponent(payload.species_name)}&limit=1`,
    { headers: { Authorization: `Bearer ${apiKey}` } },
  )

  const data = await existing.json()
  const existingDoc = data.docs?.[0]

  if (dryRun) {
    return existingDoc ? 'updated' : 'created'
  }

  if (existingDoc) {
    await fetch(`${baseUrl}/api/${collection}/${existingDoc.id}`, {
      method: 'PATCH',
      headers: {
        Authorization: `Bearer ${apiKey}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(payload),
    })
    return 'updated'
  } else {
    await fetch(`${baseUrl}/api/${collection}`, {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${apiKey}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(payload),
    })
    return 'created'
  }
}
```

- [ ] **Step 2: Create the sync CLI**

Create `bin/sync-cms.ts`:

```typescript
#!/usr/bin/env tsx
import { resolve } from 'node:path'
import { readFileSync } from 'node:fs'
import OpenAI from 'openai'
import { loadConfig } from '../src/config.js'
import { initDb } from '../src/db/schema.js'
import { createEmbedder } from '../src/ingest/embedder.js'
import { discoverMarkdownFiles } from '../src/ingest/index.js'
import { extractMarkdown } from '../src/ingest/extractors/markdown.js'
import { validateDraft } from '../src/tools/validateDraft.js'
import { mapVaultToCms } from '../src/sync/fieldMapping.js'
import { upsertSpecies } from '../src/sync/payloadClient.js'
import type { PayloadConfig, SyncResult } from '../src/sync/payloadClient.js'

const args = process.argv.slice(2)
let vaultPath: string | undefined
let dryRun = false
let force = false

for (const arg of args) {
  if (arg === '--dry-run') dryRun = true
  else if (arg === '--force') force = true
  else if (!arg.startsWith('--')) vaultPath = arg
}

if (!vaultPath) {
  console.error('Usage: pnpm sync:cms <vault-directory> [--dry-run] [--force]')
  process.exit(1)
}

const payloadBaseUrl = process.env.PAYLOAD_CMS_URL
const payloadApiKey = process.env.PAYLOAD_API_KEY
const payloadCollection = process.env.PAYLOAD_COLLECTION ?? 'species'

if (!payloadBaseUrl || !payloadApiKey) {
  console.error('[ORACLE] Error: PAYLOAD_CMS_URL and PAYLOAD_API_KEY must be set in .env')
  process.exit(1)
}

const payloadConfig: PayloadConfig = {
  baseUrl: payloadBaseUrl,
  apiKey: payloadApiKey,
  collection: payloadCollection,
}

const config = loadConfig()
const openaiClient = new OpenAI({ apiKey: config.openaiApiKey })
const db = initDb(config.dbPath)
const embedder = createEmbedder({ openaiClient, model: config.embeddingModel })
const embedFn = (text: string) => embedder.embedOne(text)
const chatFn = (params: any) => openaiClient.chat.completions.create(params)

const resolvedPath = resolve(vaultPath)
const speciesDir = resolve(resolvedPath, 'species')
const files = discoverMarkdownFiles(speciesDir)

console.log(`[ORACLE] Syncing ${files.length} species pages to CMS ${dryRun ? '(DRY RUN)' : ''}`)

const result: SyncResult = { created: [], updated: [], skipped: [], failed: [] }

for (let i = 0; i < files.length; i++) {
  const filePath = files[i]
  const content = readFileSync(filePath, 'utf-8')
  const extracted = extractMarkdown(content, filePath)
  const speciesName = extracted.frontmatter.scientific_name ?? extracted.metadata.title ?? 'Unknown'

  console.log(`[${i + 1}/${files.length}] ${speciesName}...`)

  // Pre-sync validation (skip pages with errors unless --force)
  if (!force) {
    const validation = await validateDraft(db, {
      species_name: speciesName,
      draft_text: content,
      embedFn,
      chatFn,
    })
    if (validation.verdict === 'major_issues') {
      console.log(`  Skipped (validation errors)`)
      result.skipped.push(speciesName)
      continue
    }
  }

  // Build sections map
  const sections: Record<string, string> = {}
  for (const page of extracted.pages) {
    if (page.section) {
      sections[page.section] = page.text
    }
  }

  const cmsPayload = mapVaultToCms(extracted.frontmatter, sections)

  try {
    const action = await upsertSpecies(payloadConfig, cmsPayload, dryRun)
    if (action === 'created') result.created.push(speciesName)
    else if (action === 'updated') result.updated.push(speciesName)
    console.log(`  ${action}`)
  } catch (error) {
    const msg = error instanceof Error ? error.message : String(error)
    console.error(`  Failed: ${msg}`)
    result.failed.push({ species: speciesName, error: msg })
  }
}

console.log(`\n[ORACLE] === Sync Summary ${dryRun ? '(DRY RUN)' : ''} ===`)
console.log(`  Created: ${result.created.length}`)
console.log(`  Updated: ${result.updated.length}`)
console.log(`  Skipped: ${result.skipped.length}`)
console.log(`  Failed:  ${result.failed.length}`)

if (result.failed.length > 0) {
  console.log(`  Failures:`)
  for (const f of result.failed) {
    console.log(`    - ${f.species}: ${f.error}`)
  }
}

db.close()
```

- [ ] **Step 3: Add script to package.json**

Add to `scripts`:

```json
"sync:cms": "tsx bin/sync-cms.ts"
```

- [ ] **Step 4: Add Payload env vars to .env.example**

Append to `.env.example` (create if it doesn't exist):

```
PAYLOAD_CMS_URL=https://your-payload-cms.example.com
PAYLOAD_API_KEY=your-api-key
PAYLOAD_COLLECTION=species
```

- [ ] **Step 5: Commit**

```bash
cd C:\Users\Moonman\mushroom-oracle
git add src/sync/payloadClient.ts bin/sync-cms.ts package.json
git commit -m "feat: add one-way CMS sync with pre-sync validation and dry-run mode"
```

---

## Task 12: Integration Test — Full Workflow

**Files:**
- Create: `C:\Users\Moonman\mushroom-oracle\tests\integration\vault-workflow.test.ts`

- [ ] **Step 1: Write integration test that exercises the full flow**

Create `tests/integration/vault-workflow.test.ts`:

```typescript
import { describe, it, expect, vi } from 'vitest'
import { initDb } from '../../src/db/schema.js'
import { insertSource, insertChunk, getVaultSpeciesNames } from '../../src/db/store.js'
import { extractMarkdown } from '../../src/ingest/extractors/markdown.js'
import { mapVaultToCms } from '../../src/sync/fieldMapping.js'
import { validateDraft } from '../../src/tools/validateDraft.js'
import type { EnrichedMetadata } from '../../src/types.js'

const VAULT_PAGE = `---
common_name: Turkey Tail
scientific_name: Trametes versicolor
phylum: Basidiomycota
class: Agaricomycetes
order: Polyporales
family: Polyporaceae
tags:
  - medicinal
  - common
edibility: inedible (too tough)
habitat: Dead hardwood logs and stumps
distribution: Worldwide
last_verified: 2026-05-01
---

## Identification

Thin, flexible, semicircular brackets with concentric color zones. Pore surface white.

## Biology

White-rot saprotroph on dead hardwood. One of the most common bracket fungi worldwide.

## Bioactive Compounds

Contains polysaccharide-K (PSK) and polysaccharopeptide (PSP). Used in cancer adjunct therapy in Japan.
`

describe('Vault workflow integration', () => {
  it('extracts markdown, maps to CMS, and validates', async () => {
    const db = initDb(':memory:')

    // Simulate research evidence in the DB
    insertSource(db, {
      id: 'research1',
      file_path: '/papers/turkey-tail-review.pdf',
      title: 'Turkey Tail Immunology Review',
      authors: JSON.stringify(['Chen W']),
      year: 2022,
      source_type: 'research',
    })

    const embedding = new Float32Array(1536).fill(0.1)
    insertChunk(db, {
      sourceId: 'research1',
      chunkIndex: 0,
      pageRange: 'p. 8',
      section: null,
      text: 'Trametes versicolor PSK has demonstrated immunomodulatory activity in clinical trials as adjunct cancer therapy.',
      embedding,
      metadata: {
        taxon_names: ['Trametes versicolor'],
        common_names: ['Turkey Tail'],
        accepted_name: 'Trametes versicolor',
        synonyms: [],
        family: 'Polyporaceae',
        topics: ['medicinal'],
        region: null,
        edibility_claims: [],
        toxicity_claims: [],
        lookalikes: [],
        diagnostic_features: [],
        cultivation_notes: [],
      },
    })

    // Step 1: Extract markdown
    const extracted = extractMarkdown(VAULT_PAGE, 'Trametes versicolor.md')
    expect(extracted.frontmatter.scientific_name).toBe('Trametes versicolor')
    expect(extracted.pages.length).toBe(3)

    // Step 2: Map to CMS
    const sections: Record<string, string> = {}
    for (const page of extracted.pages) {
      if (page.section) sections[page.section] = page.text
    }
    const cmsPayload = mapVaultToCms(extracted.frontmatter, sections)
    expect(cmsPayload.species_name).toBe('Trametes versicolor')
    expect(cmsPayload.taxonomy.family).toBe('Polyporaceae')
    expect(cmsPayload.categories).toContain('medicinal')

    // Step 3: Simulate vault source insertion
    insertSource(db, {
      id: 'vault1',
      file_path: 'C:\\vault\\species\\Trametes versicolor.md',
      title: 'Trametes versicolor',
      source_type: 'vault',
    })
    insertChunk(db, {
      sourceId: 'vault1',
      chunkIndex: 0,
      pageRange: null,
      section: 'Bioactive Compounds',
      text: 'Contains PSK and PSP. Used in cancer adjunct therapy in Japan.',
      embedding,
      metadata: {
        taxon_names: ['Trametes versicolor'],
        common_names: ['Turkey Tail'],
        accepted_name: 'Trametes versicolor',
        synonyms: [],
        family: 'Polyporaceae',
        topics: ['medicinal'],
        region: null,
        edibility_claims: [],
        toxicity_claims: [],
        lookalikes: [],
        diagnostic_features: [],
        cultivation_notes: [],
      },
    })

    // Step 4: Verify vault species lookup
    const vaultSpecies = getVaultSpeciesNames(db)
    expect(vaultSpecies).toContain('Trametes versicolor')

    // Step 5: Validate (with mocked LLM)
    const mockEmbedFn = vi.fn().mockResolvedValue(new Float32Array(1536).fill(0.1))
    const mockChatFn = vi.fn().mockResolvedValue({
      choices: [{ message: { content: JSON.stringify({ issues: [], total_claims_checked: 4 }) } }],
    })

    const validation = await validateDraft(db, {
      species_name: 'Trametes versicolor',
      draft_text: VAULT_PAGE,
      embedFn: mockEmbedFn,
      chatFn: mockChatFn,
    })

    expect(validation.verdict).toBe('pass')
    expect(validation.species_name).toBe('Trametes versicolor')
  })
})
```

- [ ] **Step 2: Run the integration test**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm vitest run tests/integration/vault-workflow.test.ts`
Expected: PASS.

- [ ] **Step 3: Commit**

```bash
cd C:\Users\Moonman\mushroom-oracle
git add tests/integration/vault-workflow.test.ts
git commit -m "test: add integration test for full vault workflow (extract → map → validate)"
```

---

## Task 13: Final Verification and Cleanup

- [ ] **Step 1: Run full test suite**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm test`
Expected: All tests pass.

- [ ] **Step 2: Verify vault ingestion works end-to-end**

Run: `cd C:\Users\Moonman\mushroom-oracle && pnpm ingest:vault "C:\Users\Moonman\MushroomOracle"`
Expected: Ingests 148 markdown files, reports processed count, builds updated taxon reference.

- [ ] **Step 3: Verify MCP server starts with new tools**

Run: `cd C:\Users\Moonman\mushroom-oracle && timeout 5 pnpm serve 2>&1 || true`
Expected: See `[ORACLE] MCP server running on stdio` without errors.

- [ ] **Step 4: Commit any final fixes**

```bash
cd C:\Users\Moonman\mushroom-oracle
git add -A
git commit -m "chore: final cleanup and verification"
```
