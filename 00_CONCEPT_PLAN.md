# Olivia's Space Force — Hardened Concept Plan

**Status:** Concept only — Bunny + Olivia discussion. Not authorized.  
**Date:** 2026-10-27 (Heavy review)  
**Intent:** Turn idle Colab Pro+ compute units into an agent-driven, multi-session GPU pool coordinated from a Vultr VPS, with results staged on the VPS then handed to Drive or local steel.

## 1. Why Heavy Helped

Heavy ran parallel checks against current CLI docs, open issues, and MCP forks. The synthesis below incorporates those checks. Single-pass mode would have locked onto “call `colab run`”; the cross-check surfaced the timeout, orphan-session, and Drive-mount problems before any code was written.

## 2. Core Idea (spirit preserved)

We have:

- Colab Pro+ with a large idle unit balance and access to T4 / L4 / G4 / A100 / H100 / TPU.
- Official `google-colab-cli` (agent-ready).
- MCP forks that wrap the CLI for headless, multi-session use.
- Vultr VPS fleet (Aider box or sibling) that can host the control plane.
- Drive as the durable crate and local steel as the eventual edge.

The control plane on one Vultr box:

1. Owns Colab CLI auth and session state.
2. Accepts work units via a simple queue (local JSON/SQLite or a Drive claim file).
3. Dispatches them as `colab run` (preferred) or short-lived named sessions.
4. Stages results on the VPS, then pushes to Drive or local steel.
5. Exposes a safe MCP surface so Cursor, Google AI Studio, or a later bot can submit jobs as clients.

Bots are clients of the plane, not owners of the box. That is the deliberate difference from the earlier “one bot owns one machine” experiment.

## 3. Surfaces (updated)

| Surface | Best for | Limits | Notes |
|---------|----------|--------|-------|
| `google-colab-cli` | Ephemeral jobs, short named sessions | Default `--timeout 30` is a whole-run limit (#183). Token expiry can orphan VMs (#147). Concurrent limit surfaces as `TooManyAssignmentsError`. | Prefer `colab run`. Always pass `--timeout`. Pin CLI version. |
| Official `colab-mcp` | Notebook editing when a browser tab is acceptable | Needs `tools/list_changed`. | Notebook-centric. |
| CLI-wrapping forks (SebastianGilPinzon, anluin, amgallego8, cool-colab-mcp) | Headless agent reach, programmatic GPU, persistent notebooks | Smaller communities, platform variance. | Practical path for external agents. |
| Vultr control plane | Auth, queue, hygiene cron, MCP host | No GPU of its own. | Install CLI here. Restrict `~/.config/colab-cli/`. |
| Drive | Durable crate | Latency, quota, headless mount issues (#113). | Cold storage and handoff, not the hot path for large binaries. |
| RunPod | Always-on, custom images, long services | Paid per second. | Escape hatch. Do not replace yet. |
| Local steel | Edge inference, final assembly | Limited VRAM. | Pull results; do not push heavy work. |

## 4. Job Contract (new)

Every work unit carries:

- Owner (human or agent id).
- TTL (max wall time).
- Explicit `--timeout` ≥ TTL.
- Expected outputs (paths + checksums).
- Fallback ladder (T4 → L4 → A100 → RunPod queue).
- Idempotency key.

## 5. Phased Plan (concept)

**Phase 0 — Human-only smoke.**  
Install CLI on Vultr, run three `colab run --timeout 300` CPU jobs, log `colab usage` before/after. No agents.

**Phase 1 — Hair/wig batch (UC1).**  
`colab run --gpu L4 --timeout 1800` on a self-contained script. Results stage on Vultr, then to Drive. Human watches.

**Phase 2 — Named session for tool install (UC2).**  
`colab new -s tooltest --gpu L4`, iterate, then `colab stop`. Snapshot requirements.

**Phase 3 — Parallel passes (UC3).**  
Two concurrent `colab run` jobs. Confirm concurrent behavior on this account.

**Phase 4 — External agent reach (UC4).**  
Thin MCP on the control plane. Submit and status open; stop guarded.

**Phase 5 — Optional bot client.**  
Only after explicit approval. Bot talks to the plane; plane owns the CLI.

## 6. Hard Rules

- Prefer `colab run`.
- Always pass `--timeout`.
- Named sessions have an owner and a TTL; hygiene cron enforces it.
- No bot has raw stop rights until Phase 5.
- First ten jobs are human-supervised.
- `colab usage` logged before and after every batch.
- RunPod stays as the escape hatch.

## 7. Open Questions

- Exact concurrent limit on this Pro+ account (measure in Phase 0).
- Whether token-refresh path in newer CLI builds closes #147.
- Cost of staging large image sets on Vultr vs. direct Drive upload after the job.

*No sessions have been started. No credentials have been touched.*
