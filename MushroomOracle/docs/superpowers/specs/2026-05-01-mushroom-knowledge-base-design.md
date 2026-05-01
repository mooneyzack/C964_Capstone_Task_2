# MushroomOracle Knowledge Base — Design Spec

## Overview

MushroomOracle is a comprehensive, scientifically accurate mushroom knowledge base stored as an Obsidian vault in a git repository. It serves as the source of truth for a mushroom education organization that creates articles and social media posts.

The vault uses Markdown files with YAML frontmatter, `[[wiki links]]` for cross-referencing, and tags for filtering. Content is organized by topic domain, with species profiles as cross-referenced anchors. A dedicated `facts/` directory holds bite-sized, citation-backed claims optimized for content creation.

## Vault Structure

```
MushroomOracle/
├── species/                    ← One note per species (core anchors)
│   ├── Pleurotus ostreatus.md
│   ├── Hericium erinaceus.md
│   └── ...
├── taxonomy/                   ← Kingdom, phyla, classification
│   ├── Kingdom Fungi.md
│   ├── Basidiomycota.md
│   └── ...
├── biology/                    ← Life cycle, cell biology, reproduction
│   ├── Fungal Life Cycle.md
│   ├── Mycelial Networks.md
│   └── ...
├── cultivation/                ← Growing knowledge
│   ├── Substrates.md
│   ├── Sterile Technique.md
│   ├── Contamination.md
│   ├── Environmental Parameters.md
│   ├── Growing Methods.md
│   └── ...
├── health/                     ← Medicinal, nutritional, bioactive compounds
│   ├── Beta-Glucans.md
│   ├── Psilocybin Research.md
│   ├── Nutritional Profiles.md
│   └── ...
├── identification/             ← Field ID, look-alikes, foraging
│   ├── Spore Prints.md
│   ├── Dangerous Look-Alikes.md
│   ├── Foraging Safety.md
│   └── ...
├── culinary/                   ← Cooking, preservation, recipes
│   ├── Flavor Profiles.md
│   ├── Preservation Methods.md
│   ├── Cooking Techniques.md
│   └── ...
├── ecology/                    ← Ecosystem roles, mycorrhizae
│   ├── Mycorrhizal Relationships.md
│   ├── Decomposition and Carbon Cycling.md
│   ├── Wood Decay.md
│   └── ...
├── industrial/                 ← Mycoremediation, materials, dyes
│   ├── Mycoremediation.md
│   ├── Myco-Materials.md
│   ├── Fungal Dyes.md
│   └── ...
├── history/                    ← Ethnomycology, cultural significance
│   ├── Ethnomycology.md
│   ├── Traditional Medicine.md
│   └── ...
├── facts/                      ← Bite-sized, citation-backed claims for content
│   ├── Lion's Mane and Nerve Growth.md
│   ├── Mushrooms and Vitamin D.md
│   └── ...
├── templates/                  ← Obsidian templates for new notes
│   ├── Species Template.md
│   ├── Compound Template.md
│   ├── Fact Template.md
│   └── Topic Template.md
├── artifacts/                  ← Raw research (existing)
│   └── research/
│       └── 2026-05-01-mushroom-encyclopedia-deep-dive.md
└── docs/
    └── superpowers/
        └── specs/
            └── (this file)
```

## Note Formats

### Species Notes

YAML frontmatter for metadata, structured sections with wiki links:

```markdown
---
common_name: Lion's Mane
scientific_name: Hericium erinaceus
phylum: Basidiomycota
class: Agaricomycetes
order: Russulales
family: Hericiaceae
tags:
  - medicinal
  - gourmet
  - beginner-friendly
  - wood-loving
edibility: choice edible
habitat: hardwood logs and stumps
distribution: North America, Europe, Asia
last_verified: 2026-05-01
---

# Hericium erinaceus (Lion's Mane)

## Identification
Physical description, distinguishing features, look-alikes...
Links to [[Dangerous Look-Alikes]], [[Spore Prints]]

## Biology
Life cycle specifics, ecology, substrate preferences...
Links to [[Fungal Life Cycle]], [[Wood Decay]]

## Cultivation
Species-specific growing parameters, substrate, conditions...
Links to [[Substrates]], [[Environmental Parameters]]

## Bioactive Compounds
Key compounds, mechanisms, research...
Links to [[Beta-Glucans]], specific fact notes

## Culinary
Flavor, texture, cooking methods, pairings...
Links to [[Cooking Techniques]], [[Flavor Profiles]]

## Sources
- [Author, Year. Title. Journal.](URL)
```

### Fact Notes

Single-claim notes with full citation, designed for content creation:

