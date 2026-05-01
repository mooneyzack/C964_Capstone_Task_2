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
