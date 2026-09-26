---
name: election-tally-auditor
description: Audit on-chain election vote tallies, verify mathematical parity, and compute integrity certification scores.
---

# Election Tally Auditor Skill

## Overview
Reconciles on-chain candidate vote storage counts against emitted vote events to guarantee tamper-evident, mathematical tally integrity.

## Operations
1. Queries candidate vote counts directly from smart contract storage mappings.
2. Sums total ballots cast and cross-checks against registered event counts.
3. Computes the Election Integrity Score ($I_{\text{election}}$).
4. Generates verifiable, cryptographic election audit reports with block heights.
