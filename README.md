# Olivia's Space Force

**Concept only.** Bunny + Olivia discussion. Not authorized. Nothing executed. No bots contacted.

Hardened review of the Colab cyberforce concept: using idle Colab Pro+ compute units via the official `google-colab-cli` (and optional thin MCP wrappers) from a thin control plane on a Vultr VPS. Prefer `colab run`. Named sessions require owner + TTL. Bots are clients, not owners.

## Contents

- `00_CONCEPT_PLAN.md` — Hardened architecture, surfaces (with corrections), phased plan, open questions.
- `01_RED_TEAM.md` — Failure modes with verified issues (timeout #183, orphan sessions, TooManyAssignmentsError) and mitigations (not implemented).
- `02_USE_CASES.md` — Concrete job shapes with explicit-timeout and prefer-ephemeral discipline.
- `03_DELTA_NOTE.md` — What the Heavy review changed and why, with sources.
- `04_RESEARCH_NOTES.md` — Verified CLI limits, issues, and MCP status.

## Why this exists

Colab Pro+ balance is large and idle. Official CLI makes agent-driven use realistic, but the default timeout is a known open issue and official MCP is notebook-centric. Prior pattern of parking one bot on one Vultr machine did not stick. This concept keeps the control plane as CLI + queue on a Vultr box, treats bots as clients, and uses GitHub for the plan itself while Drive remains the operational crate.

## Grok Heavy

Used for the multi-agent cross-check that verified claims against current docs and issues, then produced the hardened package. Single-pass is sufficient for “how to call the CLI.” Heavy is useful for “what breaks when we try to run this as a small cyber force.”

## Next

1. Review in sandbox and on GitHub.
2. Ask Vesper for comment.
3. Only after explicit approval, discuss with Olette as a concept, not a directive.

No bots have been contacted. No sessions have been started. No credentials have been touched.

---

*Errors are data. Sources cited where claims were corrected.*
