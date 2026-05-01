# MushroomOracle Knowledge Base Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a comprehensive Obsidian vault of mushroom knowledge organized by topic domain, with species profiles as cross-referenced anchors and a facts library for content creation.

**Architecture:** Markdown files with YAML frontmatter, organized in topic directories. Species files are the core anchors, linked from topic notes via `[[wiki links]]`. A `facts/` directory holds bite-sized, citation-backed claims for social media and articles. Templates enforce consistent structure.

**Tech Stack:** Obsidian vault (plain Markdown + YAML frontmatter), git for version control

**Source material:** `artifacts/research/2026-05-01-mushroom-encyclopedia-deep-dive.md` (2,931 lines) — the existing research to migrate. Section map:
- Lines 26–1161: Biology & Taxonomy (taxonomy, life cycle, ecology, 19 species profiles)
- Lines 1162–1970: Cultivation (substrates, sterile technique, growing methods, environmental params, harvesting, troubleshooting)
- Lines 1971–2809: Health & Medicinal (nutrition, bioactive compounds, species-specific research, psilocybin, extraction, safety)

---

## Phase 1: Scaffold + Migrate Existing Research

### Task 1: Create vault structure and templates

**Files:**
- Create: `templates/Species Template.md`
- Create: `templates/Topic Template.md`
- Create: `templates/Fact Template.md`
- Create: `templates/Compound Template.md`
- Create directories: `species/`, `taxonomy/`, `biology/`, `cultivation/`, `health/`, `identification/`, `culinary/`, `ecology/`, `industrial/`, `history/`, `facts/`

- [ ] **Step 1: Create all directories**

```bash
cd C:/Users/Moonman/MushroomOracle
mkdir -p species taxonomy biology cultivation health identification culinary ecology industrial history facts templates
```

- [ ] **Step 2: Create Species Template**

Write `templates/Species Template.md`:

```markdown
---
common_name: 
scientific_name: 
phylum: 
class: 
order: 
family: 
tags: []
edibility: 
habitat: 
distribution: 
last_verified: {{date:YYYY-MM-DD}}
---

# {{scientific_name}} ({{common_name}})

## Identification


## Biology


## Cultivation


## Bioactive Compounds


## Culinary


## Sources

```

- [ ] **Step 3: Create Topic Template**

Write `templates/Topic Template.md`:

```markdown
---
tags: []
last_verified: {{date:YYYY-MM-DD}}
---

# {{title}}

## Overview


## Details


## Related
- 

## Sources

```

- [ ] **Step 4: Create Fact Template**

Write `templates/Fact Template.md`:

```markdown
---
claim: ""
species: []
compounds: []
confidence: 
source_type: 
last_verified: {{date:YYYY-MM-DD}}
tags: []
---

# {{title}}



**Key evidence:**
- 

**Context for content creators:**


## Sources

```

- [ ] **Step 5: Create Compound Template**

Write `templates/Compound Template.md`:

```markdown
---
compound_name: 
compound_class: 
found_in: []
tags: []
last_verified: {{date:YYYY-MM-DD}}
---

# {{compound_name}}

## Overview


## Mechanism of Action


## Species Sources


## Research Evidence


## Sources

```

- [ ] **Step 6: Commit**

```bash
git add templates/ species/ taxonomy/ biology/ cultivation/ health/ identification/ culinary/ ecology/ industrial/ history/ facts/
git commit -m "scaffold: create vault structure and templates"
```

Note: git won't track empty directories. Add a `.gitkeep` in each empty directory, or wait until content is added. Templates directory will have files so it will be tracked.

---

### Task 2: Migrate taxonomy content

**Source:** `artifacts/research/2026-05-01-mushroom-encyclopedia-deep-dive.md` lines 26–170
**Files:**
- Create: `taxonomy/Kingdom Fungi.md`
- Create: `taxonomy/Basidiomycota.md`
- Create: `taxonomy/Ascomycota.md`
- Create: `taxonomy/Minor Phyla.md`
- Create: `taxonomy/Classification Methods.md`

