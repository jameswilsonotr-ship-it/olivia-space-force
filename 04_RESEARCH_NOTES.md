# Research Notes — Verified Claims

**Date:** 2026-10-27

## CLI Timeout

- Source: [googlecolab/google-colab-cli#183](https://github.com/googlecolab/google-colab-cli/issues/183) (opened 2026-10-24).
- Default `--timeout 30.0` is a whole-run wall-clock budget passed to jupyter-kernel-client.
- With pinned 0.9.0: longer runs busy-loop a local CPU core until completion.
- With 1.0.2: same default raises TimeoutError and stops the session.
- Mitigation: always pass explicit `--timeout`. Pin CLI version.

## Token Expiry / Orphans

- Source: [issue #147](https://github.com/googlecolab/google-colab-cli/issues/147).
- Runtime-proxy token expires ~1 h. CLI can prune local entry while VM assignment stays alive → TooManyAssignmentsError (HTTP 412).
- Mitigation: prefer `colab run`; hygiene cron stops unresponsive sessions, not only old ones.

## Drive Mount

- Source: [issue #113](https://github.com/googlecolab/google-colab-cli/issues/113).
- `colab drivemount` times out at ~120 s on headless hosts even after successful browser auth.
- Prefer rsync/scp or pull-to-Vultr for result handoff.

## MCP

- Official: googlecolab/colab-mcp — browser-tab-centric, requires `tools/list_changed`. Many clients ignore the notification → tools invisible.
- Forks (SebastianGilPinzon/colab-mcp, anluin/colab-mcp, etc.) pre-register tools and restore programmatic GPU control. Prefer a fork or thin local wrapper for headless control plane.

## Concurrent / Cost

- No published hard concurrent-session number. Backend returns 412 TooManyAssignmentsError.
- A100/H100 require paid plan + units and are availability-dependent. Treat published burn rates as snapshots. Log `colab usage` before/after.
- RunPod remains the paid escape hatch for always-on or custom-image work.

*Research only. No credentials used. No runtimes provisioned.*
