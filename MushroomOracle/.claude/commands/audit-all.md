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