```markdown
---
claim: "Lion's mane stimulates NGF synthesis via hericenones and erinacines"
species:
  - "[[Hericium erinaceus]]"
compounds:
  - "[[Hericenones and Erinacines]]"
confidence: high
source_type: peer-reviewed
last_verified: 2026-05-01
tags:
  - medicinal
  - neuroscience
  - social-media-ready
---

# Lion's Mane and Nerve Growth

Lion's mane (Hericium erinaceus) contains two families of compounds —
hericenones (from the fruiting body) and erinacines (from the mycelium) —
that stimulate synthesis of nerve growth factor (NGF) in vitro and in vivo.

**Key evidence:**
- Mori et al. (2009): 30 Japanese adults with mild cognitive impairment
  showed significant improvement after 16 weeks of 3g/day lion's mane
  powder vs placebo.
- Lai et al. (2013): erinacine A promoted NGF synthesis and neurite
  outgrowth in rat brain cells.

**Context for content creators:**
Safe to state as well-supported. Qualify with "research suggests" for
human cognitive claims. The NGF mechanism itself is well-established
in cell/animal models.

## Sources
- Mori K, et al. (2009). Phytother Res. 23(3):367-72.
- Lai PL, et al. (2013). J Agric Food Chem. 61(19):4551-8.
```

### Topic Notes

Standard Markdown with YAML frontmatter (tags, last_verified), wiki links to species and related topics, and source citations at bottom. No rigid template — depth scales to the topic's complexity.

## Content Scope

| Domain | Target Content | Status |
|---|---|---|
| Taxonomy | Kingdom Fungi, all major phyla, classification methods, molecular phylogenetics | ~80% from existing research |
| Biology | Life cycle, reproduction, spore types, mycelium structure, fruiting triggers, cell biology | ~80% from existing research |
| Cultivation | Substrates, sterile technique, growing methods, environmental params, contamination, harvesting, troubleshooting | ~80% from existing research |
| Health & Medicinal | Nutritional science, bioactive compounds, species-specific clinical evidence, extraction methods, safety | ~80% from existing research |
| Psilocybin | Pharmacology, clinical trials, legal status, microdosing research, therapeutic applications | ~70% from existing research |
| Ecology | Mycorrhizae, decomposition, carbon cycling, wood decay, ecosystem services, soil food web | ~50% from existing research |
| Identification | Field ID keys, morphological features, spore prints, habitat associations, dangerous look-alikes, foraging safety, regional guides | New research needed |
| Culinary | Flavor profiles by species, cooking methods, preservation (drying/freezing/pickling), pairings, nutritional cooking tips | New research needed |
| Industrial | Mycoremediation, myco-materials (packaging, leather, building), fungal dyes, biotech applications, myco-textiles | New research needed |
| History & Culture | Ethnomycology, historical use across cultures, mycophobia vs mycophilia, folklore, traditional medicine | New research needed |
| Species profiles | ~50+ species across gourmet, medicinal, psychoactive, poisonous, and ecologically significant | ~19 from existing, 30+ more needed |
| Facts | Bite-sized, citation-backed claims organized for content creation | All new — derived from above |

## Phased Population Plan

### Phase 1 — Scaffold + migrate existing research
- Create vault structure (directories, templates)
- Break the existing 2,931-line encyclopedia into individual notes
- Add frontmatter, wiki links, and tags to each note
- Result: ~30% populated

### Phase 2 — Expand existing domains
- Deepen 19 existing species profiles (add ID, culinary, ecology sections)
- Fill gaps in ecology, cultivation troubleshooting, compound profiles
- Generate initial fact notes from existing research

### Phase 3 — New domain research
- Identification & foraging
- Culinary use
- Industrial applications
- History & ethnomycology
- Each domain gets its own research pass, then is broken into notes

### Phase 4 — Species expansion
- Research and add ~30+ more species:
  - More gourmet (king trumpet, wine cap, chicken of the woods, etc.)
  - Poisonous/dangerous (death cap, destroying angel, Galerina, false morels)
  - Ecologically significant (Armillaria, Pilobolus, etc.)
  - Lesser-known medicinals (meshima, tremella, etc.)

### Phase 5 — Facts library
- Extract key claims from all notes into standalone fact notes
- Tag for content use (`#social-media-ready`, `#article-ready`, `#needs-qualification`)
- Each fact gets confidence level and source citation

## Quality Standards

### Source attribution
- Every factual claim links to at least one source
- Peer-reviewed research preferred; reputable institutions (USDA, NIH, university extension services) as secondary
- Popular articles and forums are not sources (but can point to sources)

### Confidence levels (fact note frontmatter)
- `high` — peer-reviewed evidence, multiple studies, scientific consensus
- `medium` — single study, reputable but limited evidence, established traditional use
- `low` — preliminary research, anecdotal, or conflicting evidence
- Every fact note states what qualifiers content creators should use ("research suggests" vs "studies show" vs "is well established")

### Currency
- `last_verified` date in frontmatter on species and fact notes
- Claims based on clinical trials note the trial phase and date
- Legal status notes (especially psilocybin) flag that laws change rapidly

### No unsupported health claims
- No "mushroom X cures disease Y" without clinical evidence
- Clear distinction between in vitro, animal model, and human trial results
- Supplement industry marketing claims are flagged, not repeated
