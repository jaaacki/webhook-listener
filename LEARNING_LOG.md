# Learning Log

## 2026-02-13 — Issue #16: EVENT_TTL_DAYS implementation

### What was done

Added `EVENT_TTL_DAYS` environment variable for automatic event expiration:

- Default is 0 (disabled) to maintain backward compatibility
- On startup, `pruneDiskFile()` rewrites JSONL files to remove old events
- `loadEvents()` filters events by TTL before loading into memory
- Documented in `.env.example` and `README.md`

### Design decisions

- **Startup-only cleanup**: Issue #17 will add periodic cleanup. Keeping this issue minimal avoids complexity.
- **No TTL enforcement on append**: New events are always written. Old events are only removed on startup. This keeps the append path simple and fast.
- **TTL check uses timestamp field**: Events have ISO timestamp, so we compare against `Date.now() - ttlDays * 24 * 60 * 60 * 1000`.

## 2026-02-09 — Clearing all open issues/PRs to main

### Why this design (Architect)

**Merge order: README → XSS → Memory → Docker**

- README first: zero conflict risk, warms up the pipeline
- XSS second: security fix takes priority; touches index.html which PR #7 also touches
- Memory third: depends on XSS being in place so the `esc()` function exists for new innerHTML points
- Docker last: independent files but benefits from all prior changes being stable

**Reset-and-reapply strategy over rebase**: PRs #4 and #7 both independently rewrote the entire `<script>` block (~200 lines) with incompatible architectures (WS-driven vs REST pagination). A mechanical git rebase would have produced a single massive conflict marker. Resetting each branch to main and applying clean, minimal changes was far more reliable.

### What just happened (Builder)

| PR | Issue | What landed |
|----|-------|-------------|
| #2 | #1 | README expanded with 7 inaccuracies fixed before merge |
| #4 | #3 | Minimal XSS fix: `esc()` function + applied to all innerHTML interpolation. Stripped unnecessary refactoring from original PR |
| #7 | #6 | Server-side pagination (`limit`/`offset`/`hasMore` on `/api/events`), `MAX_EVENTS_PER_NAMESPACE` cap, frontend Load More button |
| #8 | #5 | Docker Compose profiles (`--profile dev`/`--profile prod`), deleted `docker-compose.dev.yml`, added `scripts/compose` wrapper |

### What could go wrong (Critic)

- **Disk file still grows unboundedly**: `events.jsonl` is append-only. Only in-memory is capped. A future Phase 2 should add disk retention/rotation.
- **`prod` branch**: Still exists and is in sync with main as of the start of this session, but is now 4 commits behind. If it serves as a deployment branch, it needs updating.
- **No CI/tests**: All validation was manual code review. Adding tests would catch regressions.
- **`esc()` in attribute context**: The `textContent`→`innerHTML` technique doesn't guarantee quote escaping per spec. Currently safe because namespace values are operator-controlled, but worth hardening if user-controlled values ever appear in attributes.
