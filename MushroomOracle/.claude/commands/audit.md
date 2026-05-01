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