- [ ] **Step 1: Read source lines 26–170**

Read the existing research doc from line 26 to line 170. This covers:
- 1.1 Overview of the Fungal Kingdom
- 1.2 Full Taxonomic Hierarchy
- 1.3 The Nine Major Phyla
- 1.4 Basidiomycota vs Ascomycota
- 1.5 Morphological vs Molecular Phylogenetics
- 1.6 Notable Genera

- [ ] **Step 2: Create `taxonomy/Kingdom Fungi.md`**

Extract section 1.1 (Overview) and 1.2 (Taxonomic Hierarchy). Add frontmatter:

```yaml
---
tags:
  - taxonomy
  - fundamentals
last_verified: 2026-05-01
---
```

Add wiki links to `[[Basidiomycota]]`, `[[Ascomycota]]`, `[[Classification Methods]]`. Include all factual content from the source — do not summarize or truncate.

- [ ] **Step 3: Create `taxonomy/Basidiomycota.md`**

Extract section 1.3.1 (Basidiomycota) and the Basidiomycota portion of section 1.4. Add frontmatter with tags `taxonomy`, `basidiomycota`. Wiki-link to relevant species (`[[Agaricus bisporus]]`, `[[Pleurotus ostreatus]]`, etc.) and to `[[Ascomycota]]`, `[[Kingdom Fungi]]`.

- [ ] **Step 4: Create `taxonomy/Ascomycota.md`**

Extract section 1.3.2 (Ascomycota) and the Ascomycota portion of section 1.4. Same frontmatter pattern. Wiki-link to `[[Morchella]]` species and back to `[[Kingdom Fungi]]`.

- [ ] **Step 5: Create `taxonomy/Minor Phyla.md`**

Extract sections 1.3.3 through 1.3.9 (Glomeromycota, Chytridiomycota, Blastocladiomycota, Neocallimastigomycota, Zoopagomycota, Mucoromycota, Microsporidia). Single file since these are shorter. Wiki-link to `[[Kingdom Fungi]]`, `[[Mycorrhizal Relationships]]`.

- [ ] **Step 6: Create `taxonomy/Classification Methods.md`**

Extract section 1.5 (Morphological vs Molecular Phylogenetics) and 1.6 (Notable Genera). Wiki-link to `[[Kingdom Fungi]]`, relevant species.

- [ ] **Step 7: Commit**

```bash
git add taxonomy/
git commit -m "content: migrate taxonomy notes from existing research"
```

---

### Task 3: Migrate biology content

**Source:** Lines 173–390
**Files:**
- Create: `biology/Fungal Life Cycle.md`
- Create: `biology/Spore Biology.md`
- Create: `biology/Mycelium Structure.md`
- Create: `biology/Fruiting Body Anatomy.md`
- Create: `biology/Fruiting Triggers.md`
- Create: `biology/Mating and Reproduction.md`

- [ ] **Step 1: Read source lines 173–390**

This covers:
- 2.1 Complete Fungal Life Cycle
- 2.2 Mating Types and Sexual Reproduction
- 2.3 Spore Types and Dispersal
- 2.4 Mycelium Structure
- 2.5 Fruiting Body Anatomy
- 2.6 Environmental Triggers for Fruiting

- [ ] **Step 2: Create `biology/Fungal Life Cycle.md`**

Extract section 2.1. Frontmatter tags: `biology`, `fundamentals`. Wiki-link to `[[Spore Biology]]`, `[[Mycelium Structure]]`, `[[Fruiting Body Anatomy]]`, `[[Mating and Reproduction]]`. This is the hub note for biology — link to all other biology notes.

- [ ] **Step 3: Create `biology/Mating and Reproduction.md`**

Extract section 2.2. Wiki-link to `[[Fungal Life Cycle]]`, `[[Spore Biology]]`.

- [ ] **Step 4: Create `biology/Spore Biology.md`**

Extract section 2.3. Wiki-link to `[[Fungal Life Cycle]]`, `[[Spore Prints]]` (identification), relevant species.

- [ ] **Step 5: Create `biology/Mycelium Structure.md`**

