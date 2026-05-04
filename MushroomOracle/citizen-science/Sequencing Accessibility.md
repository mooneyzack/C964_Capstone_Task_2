---
tags:
  - citizen-science
  - sequencing
  - DNA-barcoding
  - technology
last_verified: 2026-05-04
---

# Sequencing Accessibility

DNA sequencing for fungal identification has become remarkably affordable, placing molecular taxonomy within reach of dedicated amateurs and citizen science initiatives. This page covers the practical landscape: costs, services, field-deployable technology, and educational tools.

---

## Sanger Sequencing

The workhorse method for single-specimen barcoding (see [[DNA Barcoding]]).

**Commercial services**: Psomagen offers Sanger sequencing at **$3.50/reaction** ($249/full plate of 96).

**Academic core facilities**: Prices range from ~$2.00/well (Yale) to $6.75/reaction (Iowa, individual tubes), with plate submissions typically $3–4/reaction.

**Total cost per identification**: An amateur sending a dried mushroom sample for DNA extraction, PCR, and Sanger sequencing of ITS can expect **$10–25 per sample** through commercial services (including extraction and PCR).

**Trend**: Some academic institutions have discontinued Sanger services as of 2024, reflecting the shift to next-gen methods in research — but commercial services remain accessible.

---

## Oxford Nanopore MinION: Field Sequencing

The MinION has emerged as a potentially transformative tool for field mycology:

**Cost**: ~$1,000 for the starter pack (Mk1B device, sequencing kit, 2 flow cells). Per-run consumable costs (flow cells) remain non-trivial.

**Capabilities**: Palm-sized device generates **real-time results**. With the EPI2ME platform, fungal species can be identified within minutes of starting a run. Longer reads than Sanger improve phylogenetic analysis.

**Field deployments**: Used in remote pop-up labs from Arctic Canada to the Amazon. Recovered all mock community members in controlled tests.

### NAMA's 2024 Mass Barcoding Event

The NAMA DNA Sequencing Committee (DNAMA) used Oxford Nanopore at the 2024 PNW NAMA Foray at Cispus Center, Washington:

- **803 ITS barcodes** generated (83.7% success rate)
- **513 unique taxa** in 209 genera
- A landmark achievement for citizen-science-driven sequencing

### Practical Challenges

- DNA extraction, PCR, and sequencing protocols are still difficult in truly remote settings
- **Higher error rates** than Sanger, especially in homopolymeric regions (insertions/deletions)
- Massive data output creates challenges for transfer and processing where connectivity is limited
- Requires personnel trained in molecular diagnostics — "apparent ease of use" applies only with skilled operators

---

## Educational Tools

**miniPCR**: Offers a "Mushroom ID Project: Fungal DNA Barcoding Kit" for educational settings, lowering the barrier to entry for PCR and barcoding in classrooms and workshops.

---

## Reference Databases

See [[Fungal Databases]] for details on where to compare sequences:

- **UNITE** — curated ITS reference database (gold standard)
- **GenBank/NCBI** — comprehensive but 10–20% of fungal ITS sequences carry incorrect names
- **BOLD (Barcode of Life Data Systems)** — data management for the International Barcode of Life project; less well-curated for fungi than UNITE

---

## Related

- [[DNA Barcoding]] — the molecular identification methods
- [[Fungal Databases]] — where to deposit and compare sequences
- [[Citizen Science Platforms]] — community initiatives using these tools
- [[Environmental DNA]] — high-throughput sequencing for communities

## Sources

- Psomagen Sanger Sequencing pricing
- Iowa IIHG sequencing fees; Yale Research sequencing fees
- Oxford Nanopore MinION (nanoporetech.com)
- NAMA DNAMA 2024 biorxiv preprint
- Nature Scientific Reports (2023) — Nanopore fungal identification
- NAMA DNA Sequencing (namyco.org)
- miniPCR Fungal DNA Barcoding Kit
