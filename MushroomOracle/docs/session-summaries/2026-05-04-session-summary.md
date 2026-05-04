# Session Summary — May 4, 2026

Branch: `main` (3 commits in MushroomOracle on top of `70d5bc4`, 1 commit in mushroom-oracle on top of `2a2f500`)

Total: **36 files changed, 2,343 insertions** in MushroomOracle; **1 file changed, 2 insertions** in mushroom-oracle.

---

## 1. Deep-Dive Research Across Four Mushroom Domains

### Original State
The MushroomOracle vault had strong coverage of biology, taxonomy, ecology, cultivation, health/medicinal, culinary, industrial, history, and identification — but no content on fermentation science, food technology, citizen science, molecular taxonomy, or the commercial mycotechnology landscape. The source acquisition strategy was ad hoc.

### Problem
The user wanted to identify the most interesting and content-rich domains adjacent to the existing vault, understand where to find high-quality source material for each, and build a concrete plan for systematic expansion.

### Changes Made
Ran four parallel research agents, each performing web searches across academic databases, industry reports, and organizational websites:

1. **Mycotechnology** — mycelium materials, biocomputing (Adamatzky lab), construction (MycoHAB), commercial landscape ($3.6B market), key failures (MycoWorks insolvency, Bolt Threads restructuring)
2. **Psychedelic science** — COMPASS Phase 3 results (COMP005: -3.6 MADRS, COMP006: -3.8 MADRS), three neuroplasticity pathways, microdosing self-blinding trial (no benefits beyond placebo), Oregon/Colorado/Australia legal landscape, therapy protocols
3. **Fermentation & food science** — koji enzymology (65 endopeptidases, 69 exopeptidases), tempeh B12 co-fermentation, mycoprotein (Quorn/Meati/Nature's Fynd), precision fermentation (Perfect Day whey), umami synergy (30x amplification)
4. **Citizen science & DNA barcoding** — ITS as universal barcode (Schoch 2012), eDNA detecting 2-10x more species, UNITE database (3.85M sequences), NAMA MinION mass barcoding (803 barcodes at 83.7% success), sequencing at $3.50/reaction

Research artifact saved at `artifacts/research/2026-05-04-mushroom-domains-deep-dive.md` (228 lines).

### Final State
Comprehensive research dossier covering four domains with source-backed claims, key researchers, organizations, journals, and databases identified. Ready for vault integration.

---

## 2. Vault Expansion: Three New Sections and 17 Seed Pages

### Original State
The vault had 13 directories covering established domains. No content existed for fermentation science, citizen science/molecular taxonomy, or source tracking.

### Problem
Needed to translate the research findings into vault pages that match the existing style (frontmatter, wiki-links, tables, citations, content-creator notes) and are immediately useful for content creation.

### Changes Made
Created three new vault sections with seed pages:

**`fermentation/` (6 pages):**
- Koji Fermentation — A. oryzae enzyme arsenal, umami mechanism, Western applications
- Umami Science — glutamate/GMP/synergistic effect, drying concentration, T1R1/T1R3 receptor
- Mycoprotein — Quorn (F. venenatum), Meati (N. crassa), Nature's Fynd, texture science
- Tempeh and Rhizopus — fermentation process, nutritional transformation, B12 co-fermentation, novel substrates, traditional production
- Precision Fermentation — GEMs, Perfect Day/EVERY/New Culture, process workflow, non-food applications
- Mushroom Food Ingredients — species-specific powders, functional beverages, chitosan, blenditarian concept

**`citizen-science/` (5 pages):**
- DNA Barcoding — ITS region, limitations, supplementary markers (TEF1-α, RPB1/2)
- Environmental DNA — metabarcoding workflow, key studies (2-10x detection), limitations/biases
- Citizen Science Platforms — iNaturalist (70% wrong for fungi), Mushroom Observer, FunDiS, MycoMap, BioBlitz, dark taxa
- Fungal Databases — MycoBank, Index Fungorum, Species Fungorum, GenBank, UNITE, GBIF, FungiDB, MushroomExpert.com
- Sequencing Accessibility — Sanger at $3.50/reaction, MinION field sequencing, NAMA 2024 mass barcoding

**`health/` (3 new pages):**
- Microdosing Evidence — self-blinding trial, controlled lab studies, evidence does not support benefits beyond placebo
- Psychedelic Therapy Protocols — three-phase structure, music playlist, FDA REMS requirements
- Psychedelic Legal Landscape — Oregon (16K clients, 6 adverse reactions, market contraction), Colorado, Australia, Canada, FDA pathway

**`industrial/` (2 new pages):**
- Fungal Biocomputing — Adamatzky electrical signaling, Boolean logic, MycelioTronics, memristors, smart buildings
- Mycelium Construction — MycoHAB, self-healing materials, Biohm, Mogu, thermal/fire properties

**`sources/` (1 page):**
- Source Registry — 50+ journals, 9 databases, 15+ organizations, 8 key books, industry intelligence sources, prioritized acquisition order

### Final State
17 seed pages totaling 1,837 lines of content across 3 new sections and 2 expanded existing sections. All pages follow vault conventions: YAML frontmatter with tags and last_verified, wiki-links to related pages, source citations, tables for structured data.

---

## 3. Cross-Linking and Species Pages

### Original State
New pages contained wiki-links to existing vault pages, but existing pages had no backlinks to the new content. Four organisms referenced heavily across new sections — *Aspergillus oryzae*, *Rhizopus oligosporus*, *Fusarium venenatum*, *Neurospora crassa* — had no species pages.

### Problem
Obsidian's graph view and backlink panels are only useful when links are bidirectional. Missing species pages created dead wiki-links throughout the new sections.

### Changes Made
**Cross-linking (via background agent):** Edited 14 existing vault pages to add backlinks to new content:
- `culinary/Flavor Profiles.md`, `Cooking Techniques.md`, `Preservation Methods.md` → fermentation links
- `health/Psilocybin Research.md`, `Nutritional Profiles.md`, `Safety and Contraindications.md` → psychedelic/food science links
- `industrial/Myco-Materials.md`, `Biotech Applications.md`, `Mycoremediation.md` → construction/biocomputing/fermentation links
- `ecology/Soil Health and Ecosystem Services.md`, `biology/Spore Biology.md`, `Mycelium Structure.md` → eDNA/biocomputing links
- `taxonomy/Classification Methods.md`, `identification/Morphological Identification.md` → barcoding/platforms links

**4 new species pages:**
- *Aspergillus oryzae* — domestication from A. flavus, 37 Mb genome, enzyme arsenal, GRAS status, koji applications
- *Rhizopus oligosporus* — Mucoromycota taxonomy, mechanical binding, phytase, B12 co-fermentation
- *Fusarium venenatum* — Quorn production (300 kg/hr, 150K-liter fermenters), nutritional profile, environmental footprint (55x lower carbon than beef)
- *Neurospora crassa* — 1958 Nobel Prize (Beadle & Tatum), genome (first filamentous fungus sequenced), circadian biology, Meati/oncom/self-healing materials

### Final State
Vault now has 49 species pages (up from 45). All new sections are bidirectionally linked to existing content. Zero dead wiki-links across new pages.

---

## 4. Seed Page Expansion

### Original State
Four seed pages were below the vault's depth standard at 66-79 lines, compared to existing pages averaging 100-200 lines.

### Problem
Thin pages would feel incomplete next to the rest of the vault and provide less value for content creation.

### Changes Made
Expanded four pages with additional sections:
- **Precision Fermentation** (+24 lines): Added step-by-step process workflow (gene insertion → fermentation → separation → purification → processing) and non-food applications section
- **Tempeh and Rhizopus** (+30 lines): Added traditional Indonesian production details, quality indicators, comparison table (tempeh vs. oncom vs. Quorn vs. Meati), content creator notes
- **Mushroom Food Ingredients** (+14 lines): Added species-specific powder applications table (shiitake, porcini, button, maitake, lion's mane) and mushroom coffee/functional beverages section
- **Environmental DNA** (+16 lines): Added limitations and biases section (PCR bias, DNA persistence, quantification challenges, reference database gaps, cost)

### Final State
All pages now range from 80-139 lines, within the vault's depth standard.

---

## 5. Content Pipeline Bootstrap: Database Population and First Audit

### Original State
The mushroom-oracle MCP server code (15 tools, vault ingestion pipeline, batch audit CLI) existed from a previous session but had never been run. No database existed (`oracle.db` absent). The pipeline was entirely theoretical.

### Problem
The pipeline needed to be exercised end-to-end to prove it worked: vault ingestion → embedding → search → validation → report generation.

### Changes Made
1. **Vault ingestion**: Ran `pnpm ingest:vault` against the full MushroomOracle vault. Processed 179 markdown files, extracting sections by `##` headings, generating embeddings via OpenAI text-embedding-3-small, and storing in SQLite with sqlite-vec.

2. **Search verification**: Tested the query interface with "what are the health benefits of lion's mane" — returned 12 relevant chunks with similarity scores, taxa metadata, and topic tags. Search is working correctly.

3. **Batch audit**: Ran `pnpm audit:vault` on all 49 species pages. Each page was validated against the evidence base via GPT-4o-mini.

4. **Bug fix**: Fixed a pre-existing Windows path separator issue in `tests/ingest/orchestrator.test.ts` — replaced `f.split('/').pop()` with `path.basename(f)`. Test suite now passes 100/100 (5 skipped).

**First audit attempted the wrong path** (`species/species` doubled directory), discovered the CLI appends `/species` to the vault root. Second run succeeded.

### Final State
- **Database**: 179 sources, 581 chunks, 71 species indexed in `data/oracle.db`
- **Test suite**: 100 passed, 0 failed, 5 skipped (all in mushroom-oracle)
- **Audit report**: `docs/audit-report-2026-05-04.md` — 49 species audited. 2 pass, 47 flagged with `missing_safety` issues
- **Audit finding quality**: Most flags are false positives from GPT-4o-mini hallucinating implausible look-alike confusions (e.g., *Trametes versicolor* confused with *Galerina marginata*). One genuine contradiction found: *Ganoderma lucidum* described as white rot in vault but evidence says brown rot. The validator needs prompt tuning to reduce false-positive safety flags, and the evidence base needs external research papers (currently only vault content is ingested, so it's cross-referencing pages against each other).

---

## Files Created/Modified

### New files (MushroomOracle)

| File | Purpose |
|------|---------|
| `fermentation/Koji Fermentation.md` | A. oryzae enzyme science, umami mechanism, Western applications |
| `fermentation/Umami Science.md` | Glutamate, GMP, synergistic effect, T1R1/T1R3 receptor |
| `fermentation/Mycoprotein.md` | Quorn, Meati, Nature's Fynd, texture science |
| `fermentation/Tempeh and Rhizopus.md` | R. oligosporus fermentation, B12, novel substrates |
| `fermentation/Precision Fermentation.md` | GEMs, Perfect Day, EVERY, New Culture |
| `fermentation/Mushroom Food Ingredients.md` | Powders, chitosan, blenditarian, functional beverages |
| `citizen-science/DNA Barcoding.md` | ITS barcode, supplementary markers, taxonomy revolution |
| `citizen-science/Environmental DNA.md` | Metabarcoding, eDNA vs. surveys, air sampling, limitations |
| `citizen-science/Citizen Science Platforms.md` | iNaturalist, Mushroom Observer, FunDiS, MycoMap |
| `citizen-science/Fungal Databases.md` | MycoBank, UNITE, GenBank, GBIF, FungiDB |
| `citizen-science/Sequencing Accessibility.md` | Sanger costs, MinION, NAMA mass barcoding |
| `health/Microdosing Evidence.md` | Self-blinding trial, controlled studies, placebo finding |
| `health/Psychedelic Therapy Protocols.md` | Three-phase model, music, FDA REMS |
| `health/Psychedelic Legal Landscape.md` | Oregon, Colorado, Australia, Canada, FDA pathway |
| `industrial/Fungal Biocomputing.md` | Adamatzky, electrical signaling, logic gates, memristors |
| `industrial/Mycelium Construction.md` | MycoHAB, self-healing materials, Biohm, Mogu |
| `sources/Source Registry.md` | 50+ journals, 9 databases, 15+ orgs, acquisition priority |
| `species/Aspergillus oryzae.md` | Koji mold species page |
| `species/Fusarium venenatum.md` | Quorn fungus species page |
| `species/Neurospora crassa.md` | Red bread mold / Meati fungus species page |
| `species/Rhizopus oligosporus.md` | Tempeh mold species page |
| `artifacts/research/2026-05-04-mushroom-domains-deep-dive.md` | Full research synthesis |
| `docs/session-summaries/2026-05-04-session-summary.md` | This file |

### Modified files (MushroomOracle)

| File | Change |
|------|--------|
| `biology/Mycelium Structure.md` | Added links to Fungal Biocomputing, Mushroom Food Ingredients |
| `biology/Spore Biology.md` | Added link to Environmental DNA |
| `culinary/Cooking Techniques.md` | Added links to Umami Science, Koji Fermentation |
| `culinary/Flavor Profiles.md` | Added links to Umami Science, Koji Fermentation, Mushroom Food Ingredients |
| `culinary/Preservation Methods.md` | Added links to Umami Science, Mushroom Food Ingredients |
| `ecology/Soil Health and Ecosystem Services.md` | Added Related Notes section with Environmental DNA link |
| `health/Nutritional Profiles.md` | Added links to Mycoprotein, Tempeh, Mushroom Food Ingredients |
| `health/Psilocybin Research.md` | Added links to Microdosing Evidence, Therapy Protocols, Legal Landscape |
| `health/Safety and Contraindications.md` | Added links to Therapy Protocols, Legal Landscape |
| `identification/Morphological Identification.md` | Added links to DNA Barcoding, Citizen Science Platforms |
| `industrial/Biotech Applications.md` | Added links to Koji Fermentation, Precision Fermentation, Mycoprotein |
| `industrial/Myco-Materials.md` | Added links to Mycelium Construction, Fungal Biocomputing |
| `industrial/Mycoremediation.md` | Added link to Mycelium Construction |
| `taxonomy/Classification Methods.md` | Added links to DNA Barcoding, Fungal Databases |

### New/modified files (mushroom-oracle)

| File | Change |
|------|--------|
| `tests/ingest/orchestrator.test.ts` | Fixed Windows path separator bug (split('/') → basename()) |

### Generated files (not committed)

| File | Contents |
|------|----------|
| `mushroom-oracle/data/oracle.db` | SQLite database: 179 sources, 581 chunks, 71 species |
| `docs/audit-report-2026-05-04.md` | Batch audit of 49 species: 2 pass, 47 flagged |
| `docs/session-summaries/2026-05-01-session-summary.md` | Moved from `docs/` to `docs/session-summaries/` |