Extract section 2.4. Wiki-link to `[[Fungal Life Cycle]]`, `[[Mycelial Networks]]` (ecology), `[[Substrates]]` (cultivation).

- [ ] **Step 6: Create `biology/Fruiting Body Anatomy.md`**

Extract section 2.5. Wiki-link to `[[Fungal Life Cycle]]`, `[[Fruiting Triggers]]`, species pages.

- [ ] **Step 7: Create `biology/Fruiting Triggers.md`**

Extract section 2.6. Wiki-link to `[[Environmental Parameters]]` (cultivation), `[[Fruiting Body Anatomy]]`, species pages.

- [ ] **Step 8: Commit**

```bash
git add biology/
git commit -m "content: migrate biology notes from existing research"
```

---

### Task 4: Migrate ecology content

**Source:** Lines 390–540
**Files:**
- Create: `ecology/Mycorrhizal Relationships.md`
- Create: `ecology/Decomposition and Carbon Cycling.md`
- Create: `ecology/Wood Decay.md`
- Create: `ecology/Common Mycorrhizal Networks.md`
- Create: `ecology/Parasitic Fungi.md`
- Create: `ecology/Soil Health and Ecosystem Services.md`

- [ ] **Step 1: Read source lines 390–540**

Covers sections 3.1–3.6: mycorrhizae, saprophytes, white/brown rot, Wood Wide Web, parasitic fungi, soil health.

- [ ] **Step 2: Create each note**

For each section (3.1 through 3.6), create a separate note file with:
- YAML frontmatter: tags `ecology` plus relevant subtags, `last_verified: 2026-05-01`
- Full content from the source — no truncation
- Wiki links to related species (`[[Amanita muscaria]]`, `[[Armillaria]]`, etc.)
- Wiki links to related topics (`[[Mycelium Structure]]`, `[[Substrates]]`, `[[Mycoremediation]]`)
- Sources section at bottom

Files to create:
- `ecology/Mycorrhizal Relationships.md` ← section 3.1 (ectomycorrhizal, arbuscular, ericoid, orchid)
- `ecology/Decomposition and Carbon Cycling.md` ← section 3.2
- `ecology/Wood Decay.md` ← section 3.3 (white rot vs brown rot)
- `ecology/Common Mycorrhizal Networks.md` ← section 3.4 (the "Wood Wide Web" — include the scientific controversy)
- `ecology/Parasitic Fungi.md` ← section 3.5
- `ecology/Soil Health and Ecosystem Services.md` ← section 3.6

- [ ] **Step 3: Commit**

```bash
git add ecology/
git commit -m "content: migrate ecology notes from existing research"
```

---

### Task 5: Migrate species profiles

**Source:** Lines 541–1161 (Part 4 of existing research)
**Files:**
- Create: 19 files in `species/`, one per species

- [ ] **Step 1: Read source lines 541–1161**

This covers species profiles for:

**Gourmet (10):** Pleurotus ostreatus, Lentinula edodes, Hericium erinaceus, Grifola frondosa, Morchella spp., Cantharellus cibarius, Agaricus bisporus, Flammulina velutipes, Laetiporus sulphureus, Boletus edulis

**Medicinal (5):** Ganoderma lucidum, Trametes versicolor, Cordyceps militaris, Inonotus obliquus, Ophiocordyceps sinensis

**Psychoactive (4):** Psilocybe cubensis, Psilocybe semilanceata, Psilocybe azurescens, Amanita muscaria

- [ ] **Step 2: Create species files — gourmet batch**

For each of the 10 gourmet species, create `species/<Scientific name>.md` with:

Full species frontmatter per the Species Template:
```yaml
---
common_name: <from source>
scientific_name: <from source>
phylum: <from source>
class: <from source>
order: <from source>
family: <from source>
tags:
  - gourmet
  - <additional relevant tags>
edibility: <from source>
habitat: <from source>
distribution: <from source>
last_verified: 2026-05-01
---
```

Sections: `## Identification`, `## Biology`, `## Cultivation`, `## Bioactive Compounds`, `## Culinary`, `## Sources`

