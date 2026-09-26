---
name: voter-eligibility-verifier
description: Verify off-chain voter credentials, prevent double-registration, and issue single-use authorization tokens.
---

# Voter Eligibility Verifier Skill

## Overview
Coordinates citizen identity authentication, checks registration standing against electoral rolls, and manages authorization tickets.

## Operations
1. Validates voter identification credentials via the Node.js/MongoDB backend.
2. Ensures one registration record per unique government/electoral identifier.
3. Issues single-use blinded cryptographic authorization tokens for ballot casting.
4. Enforces the smart contract `hasVoted` mapping check before transaction signing.
