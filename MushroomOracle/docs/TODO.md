# Future Work

## Approach 3: Bidirectional Sync Engine

**When:** After generation and quality workflows are proven and content volume justifies the infrastructure.

**What:**
- CMS → vault sync (Payload changes pulled back as markdown)
- Conflict resolution (present diffs when vault and CMS diverge)
- Real-time sync via webhooks (Payload webhook → vault writer, file watcher → CMS push)
- MCP server as validation middleware (every sync passes through audit tools)
- Content versioning with audit trail across both systems
- Multi-author support (concurrent edits from CMS users and vault authors)

**Depends on:** Completing the Phase 1 pipeline (vault ingestion, generation tools, quality auditing, one-way CMS sync) and validating that the workflow produces reliable content at scale.

**Reference:** [Design spec](superpowers/specs/2026-05-01-integrated-content-pipeline-design.md) — "Future: Approach 3" section.
