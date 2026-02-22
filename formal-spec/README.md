# Formal Specification — DevSwarm

This directory contains a formal security model of the DevSwarm protocol
written in the [Tamarin Prover](https://tamarin-prover.github.io/) security
protocol verification language.

## File

| File | Description |
|------|-------------|
| `devswarm.spthy` | Tamarin security protocol theory for the full DevSwarm swarm-coordination protocol |

## What is Tamarin?

[Tamarin Prover](https://tamarin-prover.github.io/) is an automated tool for
the symbolic verification of security protocols.  It can either prove that a
protocol satisfies a security property for all possible adversary strategies,
or find a concrete attack trace that violates the property.

## Running the Proofs

### Prerequisites

Install Tamarin Prover (requires version >= 1.6 for `restriction` keyword support):

```bash
# macOS (Homebrew)
brew install tamarin-prover/tap/tamarin-prover

# Ubuntu/Debian — pre-built binary from GitHub releases
# https://github.com/tamarin-prover/tamarin-prover/releases
```

### Prove All Lemmas (Batch Mode)

```bash
tamarin-prover formal-spec/devswarm.spthy --prove
```

### Interactive GUI

```bash
tamarin-prover interactive formal-spec/devswarm.spthy
# Then open http://127.0.0.1:3001 in your browser
```

## Protocol Summary

The DevSwarm protocol has three core primitives modelled in the theory:

| Primitive | Protocol Role | Tamarin Rules |
|-----------|--------------|---------------|
| Fork = Replication | Copies a node, registers fork relationship | `Fork_Create` |
| CI/CD = Coordination | Discovers topology, publishes manifest | `Coordinator_Discover_Fork`, `Coordinator_Update_Manifest` |
| Git = Consensus | Signed commits, upstream sync | `Node_New_Commit`, `Fork_Sync_Upstream` |

The Platform (Git forge) acts as a trusted registry that signs fork-listing
API responses (`Platform_Register_Fork`).

## Security Lemmas

| Lemma | Type | Description |
|-------|------|-------------|
| `Executable` | `exists-trace` | Sanity check: the protocol can complete all key steps |
| `Fork_Ancestry` | `all-traces` | Every fork traces back to the swarm origin via a chain of legitimate forks |
| `Sync_Integrity` | `all-traces` | Forks only sync from their declared upstream parent |
| `Platform_API_Integrity` | `all-traces` | Platform API listings reflect real, registered forks |
| `Manifest_Authenticity` | `all-traces` | Manifests list only forks verified by the Platform |
| `Swarm_Membership_Integrity` | `all-traces` | Every manifest entry is a legitimately created fork node |

## Modelling Assumptions

1. **Signing abstraction** — Git commit integrity (SHA hash chains) is
   modelled as a standard public-key signing scheme.  Each node holds a
   long-term signing key; the Platform holds its own key for API responses.

2. **Dolev-Yao adversary** — The adversary controls the network and can
   intercept, replay, and inject messages, but cannot forge signatures
   without the corresponding private key.

3. **Platform trust** — The Platform (Git forge) is assumed honest: it
   correctly records fork announcements and only issues API responses for
   forks it has registered.  Compromise of the Platform is out of scope.

4. **No key compromise** — Long-term private keys are never revealed.
   Key-compromise resilience is left for future work.

5. **Unbounded swarm** — The model allows an unbounded number of nodes and
   commits; Tamarin explores all possible traces.