Populate each section with content from the existing research. For sections where the existing research is thin (e.g., Culinary for most species), include what's available and note that expansion is planned.

Wiki-link to relevant topic notes: `[[Substrates]]`, `[[Environmental Parameters]]`, `[[Fruiting Triggers]]`, taxonomy notes, ecology notes.

- [ ] **Step 3: Create species files — medicinal batch**

Same process for the 5 medicinal species. Additional tags: `medicinal`. Wiki-link to `[[Beta-Glucans]]`, compound notes, health notes.

- [ ] **Step 4: Create species files — psychoactive batch**

Same process for the 4 psychoactive species. Additional tags: `psychoactive`. Wiki-link to `[[Psilocybin Research]]`.

For Amanita muscaria: tag as `psychoactive`, `poisonous` (ibotenic acid/muscimol, not psilocybin).

- [ ] **Step 5: Commit**

```bash
git add species/
git commit -m "content: migrate 19 species profiles from existing research"
```

---

### Task 6: Migrate cultivation content

**Source:** Lines 1162–1970
**Files:**
- Create: `cultivation/Substrates.md`
- Create: `cultivation/Sterile Technique.md`
- Create: `cultivation/Contamination.md`
- Create: `cultivation/Growing Methods.md`
- Create: `cultivation/Environmental Parameters.md`
- Create: `cultivation/Harvesting and Post-Harvest.md`
- Create: `cultivation/Troubleshooting.md`

- [ ] **Step 1: Read source lines 1162–1970**

Covers: substrate science, sterile technique & lab work, growing methods (PF Tek, monotub, bag culture, logs, outdoor beds, Martha tent, bucket tek, commercial), environmental parameters by species, harvesting/post-harvest, troubleshooting.

- [ ] **Step 2: Create cultivation notes**

For each file:
- YAML frontmatter: tags `cultivation` plus subtags, `last_verified: 2026-05-01`
- Full content from source — preserve all tables, numbers, parameters, recipes
- Wiki-link to relevant species pages (e.g., `[[Pleurotus ostreatus]]` in bucket tek section)
- Wiki-link across cultivation notes (e.g., Substrates ↔ Growing Methods ↔ Environmental Parameters)
- Sources section

Mapping:
- `Substrates.md` ← Section 1 (straw, sawdust, grain, manure, logs, cardboard, coffee grounds, C:N ratios, moisture, supplementation)
- `Sterile Technique.md` ← Section 2 (laminar flow, SAB, agar work, grain spawn, liquid culture)
- `Contamination.md` ← Section 2 contamination portions (Trichoderma, cobweb, bacillus, prevention strategies)
- `Growing Methods.md` ← Section 3 (PF Tek, monotub, bag culture, log inoculation, outdoor beds, Martha tent, bucket tek, commercial)
- `Environmental Parameters.md` ← Section 4 (species-by-species tables — preserve all temperature, humidity, FAE, light, timeline data)
- `Harvesting and Post-Harvest.md` ← Section 5 (harvest timing, spore prints, drying, storage, yield expectations)
- `Troubleshooting.md` ← Section 6 (overlay, aborts, bacterial blotch, side pinning, fuzzy feet, metabolites, wet/dry bubble)

- [ ] **Step 3: Commit**

```bash
git add cultivation/
git commit -m "content: migrate cultivation notes from existing research"
```

---

### Task 7: Migrate health and medicinal content

**Source:** Lines 1971–2809
**Files:**
- Create: `health/Nutritional Profiles.md`
- Create: `health/Beta-Glucans.md`
- Create: `health/Terpenoids and Triterpenoids.md`
- Create: `health/Hericenones and Erinacines.md`
- Create: `health/Cordycepin.md`
- Create: `health/Psilocybin Research.md`
- Create: `health/Extraction Methods.md`
- Create: `health/Safety and Contraindications.md`
- Create: `health/Ergothioneine.md`

- [ ] **Step 1: Read source lines 1971–2809**

