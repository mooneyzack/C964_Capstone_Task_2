---
tags:
  - citizen-science
  - molecular-taxonomy
  - DNA-barcoding
  - ITS
last_verified: 2026-05-04
---

# DNA Barcoding for Fungi

DNA barcoding — identifying species by comparing short, standardized gene regions against reference databases — has revolutionized fungal taxonomy. It has revealed that morphological identification alone dramatically underestimates true diversity, splitting long-accepted "species" into complexes of cryptic taxa and accelerating the discovery of undescribed species.

---

## The ITS Region: Universal Fungal Barcode

In 2012, the International Fungal Barcoding Consortium (Schoch et al., *PNAS*, 109:6241–6246) formally recommended the **internal transcribed spacer (ITS) region** of the nuclear ribosomal RNA gene cluster as the primary fungal barcode. The study evaluated six DNA regions across a multinational, multilaboratory effort.

**Why ITS won:**
- Over **90% PCR amplification success rate** across diverse sample conditions
- **~70% Probability of Correct Identification** (PCI)
- Broadly applicable across the kingdom
- Already in wide use — primers developed by White et al. (1990) had been standard for two decades

**Why COI (the standard animal barcode) was rejected for fungi:** Difficult to amplify, often contains large introns, and can be insufficiently variable.

The ITS region is approximately **600 base pairs** long, situated in the ribosomal tandem repeat gene cluster of the nuclear genome.

---

## Limitations of ITS

Despite its official status, ITS has significant shortcomings:

**Insufficient barcoding gap between sister taxa**: In evolutionarily young species complexes, ITS sequences may differ by only a few nucleotides — or not at all. ITS often resolves only to a species complex, not individual species.

**High intra-specific variation**: As a multi-copy region, ITS shows higher sequence variant numbers within species and a much higher chimera rate than protein-coding genes, complicating bioinformatic processing.

**Notable failures**: In the *Candida parapsilosis* complex, *Fusarium*, *Aspergillus*, *Penicillium*, and *Cladosporium*, ITS alone is insufficient for species-level identification.

Blaalid et al. (2013, *PNAS*) explicitly addressed these limitations.

---

## Supplementary Markers

A multi-locus approach is increasingly standard for serious taxonomy:

| Marker | Type | Strengths | Best For |
|--------|------|-----------|----------|
| **LSU (28S rDNA)** | Ribosomal | Higher-level classification; combined with ITS yields higher ID success | Initial placement of unknowns |
| **TEF1-α** | Protein-coding | Higher mutation rates than ITS in many groups; fewer chimera artifacts | Species-level resolution where ITS fails (e.g., *Candida parapsilosis* complex) |
| **RPB1** | Protein-coding (RNA Pol II) | Combined with ITS or LSU yields highest ID success in comparative studies | Fine-scale species delimitation |
| **RPB2** | Protein-coding (RNA Pol II) | Retrieves additional species; confirms clustering reliability | Complementary to RPB1 |

**Multi-locus concatenation** — combined alignments of ITS with one or more protein-coding genes — is effective for species-level identification in difficult groups.

TEF1-α is considered the most promising candidate for an additional universal fungal barcode marker.

---

## How DNA Barcoding Has Revolutionized Taxonomy

**Cryptic species discovery**: Multiple cryptic species found within previously described single morphological species, even in "well-studied" groups.

**Species complexes being split**: Blackening waxcaps (*Hygrocybe conica* complex), *Trametes versicolor* complex, and *Ganoderma lucidum* complex have all been revealed as multi-species aggregates through molecular work.

**Scale of undescribed diversity**: Current estimates suggest only **3–8% of fungal species have been described**. Molecular methods have dramatically accelerated the pace of discovery.

---

## Related

- [[Environmental DNA]] — metabarcoding extends barcoding to entire communities
- [[Fungal Databases]] — where sequences are deposited and curated
- [[Sequencing Accessibility]] — how affordable barcoding has become
- [[Classification Methods]] — broader taxonomic context
- [[Citizen Science Platforms]] — how amateur observations connect to molecular work

## Sources

- Schoch, C.L., et al. (2012). Nuclear ribosomal internal transcribed spacer (ITS) region as a universal DNA barcode marker for Fungi. *PNAS*, 109, 6241–6246.
- Blaalid, R., et al. (2013). Limits of ITS sequences as species barcodes for Fungi. *PNAS*.
- Stielow, J.B., et al. (2015). One fungus, which genes? Development and assessment of universal primers for potential secondary fungal DNA barcodes. *Persoonia*, 35, 242–263.
- Vu, D., et al. (2020). Large-scale generation and analysis of filamentous fungal DNA barcodes. *IMA Fungus*.
- Usman, M., et al. (2025). Supplementary barcoding markers. *Archives of Microbiology*.
