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
