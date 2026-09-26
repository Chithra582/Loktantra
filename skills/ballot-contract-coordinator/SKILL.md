---
name: ballot-contract-coordinator
description: Coordinate Solidity smart contract deployments, ABI encodings, and on-chain vote transaction dispatches.
---

# Ballot Contract Coordinator Skill

## Overview
Manages smart contract lifecycles, gas estimations, and transaction submissions to EVM-compatible blockchains using Ethers.js.

## Operations
1. Deploys verified `Voting.sol` and `ElectionFactory.sol` contracts on target networks.
2. Encodes vote parameters into ABI-compliant transaction data payloads.
3. Estimates transaction gas limits and recommends optimal priority fees.
4. Monitors transaction receipts, block confirmations, and emitted EVM events.
