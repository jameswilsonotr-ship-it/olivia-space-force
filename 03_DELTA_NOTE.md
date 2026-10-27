# What Heavy Changed and Why

**Date:** 2026-10-27

## Corrections

1. **Timeout is mandatory and explicit.**  
   Original plan noted “timeout quirks.” Verified against [googlecolab/google-colab-cli#183](https://github.com/googlecolab/google-colab-cli/issues/183) (opened 2026-10-24, still open). The default `--timeout 30` is a whole-run wall-clock limit. With the currently pinned `jupyter-kernel-client==0.9.0` it busy-loops a local CPU core; with 1.0.2 it fails after 30 s.  
   **Change:** Every job manifest and every use case now requires an explicit `--timeout` (e.g. 1800 or 3600). Pin the CLI version in the control-plane install notes.

2. **Orphan sessions from token expiry.**  
   Added [issue #147](https://github.com/googlecolab/google-colab-cli/issues/147). Runtime-proxy token expires ~1 h. CLI treats 401/404 as “lost” and prunes the local entry while the VM assignment stays alive, eventually causing `TooManyAssignmentsError`.  
   **Change:** Red team now lists a hygiene cron that calls `colab sessions` + forced stop of any session older than its declared TTL, plus a “prefer `colab run`” rule that avoids the problem for the majority of jobs.

3. **Drive mount is unreliable headless.**  
   [Issue #113](https://github.com/googlecolab/google-colab-cli/issues/113) shows `colab drivemount` timing out at 120 s on headless hosts even after browser consent succeeds.  
   **Change:** Architecture now prefers pulling results via `colab download` or rsync from the Vultr staging area. Drive remains the durable crate but is not the hot path for large binaries.

4. **MCP surface.**  
   Official `googlecolab/colab-mcp` requires a client that handles `notifications/tools/list_changed`; many agents do not, so tools stay invisible. Forks (SebastianGilPinzon, anluin, cool-colab-mcp) pre-register tools and restore programmatic GPU control.  
   **Change:** Concept plan lists the forks as the practical path for agent reach and notes that the official MCP is notebook-centric.

5. **Concurrent limit is not a published number.**  
   Backend returns HTTP 412 / `TooManyAssignmentsError`. Historical reports suggest 2–5 sessions depending on plan and load; Pro+ is more generous but still not unlimited.  
   **Change:** Use cases and red team now treat concurrent capacity as “check `colab sessions` before dispatch; hard stop after one refusal.”

## Additions

- Explicit job contract (owner, TTL, timeout, expected outputs, fallback ladder).
- “Bots are clients, not owners” restated as a hard rule with no raw-stop rights until Phase 5.
- Research notes file with citations.

## Unchanged

- Prefer `colab run` for anything that does not need kernel state.
- Vultr control plane owns auth and the queue.
- RunPod remains the escape hatch.
- No bot contact, no execution in this pass.
