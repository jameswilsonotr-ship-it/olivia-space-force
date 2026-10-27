# Red Team Appendix — Olivia's Space Force (Hardened)

Companion to `00_CONCEPT_PLAN.md`.  
Date: 2026-10-27 (Heavy review). Concept only. Mitigations not implemented.

## Critical verified issues

### Timeout default (issue #183)
- Official CLI defaults `--timeout` to 30.0 seconds for both `colab run` and `colab exec`.
- This value is passed as a wall-clock budget for the *entire* execution (not idle timeout).
- With the currently pinned `jupyter-kernel-client==0.9.0`, runs longer than 30s busy-loop a local CPU core until completion.
- With `jupyter-kernel-client>=1.0.2`, the same default raises `TimeoutError` and fails the run.
- Source: https://github.com/googlecolab/google-colab-cli/issues/183 (opened 2026-10-24, open at time of review).
- **Mitigation (not yet implemented):** Always pass an explicit `--timeout` larger than the expected job. Document in every job manifest. Pin CLI version. Track the issue.

### Token expiry and orphan sessions (related to issue #147)
- Runtime-proxy tokens can expire (~1 hour in some reports) while the backend VM assignment remains alive.
- Subsequent allocation attempts then hit `TooManyAssignmentsError` (HTTP 412).
- Named sessions left after SSH drop or agent crash produce the same outcome.
- **Mitigation:** Prefer `colab run` (auto-teardown). Named sessions require owner + hard TTL. Hygiene cron that runs `colab sessions` and stops orphans. Explicit stop on every failure path. Dedicated control-plane user with restricted config permissions.

### Concurrent allocation refusal
- No published hard concurrent-session number for Pro+.
- Backend returns `TooManyAssignmentsError` when it refuses.
- Multiple clients or retry loops can rapidly exhaust the allowance.
- **Mitigation:** Check `colab sessions` before every `new`. Prefer ephemeral `colab run`. Idempotent job IDs + claim files. Max concurrent and max jobs/hour. Fallback ladder.

## Session leak scenarios

1. Named session left after SSH drop or agent crash.  
   Mitigation: TTL cron + prefer `colab run`.

2. Agent retries on failure without checking existing sessions.  
   Mitigation: Check sessions first, exponential backoff, hard stop after N failures, idempotent job IDs.

3. Default 30s timeout on long jobs (see above).  
   Mitigation: Always pass explicit `--timeout`.

4. High-RAM or premium GPU burns units faster than expected.  
   Mitigation: Log `colab usage` before/after. Alert at 10% and 25% of remaining balance. Rates are not contractual and vary by GPU and account.

5. Auth on shared Vultr user.  
   Mitigation: Dedicated user or dedicated box for the control plane. Restrict file perms on `~/.config/colab-cli/`.

## Cost / availability scenarios

- A100 requested, only T4 available → job runs slower or fails. Fallback ladder required. Log actual hardware.
- Pro+ background execution is available (up to ~24h with units) but still subject to unit exhaustion and idle behavior. CLI sessions are activity-based on the kernel.
- Unit burn rate changes. Treat published rates as snapshots only. Log deltas.
- Drive quota or latency on large result sets. Prefer pull-to-Vultr staging then selective Drive upload. Compress contact sheets.

## Coordination scenarios

- Two MCP clients submit the same work unit.  
  Mitigation: Idempotent job IDs, claim file or Drive lock.

- Cursor and a future bot both have stop rights.  
  Mitigation: Stop is a guarded tool; dry-run default; human confirmation for bulk stop in early phases.

## What we deliberately do not automate yet

- Automatic GPU type selection without a logged fallback.
- Unattended multi-day named sessions.
- Any bot with raw shell on the control plane before Phase 5 and explicit approval.
- Official notebook-centric MCP as the primary headless path (use thin CLI wrapper instead).

## Residual risks we accept for now

- Colab can still refuse allocation. Escape hatch is RunPod or queue-and-wait.
- Official CLI is Linux/macOS only; Vultr is fine, cab Windows would need WSL or a jump host.
- Drive remains the slow but durable operational crate. Large binary pulls may need a Vultr staging step.
- Exact concurrent limit and current unit burn rates on this account are not known until measured.

*This appendix is part of the concept package. No mitigations have been implemented. Sources cited for verified issues.*
