# Use Case Studies — Olivia's Space Force (Hardened)

Date: 2026-10-27 (Heavy review). Concept only. None have been run.

## UC1 — Hair / wig transfer batch

**Goal:** Run a self-contained pass that consumes reference plates and a prompt matrix and emits outputs + contact sheet.  
**Dispatch:** `colab run --gpu L4 --timeout 1800 hair_pass.py` (or A100 if the script needs it). Explicit timeout required.  
**Inputs:** Drive folder with JPEGs + playbook.  
**Outputs:** Individual plates + contact sheet back to Drive (via Vultr staging if large).  
**Why this surface:** Ephemeral. No kernel state required once the script is written. Units are already paid.  
**Success metric:** Units delta logged, outputs land in the expected Drive folder, contact sheet readable, actual hardware recorded.  
**Failure mode:** Timeout or OOM. Fallback to longer timeout or smaller batch. Always pass `--timeout`.

## UC2 — Tool install and validation (MV-Adapter / FaceDetailer)

**Goal:** Prove a library installs and runs a smoke test on Colab GPU.  
**Dispatch:** Named session for the first hour (`colab new -s tooltest --gpu L4`), then switch to `colab run` once the recipe is stable. Owner + TTL required for the named session.  
**Why named first:** Iterative pip/install/debug benefits from state.  
**Exit:** Snapshot notebook + requirements to Drive, then `colab stop`. Hygiene cron as backup.

## UC3 — Parallel Pass 1 / Pass 2

**Goal:** Two independent jobs (dramatic shape vs clean pigment, or younger vs older plate) run concurrently.  
**Dispatch:** Two `colab run` invocations (preferred) or two named sessions with separate `--config` paths.  
**Merge:** Third short job or local script on Vultr produces the combined contact sheet.  
**Constraint:** Confirm concurrent behavior on this account before promising more than two. Handle `TooManyAssignmentsError`. Prefer ephemeral.

## UC4 — External agent reach (Cursor / AI Studio)

**Goal:** A non-Grok client can submit a work unit and retrieve the receipt without SSH.  
**Mechanism:** Thin MCP on the Vultr control plane exposing a safe subset of CLI commands (prefer thin wrapper over official notebook-centric MCP for headless use).  
**Guardrail:** Submit and status are open; stop is guarded (dry-run default).  
**Success:** Cursor issues a job, result appears in Drive, session count returns to zero.

## UC5 — Fallback to RunPod

**Goal:** When Colab refuses allocation or units are low, the same work unit runs elsewhere.  
**Mechanism:** Orchestrator catches the refusal or high burn, rewrites the job for RunPod (or queues it), notes the backend in the receipt.  
**Why keep RunPod:** Always-on, custom images, network volumes. Colab does not replace it.

## UC6 — Local steel handoff

**Goal:** Results that need edge processing (final compositing, local inference, cab use) are pulled to Jetson/K15.  
**Mechanism:** Drive or rsync from Vultr staging. No training pushed to local steel.  
**Success:** Checksum matches, file lands in the expected cab folder.

---

These are illustrative. None have been run under this concept. Explicit `--timeout` and prefer-`colab-run` discipline added based on verified CLI behavior.
