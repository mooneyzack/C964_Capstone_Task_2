---
tags:
  - citizen-science
  - databases
  - taxonomy
  - bioinformatics
last_verified: 2026-05-04
---

# Fungal Databases and Registries

The fungal database ecosystem is fragmented but functional — no single resource covers everything. Each serves a distinct purpose: nomenclature, curated sequences, raw sequences, occurrence records, or functional genomics.

---

## Nomenclatural Databases

### MycoBank

**Purpose**: Documents mycological nomenclatural novelties (new names, combinations) with associated descriptions and illustrations.

**Official status**: One of three official registries recognized by the International Nomenclature Committee for Fungi. Any new fungal name must obtain an identifier from MycoBank, Index Fungorum, or Fungal Names to be validly published.

**Strengths**: The most comprehensive search options on molecular data, supporting polyphasic approaches combining morphological, physiological, and molecular criteria. Launched by Pedro Crous (Westerdijk Fungal Biodiversity Institute).

**URL**: mycobank.org

### Index Fungorum

**Purpose**: Nomenclatural data for scientific names of fungi (including yeasts, lichens, chromistan and protozoan analogues, fossil forms) at all ranks.

**Official status**: One of three official nomenclatural repositories.

**Data synchronization**: The three repositories synchronize data monthly.

**URL**: indexfungorum.org

### Species Fungorum

**Purpose**: Global checklist of **accepted** scientific names. Where Index Fungorum tracks all published names, Species Fungorum tracks which are currently accepted.

**Use case**: Determine which name to use when multiple synonyms exist.

**URL**: speciesfungorum.org

---

## Sequence Databases

### GenBank / NCBI

**Purpose**: The nucleotide sequence archive maintained by the National Center for Biotechnology Information. Primary repository for publicly available DNA sequences of all organisms.

**How to use**: BLAST searches against GenBank are the most common first step in molecular identification (see [[DNA Barcoding]]).

**Limitation**: Sequences deposited by researchers with varying curation quality. Roughly **10–20% of fungal ITS sequences** in GenBank carry incorrect species names.

**URL**: ncbi.nlm.nih.gov

### UNITE Database

**Purpose**: Curated database centered on the eukaryotic nuclear ribosomal ITS region. The **gold standard** for curated fungal ITS sequences.

**Scale**: Version 10.0 contains **3,846,536 ITS sequences** and **239,180 fungal Species Hypotheses** with DOIs.

**Species Hypotheses (SH)**: All ITS sequences clustered to approximately species level at distance thresholds (0.5–3%), each cluster receiving a DOI for unambiguous communication. Expert-curated Reference Sequences (RefS) preferentially from type specimens.

**How to use**: Precompiled reference datasets downloadable for local BLAST and HTS pipelines (QIIME, DADA2). Essential for [[Environmental DNA|metabarcoding studies]].

**Strengths**: Addresses the GenBank curation gap. Handles dark taxa (see [[Citizen Science Platforms]]) through sequence-based classification.

**Limitation**: Focused on ITS; limited coverage of supplementary markers.

**URL**: unite.ut.ee

---

## Occurrence Databases

### GBIF (Global Biodiversity Information Facility)

**Purpose**: International network providing open access to occurrence data about all life. Currently integrates datasets documenting over **1.6 billion species occurrences**.

**Fungal data**: Combines specimen records from museums, citizen science (iNaturalist, Mushroom Observer), and digitized herbarium records.

**How to use**: Query by taxon, geography, date. The `rgbif` R package and GBIF Species API facilitate programmatic access.

**Limitation**: Quality issues including taxonomic errors, geocoordinate inaccuracies, incomplete coverage. Data cleaning required.

**URL**: gbif.org

---

## Functional Genomics

### FungiDB

**Purpose**: Data mining and functional genomics analysis for fungal and oomycete species. Part of the EuPathDB platform.

**Content**: Nearly 100 genomes plus transcriptomic, proteomic, and phenotypic datasets.

**How to use**: Web interface with embedded bioinformatics tools. Galaxy workspace for RNA-Seq, variant calling.

**Limitation**: Focused on model and pathogenic organisms; limited macrofungi coverage.

**URL**: fungidb.org

---

## Identification Resources

### MushroomExpert.com

**Purpose**: Extensively curated identification resource by Michael Kuo (English instructor at Eastern Illinois University, author of *100 Edible Mushrooms*, *Morels*, *Mushrooms of the Midwest*).

**Strengths**: Honest about identification difficulty; recommends look-alikes to check. Free. Accessible for amateurs with sufficient depth for advanced users. Updated 18+ years.

**Limitation**: North American focus (primarily Midwest/eastern). One-person operation. No API.

**URL**: mushroomexpert.com

---

## Related

- [[DNA Barcoding]] — the molecular methods that generate sequence data
- [[Environmental DNA]] — metabarcoding that relies on these databases
- [[Citizen Science Platforms]] — observation data that feeds into GBIF
- [[Classification Methods]] — taxonomic frameworks these databases encode
- [[Sequencing Accessibility]] — getting sequences into these databases

## Sources

- Nilsson, R.H., et al. (2019). The UNITE database for molecular identification of fungi. *Nucleic Acids Research*, 47, D259–D264.
- Abarenkov, K., et al. (2024). UNITE general FASTA release. *Nucleic Acids Research*, 52, D791–D797.
- PMC3379888 — NCBI fungal genome resources
- FungiDB, *Nucleic Acids Research* (2012)
- GBIF fungi page (gbif.org/species/5)
- MushroomExpert.com — About page