Covers: nutritional science, bioactive compounds (beta-glucans, terpenoids, hericenones/erinacines, cordycepin, psilocybin), species-specific health research, psilocybin clinical landscape, extraction methods, safety/contraindications.

- [ ] **Step 2: Create health notes**

For each file:
- YAML frontmatter: tags `health`, `medicinal`, `bioactive-compounds` as relevant, `last_verified: 2026-05-01`
- Full content — preserve all study citations, dosages, mechanisms, tables
- Wiki-link to species pages (e.g., `[[Hericium erinaceus]]` from Hericenones note)
- Wiki-link across health notes
- Sources section

Mapping:
- `Nutritional Profiles.md` ← Section 1 (macros, micros, vitamin D, ergothioneine, chitin, amino acids)
- `Beta-Glucans.md` ← Section 2 beta-glucan content (1,3/1,6 linkages, immune modulation, species-specific levels)
- `Terpenoids and Triterpenoids.md` ← Section 2 terpenoid content (ganoderic acids, biological activities)
- `Hericenones and Erinacines.md` ← Section 2 (NGF stimulation, mechanisms)
- `Cordycepin.md` ← Section 2 (mechanisms, research)
- `Ergothioneine.md` ← Extract from Section 1 (primary dietary source, antioxidant role)
- `Psilocybin Research.md` ← Section 4 (clinical trials, FDA status, TRD, end-of-life, addiction, microdosing, legal status, neuroplasticity)
- `Extraction Methods.md` ← Section 5 (hot water, alcohol, dual extraction, fruiting body vs mycelium-on-grain, bioavailability, dosing)
- `Safety and Contraindications.md` ← Section 6 (drug interactions, heavy metals, allergies, quality control, poisonous look-alikes)

- [ ] **Step 3: Fold species-specific health research into species pages**

