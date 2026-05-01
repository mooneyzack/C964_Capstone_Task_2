---
description: Sync vault content to ShroomSpy Payload CMS (dry-run first)
---

Sync vault species pages to ShroomSpy's Payload CMS:

1. Run `pnpm sync:cms --dry-run` in the mushroom-oracle directory to preview changes
2. Show me what would be created/updated/skipped
3. Ask for confirmation before proceeding
4. If confirmed, run `pnpm sync:cms` to push changes
5. Report results: pages synced, pages skipped (validation errors), any failures
