# Devswarm x AutoResearch: Concept RFC

Status: Draft (concept only, no implementation)

## 1. Overview

This document proposes a concept-level integration between:

- `devswarm` (fork-native replication and CI-based coordination), and
- `autoresearch`-style autonomous model improvement loops.

The goal is to treat each swarm fork as an experiment worker, while the origin repository acts as a coordinator that accepts only reproducible improvements.

This RFC describes architecture, constraints, and rollout strategy. It does not include code or workflow implementation.

## 2. Problem Statement

Single-repository autoresearch loops can improve quickly, but they have limits:

- search is constrained by one machine and one hardware profile,
- local noise can produce false wins,
- reproducibility checks are often manual or delayed,
- scaling experimentation usually requires separate orchestration infrastructure.

`devswarm` already provides distributed replication and periodic coordination through standard Git and CI/CD primitives. The opportunity is to use that topology as a distributed experiment network.

## 3. Goals

- Enable many independent experiment trajectories via fork topology.
- Preserve fixed-budget autonomous loop behavior per node.
- Aggregate candidate results centrally with a clear audit trail.
- Promote only replay-verified winners.
- Keep orchestration feasible in GitHub Actions-only environments.

## 4. Non-Goals

- Building a generic distributed training platform.
- Removing all trust assumptions between forks.
- Supporting unrestricted code mutation during experiments.
- Guaranteeing score comparability across different hardware lanes.

## 5. Core Model

### 5.1 Roles

- Worker node (fork): runs one bounded experiment per cycle and submits a normalized result record.
- Coordinator (origin): validates submissions, ranks candidates, and opens promotion paths.
- Replay verifier (origin-controlled): re-runs candidate changes before merge.

### 5.2 Lifecycle

1. Worker mutates allowed training surface.
2. Worker runs fixed-budget train/eval.
3. Worker records metrics plus provenance.
4. Coordinator compares against current lane champion.
5. If improvement exceeds threshold, candidate enters replay queue.
6. Replay gate accepts or rejects.
7. Accepted candidates are promoted via normal Git merge process.

## 6. GitHub Actions-Only Feasibility

This model is feasible with GitHub Actions as the orchestration layer.

Important constraints:

- Fork permissions and token scope differ from origin repository permissions.
- Cross-fork artifact collection can be fragile depending on visibility and auth.
- Untrusted fork code must not run with privileged secrets.

Recommended event transport from forks to origin:

- Preferred: PR inbox pattern (fork submits JSON record as a file-based PR).
- Alternative: `repository_dispatch` with tightly scoped token distribution.
- Avoid relying exclusively on cross-repo artifact reads as the primary channel.

## 7. Hardware Lanes and Practicality

Scores are lane-scoped.

- `cpu` lane: easiest to operate on GitHub-hosted runners; lower search throughput.
- `nvidia` lane: practical with self-hosted GPU runners; highest throughput for sustained search.
- `apple-mlx` lane: practical with self-hosted Apple Silicon runners for MLX-specific behavior.

CPU-only operation is practical for MVP orchestration validation and smaller experiments, but likely insufficient for high-velocity model search.

## 8. Trust, Safety, and Governance

### 8.1 Policy Controls

- Restrict mutation to explicit allowlist paths (for example, `train.py`).
- Require fixed or bounded experiment budget.
- Reject submissions that alter disallowed files.

### 8.2 Provenance Requirements

Each submission should include:

- repository and run identity,
- base and candidate commit references,
- metric values and timing,
- workflow/runtime metadata.

### 8.3 Replay Gate

- Candidate promotion requires independent replays.
- Replay outcomes are required status checks on candidate PRs.
- Failed replay results block merge and mark candidate as rejected.

## 9. Data Contracts (Conceptual)

### 9.1 Node Manifest Extension

Extend `.swarm/manifest.json` with `research_topology` fields:

- `enabled`, `node_role`, `hardware_lane`,
- `last_run_at`, `trial_budget_seconds`,
- `best_local`, `trust_score`.

### 9.2 Trial Result Record

Normalized trial record should include:

- schema version,
- node identity,
- experiment identity (base/candidate refs),
- metrics (`val_bpb` or lane-defined primary score),
- provenance metadata.

## 10. Candidate Policy (Conceptual Defaults)

- Candidate threshold (`epsilon`) required relative improvement over lane champion.
- Replay count (`N`) greater than 1 before promotion.
- Submission freshness limit (ignore stale records).
- Daily candidate cap to reduce PR churn.

## 11. Risks and Mitigations

- Noisy wins:
  - Mitigation: replay gate, conservative epsilon, lane-specific baselines.
- Malicious or malformed submissions:
  - Mitigation: schema validation, allowlist enforcement, provenance checks.
- CI cost escalation:
  - Mitigation: bounded budgets, schedule throttling, candidate caps.
- Fork inactivity or schedule drift:
  - Mitigation: coordinator health tracking and stale-node pruning.
- Merge churn:
  - Mitigation: batch promotions and strict replay queueing.

## 12. Rollout Plan

Phase 1: Documentation and schema agreement.

Phase 2: Worker submission flow without auto-promotion.

Phase 3: Coordinator ranking and candidate PR generation.

Phase 4: Replay-gated promotion with required checks.

Phase 5: Lane-aware optimization and trust scoring refinement.

## 13. Success Criteria

- Reliable end-to-end candidate lifecycle with auditable history.
- Low false-positive promotion rate after replay checks.
- Measurable lane-local champion improvement over time.
- Stable operation under fork churn and heterogeneous hardware.

## 14. Open Questions

- What default epsilon is robust per lane?
- What replay count balances confidence and CI cost?
- Should promotions be immediate or batched on schedule?
- What trust score policy is transparent and abuse-resistant?
- What is the minimum viable metadata set for reproducibility?

---

This RFC intentionally stays implementation-agnostic. If accepted, the next document should be an implementation spec with concrete workflow files, JSON schemas, and branch/label policies.