Section 3 of the health research (species-specific health research for lion's mane, reishi, turkey tail, cordyceps, chaga, maitake, shiitake, oyster) should be integrated into each species' `## Bioactive Compounds` section in their `species/` file. This means updating the species files created in Task 5.

- [ ] **Step 4: Commit**

```bash
git add health/ species/
git commit -m "content: migrate health/medicinal notes and enrich species profiles"
```

---

### Task 8: Create initial fact notes from existing research

**Files:**
- Create: 15-20 fact notes in `facts/`

- [ ] **Step 1: Extract high-confidence claims from existing content**

Scan the migrated notes for claims that are well-supported by cited research. Create a fact note for each. Priority claims to extract:

1. Lion's mane and nerve growth factor (NGF)
2. Mushrooms and vitamin D synthesis (UV exposure)
3. Turkey tail PSK/PSP and cancer research
4. Beta-glucans and immune modulation
5. Ergothioneine as primary dietary source
6. Reishi and immune modulation
7. Cordycepin mechanisms
8. Psilocybin and treatment-resistant depression
9. Psilocybin and end-of-life anxiety
10. Mushroom nutritional density (protein on dry weight basis)
11. Mycelium network nutrient transfer
12. Fungi as primary decomposers
13. Shiitake lentinan research
14. Chaga antioxidant capacity
15. Lovastatin in oyster mushrooms

- [ ] **Step 2: Write each fact note**

For each claim, create `facts/<Descriptive Name>.md` following the Fact Template:
- YAML frontmatter with `claim`, `species`, `compounds`, `confidence`, `source_type`, `last_verified`, `tags`
- 1-3 paragraph explanation with key evidence
- "Context for content creators" section with guidance on how to state the claim
- Sources section with full citations

- [ ] **Step 3: Commit**

```bash
git add facts/
git commit -m "content: create initial fact notes library from existing research"
```

---

## Phase 2: Expand Existing Domains

### Task 9: Research and create identification & foraging content

**Files:**
- Create: `identification/Spore Prints.md`
- Create: `identification/Morphological Identification.md`
- Create: `identification/Dangerous Look-Alikes.md`
- Create: `identification/Foraging Safety.md`
- Create: `identification/Habitat Guide.md`

- [ ] **Step 1: Research identification and foraging**

Use web search to gather comprehensive information on:
- Spore print technique, colors, and identification value
- Key morphological features for field ID (cap shape, gill attachment, stipe features, veil remnants, bruising reactions)
- Dangerous look-alikes for commonly foraged species (death cap / Amanita phalloides, destroying angel / Amanita bicicolor, Galerina marginata, false morels / Gyromitra, Jack O'Lantern / Omphalotus)
- Foraging safety rules, when NOT to eat a wild mushroom
- Habitat associations by ecosystem type (deciduous forest, conifer forest, grassland, urban)

- [ ] **Step 2: Write identification notes**

Create each note with full frontmatter, wiki links to species, and source citations. Prioritize safety — the Dangerous Look-Alikes note should be thorough and unambiguous.

- [ ] **Step 3: Update species pages with identification cross-links**

Add `[[Spore Prints]]`, `[[Dangerous Look-Alikes]]`, `[[Morphological Identification]]` links to each species' `## Identification` section where relevant.

- [ ] **Step 4: Commit**

```bash
git add identification/ species/
git commit -m "content: add identification and foraging knowledge"
```

---

### Task 10: Research and create culinary content

**Files:**
- Create: `culinary/Flavor Profiles.md`
- Create: `culinary/Cooking Techniques.md`
- Create: `culinary/Preservation Methods.md`
- Create: `culinary/Mushroom Pairings.md`

- [ ] **Step 1: Research culinary use**

Use web search to gather comprehensive information on:
- Flavor and texture profiles for each cultivated/foraged species (umami intensity, meatiness, nuttiness, seafood-like, etc.)
- Cooking methods: sauteing, roasting, grilling, braising, raw preparation, duxelles, powdering
- Preservation: dehydrating, freeze-drying, pickling, fermenting, freezing (blanch vs raw), canning safety
- Classic pairings: which mushrooms complement which cuisines, proteins, herbs, wines

- [ ] **Step 2: Write culinary notes**

Create each note with frontmatter, wiki links to species, and source citations. The Flavor Profiles note should have a per-species table.

- [ ] **Step 3: Update species pages with culinary sections**

Expand each species' `## Culinary` section with species-specific cooking and flavor information. Wiki-link to the culinary topic notes.

- [ ] **Step 4: Commit**

```bash
git add culinary/ species/
git commit -m "content: add culinary knowledge and expand species culinary sections"
```

---

### Task 11: Research and create industrial applications content

**Files:**
- Create: `industrial/Mycoremediation.md`
- Create: `industrial/Myco-Materials.md`
- Create: `industrial/Fungal Dyes.md`
- Create: `industrial/Biotech Applications.md`

- [ ] **Step 1: Research industrial applications**

Use web search to gather comprehensive information on:
- Mycoremediation: oil spill cleanup, heavy metal absorption, pesticide degradation, wastewater treatment, species used (Pleurotus, Trametes, Phanerochaete)
- Myco-materials: mycelium packaging (Ecovative), mycelium leather (Mylo, Reishi), building insulation, mycelium composites
- Fungal dyes: species that produce dyes (Cortinarius, Pisolithus, Phaeolus), traditional dyeing techniques, modern applications
- Biotech: enzyme production, pharmaceutical manufacturing, fermentation, mycoprotein (Quorn/Fusarium venenatum)

- [ ] **Step 2: Write industrial notes**

Create each note with full content, frontmatter, wiki links, and citations.

- [ ] **Step 3: Commit**

```bash
git add industrial/
git commit -m "content: add industrial applications knowledge"
```

---

### Task 12: Research and create history & ethnomycology content

**Files:**
- Create: `history/Ethnomycology.md`
- Create: `history/Traditional Medicine.md`
- Create: `history/Mycophobia and Mycophilia.md`
- Create: `history/Mushrooms in Art and Religion.md`

- [ ] **Step 1: Research history and cultural context**

Use web search to gather comprehensive information on:
- Ethnomycology: R. Gordon Wasson's work, traditional use across cultures (Mazatec, Siberian, Chinese, Japanese, European)
- Traditional medicine: TCM use of reishi/lingzhi, shiitake, cordyceps; Ayurvedic traditions; European folk medicine
- Mycophobia vs mycophilia: cultural attitudes toward mushrooms (Anglo-Saxon mycophobia vs Eastern European/Asian mycophilia), historical reasons
- Mushrooms in art and religion: Tassili cave paintings, medieval manuscripts, Soma/Amanita theory, María Sabina, modern psychedelic renaissance

- [ ] **Step 2: Write history notes**

Create each note with frontmatter, wiki links to species and other topics, and citations. Handle culturally sensitive topics (indigenous knowledge, sacred use) with respect and accuracy.

- [ ] **Step 3: Commit**

```bash
git add history/
git commit -m "content: add history and ethnomycology knowledge"
```

---

## Phase 3: Species Expansion

### Task 13: Research and add gourmet species (batch 1)

**Files:**
- Create: 8 new species files in `species/`

- [ ] **Step 1: Research new gourmet species**

Research the following species (web search for each — taxonomy, morphology, habitat, cultivation, culinary use, any medicinal properties):

1. Pleurotus eryngii (King Oyster / King Trumpet)
2. Stropharia rugosoannulata (Wine Cap / Garden Giant)
3. Cyclocybe aegerita (Pioppino / Black Poplar)
4. Hypsizygus tessellatus (Beech Mushroom / Shimeji)
5. Pholiota nameko (Nameko)
6. Sparassis crispa (Cauliflower Mushroom)
7. Tuber melanosporum (Black Truffle)
8. Tuber magnatum (White Truffle)

- [ ] **Step 2: Create species files**

For each species, create `species/<Scientific name>.md` following the Species Template with full frontmatter and all sections populated. Wiki-link to relevant topic notes.

- [ ] **Step 3: Commit**

```bash
git add species/
git commit -m "content: add 8 gourmet species profiles (king oyster, wine cap, truffles, etc.)"
```

---

### Task 14: Research and add poisonous/dangerous species

**Files:**
- Create: 8 new species files in `species/`

- [ ] **Step 1: Research dangerous species**

Research the following species (web search — taxonomy, morphology, habitat, toxicology, symptoms, treatment, look-alike confusion risk):

1. Amanita phalloides (Death Cap)
2. Amanita bisporigera (Destroying Angel)
3. Galerina marginata (Deadly Galerina)
4. Gyromitra esculenta (False Morel)
5. Omphalotus olearius (Jack O'Lantern)
6. Chlorophyllum molybdites (Green-Spored Parasol)
7. Cortinarius rubellus (Deadly Webcap)
8. Podostroma cornu-damae (Poison Fire Coral)

- [ ] **Step 2: Create species files**

For each species, create `species/<Scientific name>.md` with a modified template emphasizing:
- `edibility: deadly poisonous` or `edibility: poisonous` in frontmatter
- Tags: `poisonous`, `dangerous`
- `## Toxicology` section instead of `## Culinary` — covering toxins, symptoms, onset time, treatment
- `## Confusion Risk` section — which edible species it's confused with and how to distinguish
- Wiki-link to `[[Dangerous Look-Alikes]]`, `[[Foraging Safety]]`

- [ ] **Step 3: Update Dangerous Look-Alikes note**

Add entries for each new poisonous species in `identification/Dangerous Look-Alikes.md`, cross-referencing which edible species they mimic.

- [ ] **Step 4: Commit**

```bash
git add species/ identification/
git commit -m "content: add 8 poisonous species profiles with toxicology"
```

---

### Task 15: Research and add medicinal and ecologically significant species

**Files:**
- Create: 10 new species files in `species/`

- [ ] **Step 1: Research additional medicinal species**

Research (web search — taxonomy, habitat, bioactive compounds, clinical evidence, cultivation status):

1. Tremella fuciformis (Snow Fungus / Silver Ear)
2. Phellinus linteus (Meshima / Song Gen)
3. Auricularia auricula-judae (Wood Ear / Jelly Ear)
4. Wolfiporia extensa (Poria / Fu Ling)
5. Antrodia camphorata (Niu-Chang-Chih) — Taiwan endemic

- [ ] **Step 2: Research ecologically significant species**

Research (web search — taxonomy, ecology, distribution, significance):

6. Armillaria ostoyae (Honey Mushroom — largest organism on Earth)
7. Pisolithus arhizus (Dyeball — mycorrhizal pioneer)
8. Pilobolus crystallinus (Hat Thrower — spore dispersal)
9. Ophiocordyceps unilateralis (Zombie Ant Fungus)
10. Phanerochaete chrysosporium (White Rot model organism — bioremediation)

- [ ] **Step 3: Create species files**

Create all 10 species files following the Species Template. For ecologically significant species, emphasize `## Biology` and `## Ecology` sections. Wiki-link to relevant topic notes.

- [ ] **Step 4: Commit**

```bash
git add species/
git commit -m "content: add 10 medicinal and ecologically significant species profiles"
```

---

## Phase 4: Facts Library Expansion

### Task 16: Generate comprehensive fact notes

**Files:**
- Create: 30+ additional fact notes in `facts/`

- [ ] **Step 1: Extract facts from all new content**

Review all notes created in Tasks 9–15. Extract every claim that is:
- Well-supported by cited evidence
- Useful for content creation (interesting, shareable, educational)
- Not already covered by the initial 15 fact notes from Task 8

Target categories:
- Identification safety facts (5-8 facts)
- Culinary facts (5-8 facts)
- Industrial/innovation facts (5-8 facts)
- Ecology facts (5-8 facts)
- History/culture facts (5-8 facts)
- Additional health/medicinal facts from new species (5-8 facts)

- [ ] **Step 2: Write all fact notes**

Create each following the Fact Template. Ensure every fact has:
- Accurate `confidence` level
- At least one source citation
- "Context for content creators" guidance
- Appropriate tags including `#social-media-ready` where applicable

- [ ] **Step 3: Commit**

```bash
git add facts/
git commit -m "content: expand facts library with 30+ new citation-backed claims"
```

---

## Phase 5: Final Quality Pass

### Task 17: Cross-link audit and vault integrity check

- [ ] **Step 1: Check for broken wiki links**

Search all `.md` files for `[[...]]` patterns. Verify each linked note exists. Fix any broken links (typos in species names, notes that were named differently than expected).

```bash
# Find all wiki links
grep -roh '\[\[[^]]*\]\]' species/ taxonomy/ biology/ cultivation/ health/ identification/ culinary/ ecology/ industrial/ history/ facts/ | sort -u
```

Compare against actual filenames. Fix mismatches.

- [ ] **Step 2: Check frontmatter consistency**

Verify all species files have complete frontmatter (no empty required fields). Verify all fact notes have `confidence` and `source_type` set.

- [ ] **Step 3: Verify source citations**

Spot-check 10 random source citations across the vault to confirm they're real papers/sources (not hallucinated). Flag any that can't be verified for follow-up research.

- [ ] **Step 4: Commit any fixes**

```bash
git add -A
git commit -m "chore: cross-link audit and frontmatter consistency fixes"
```

---

## Summary

| Phase | Tasks | Notes Created (approx) |
|---|---|---|
| Phase 1: Scaffold + Migrate | Tasks 1–8 | ~60 notes (templates, taxonomy, biology, ecology, 19 species, cultivation, health, 15 facts) |
| Phase 2: Expand Domains | Tasks 9–12 | ~17 notes (identification, culinary, industrial, history) |
| Phase 3: Species Expansion | Tasks 13–15 | ~26 species profiles |
| Phase 4: Facts Library | Task 16 | ~30 fact notes |
| Phase 5: Quality Pass | Task 17 | Fixes only |
| **Total** | **17 tasks** | **~130+ notes** |

Each task is independently completable and produces a working commit. Tasks within a phase can be parallelized (especially Tasks 9–12 and Tasks 13–15). Cross-phase dependencies: Phase 2+ builds on Phase 1 structure. Phase 4 requires Phases 2–3 content to exist.
