# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **Loktantra Agent** (`loktantra-agent`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** Loktantra Agent (`loktantra-agent`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Governance / Decentralized Voting & Smart Contract Auditing  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), GDPR, SOC2  

---

## How the Agent Decides

Loktantra Agent is an autonomous decentralized governance, cryptographic voter verification, and smart contract audit intelligence agent designed for **Loktantra**, a decentralized digital voting platform. The agent coordinates off-chain identity verification, on-chain ballot casting across EVM-compatible blockchains (Ethereum, Sepolia, Polygon), atomic double-vote prevention, and immutable election tally reconciliation.

### 1. Decision Architecture

The voter verification, ballot dispatch, and election audit process operates across a deterministic, five-stage pipeline:

```
Citizen Action (Connect Wallet / Complete Verification / Cast Ballot / View Live Results)
    │
    ▼
[Stage 1: Off-Chain Voter Identity & KYC Intake]
    │  - Authenticates citizen credentials via Node.js/Express backend
    │  - Verifies registration eligibility against voter electoral registry
    │  - Issues single-use cryptographic authorization ticket without linking to voter identity
    ▼
[Stage 2: Blinded Ballot Token & Payload Construction]
    │  - Client generates vote payload with target candidate ID ($C_{\text{id}}$) and election ID ($E_{\text{id}}$)
    │  - Connects to Web3 provider (MetaMask / Ethers.js)
    │  - Signs transaction hash using citizen's private wallet key
    ▼
[Stage 3: Smart Contract Double-Voting Guard & Transaction Submission]
    │  - Solidity contract receives transaction at `castVote(electionId, candidateId)`:
    │      ├── Gate 1: `require(block.timestamp >= startTime && block.timestamp <= endTime)`
    │      ├── Gate 2: `require(!hasVoted[msg.sender][electionId], "Double-vote detected")`
    │      └── Gate 3: `require(candidateId < candidateCount[electionId], "Invalid candidate")`
    │  - Atomically marks `hasVoted[msg.sender][electionId] = true`
    │  - Increments candidate vote count: `candidates[electionId][candidateId].voteCount++`
    ▼
[Stage 4: On-Chain Event Ingestion & State Update]
    │  - Emits immutable EVM event: `emit VoteCast(electionId, candidateId, block.timestamp)`
    │  - Event stream ingested by Next.js frontend listener via WebSocket/RPC subscription
    │  - Updates local state and confirms transaction inclusion in target block
    ▼
[Stage 5: Immutable Tally Verification & Public Dashboard Rendering]
    │  - Queries candidate vote counts directly from on-chain storage mappings
    │  - Verifies mathematical parity between sum of candidate votes and total registered ballot events
    │  - Renders live, transparent election charts on Next.js public dashboard
    ▼
Public Blockchain Ledger & Citizen-Facing Audit Dashboard
```

### 2. Voter Eligibility & State Machine Lifecycle

Elections in Loktantra progress through four rigid, chronological state boundaries enforced in Solidity:

1. **`SETUP`**: Election created, candidates registered by Election Admin, voting window parameters defined. No votes can be cast.
2. **`ACTIVE`**: Current block timestamp falls between `startTime` and `endTime`. Eligible wallets with authorization tokens can execute `castVote()`.
3. **`TALLYING`**: `block.timestamp > endTime`. Voting transactions are automatically reverted by contract require checks. Final vote counts are locked.
4. **`AUDITED`**: Public tally verification complete; immutable election result hash published.

### 3. Cryptographic Election Integrity Scoring Formula

The agent audits the integrity of an election batch using an Election Integrity Score $I_{\text{election}} \in [0.0, 1.0]$:

$$I_{\text{election}} = 0.40 \cdot P_{\text{parity}} + 0.35 \cdot V_{\text{double}} + 0.25 \cdot T_{\text{temporal}}$$

- **Tally Parity ($P_{\text{parity}}$)**: $1.0$ if $\sum \text{CandidateVotes} \equiv \text{TotalVotesLogged}$; $0.0$ if any discrepancy occurs.
- **Double-Vote Defense ($V_{\text{double}}$)**: Ratio of successfully rejected duplicate transaction attempts over total attempted violations ($1.0$ indicates zero double-voting leaks).
- **Temporal Validity ($T_{\text{temporal}}$)**: Percentage of on-chain vote timestamps confirmed within the legal interval $[T_{\text{start}}, T_{\text{end}}]$.

Elections are certified as fully audited only when $I_{\text{election}} \equiv 1.00$.

### 4. Thresholding & Refusal Decision Criteria

Loktantra Agent enforces strict mathematical and operational refusal boundaries:
- **Double-Vote Refusal**: Any wallet address that has already recorded `hasVoted[msg.sender] == true` is instantly reverted at the EVM opcode level with error string `"Double-vote detected: Address has already cast a ballot."`
- **Out-of-Window Vote Refusal**: Transactions submitted before `startTime` or after `endTime` are reverted immediately, preventing early tampering or late ballot stuffing.
- **Unregistered Candidate Refusal**: Votes specifying a candidate ID outside the registered array index bounds ($C_{\text{id}} \ge \text{candidateCount}$) are rejected.
- **Contract State Mutation Refusal**: No administrative functions (e.g. adding candidates, modifying start dates) can be executed once an election transitions to the `ACTIVE` state.

### 5. Fallback Decision Mechanism

Loktantra Agent incorporates fault-tolerant fallbacks to guarantee continuous election availability:
- **Multi-RPC Failover Cascade**: If the primary Ethereum/Polygon RPC provider (e.g., Infura) encounters downtime or rate limits, the client automatically switches to backup RPC endpoints (Alchemy, QuickNode, public fallback nodes) without interrupting the voter's session.
- **Offline Transaction Staging**: If the citizen's browser temporarily disconnects while preparing a ballot, the signed transaction payload is held in local client memory and submitted upon network re-establishment.
- **Model Fallback Cascade**: When interacting with the AI governance copilot, requests default to `gemini-2.0-flash` with automatic failover to `gpt-4o` and `claude-3-5-sonnet`.

### 6. Human-in-the-Loop Governance

Loktantra enforces complete democratic transparency and multi-stakeholder governance:
- **Multi-Signature Election Administration**: Administrative controls (election creation, candidate whitelisting) require multi-sig approval from designated election commissioners before contract deployment.
- **Public Audit Sovereignty**: Every citizen, political candidate, and independent auditor can inspect the verified smart contract bytecode on Etherscan and run independent node queries to verify tallies.
- **Voter Ballot Sovereignty**: Voters retain complete private autonomy over their ballot choice. The agent never prompts, influences, or modifies the user's candidate selection.

---

## The Data It Uses

Loktantra Agent operates under strict privacy and cryptographic data minimization principles.

### 1. Ingested Input Data

The agent processes only data essential for election management and ballot casting:
- **Public Ethereum Wallet Address**: 42-character hexadecimal address (`0x...`) used as the on-chain voter identity token.
- **Election and Candidate Identifiers**: Numeric integer IDs representing the active election and selected candidate choice.
- **Cryptographic Signatures**: Web3 signature payloads confirming voter authorization without disclosing private keys.
- **Transaction Metadata**: Gas price, gas limit, and block timestamp returned by the EVM network.

### 2. Configuration & Reference Data

- **Smart Contract ABI**: Application Binary Interface definitions for `Voting.sol` and `ElectionFactory.sol`.
- **Contract Deployment Addresses**: Verified on-chain contract addresses deployed on Sepolia Testnet, Polygon, or Ethereum Mainnet.
- **Network RPC Configurations**: Chain IDs, explorer URLs, and WebSocket subscription endpoints.

### 3. Base Model & Inference Lineage

- **Deterministic EVM Blockchain Core**: Smart contracts executed deterministically by Ethereum Virtual Machine (EVM) nodes, guaranteeing tamper-proof state transitions without AI hallucination.
- **AI Governance Copilot**: Frontier foundation models (Google Gemini `gemini-2.0-flash`, OpenAI `gpt-4o`, Anthropic `claude-3-5-sonnet`) utilized strictly for voter education, platform documentation, and analytics summaries.
- **Zero Training on Voting Records**: Citizen transaction logs and voting patterns are never transmitted or used for model training.

### 4. Data Privacy, Storage, and Retention

- **Strict Identity-Ballot Separation**: The backend voter verification database stores authentication credentials independently of on-chain addresses. There exists zero relational mapping linking a citizen's real name to the specific candidate choice recorded on the blockchain.
- **Zero PII on Public Ledgers**: Government IDs, citizen names, mobile numbers, and residential addresses are never stored in smart contracts or emitted in blockchain events.
- **GDPR & SOC2 Compliance**: By implementing privacy-by-design anonymization and decoupling identities from ballot records, the platform satisfies GDPR (Article 5 data minimization, Article 17 erasure of off-chain auth records) and SOC2 security standards.

---

## Limitations

Understanding the operational boundaries and technical constraints of Loktantra Agent is essential for realistic deployment.

### 1. Sybil Attacks & Reliance on Off-Chain Identity Verification
- **Limitation**: Blockchains alone cannot verify that one human controls only one wallet address without an off-chain identity verification or proof-of-humanity protocol.
- **Mitigation**: Loktantra couples on-chain voting with a verified off-chain KYC gate, issuing single-use authorization hashes to prevent multiple wallet generation by the same individual.

### 2. Blockchain Gas Volatility & Network Congestion
- **Limitation**: During periods of high network congestion, Ethereum mainnet gas fees can spike, creating financial barriers for individual voters.
- **Mitigation**: The system is engineered for low-cost Layer-2 rollups (Polygon, Arbitrum) and incorporates gas-station sponsor contracts (ERC-2771 meta-transactions) enabling gasless voting for citizens.

### 3. Private Key Custody & Wallet Usability
- **Limitation**: Non-technical citizens can lose access to seed phrases or fall victim to phishing attacks targeting self-custody wallets.
- **Mitigation**: The client UI integrates smart account abstraction (ERC-4337) and simple social login recovery to simplify the onboarding experience for non-crypto natives.

### 4. Smart Contract Immutability vs Post-Deployment Fixes
- **Limitation**: Deployed smart contracts cannot be modified if unforeseen logic flaws or edge cases emerge mid-election.
- **Mitigation**: All contracts undergo formal verification and automated security test suites (Hardhat/Foundry) prior to deployment, alongside transparent circuit-breaker pause mechanisms controlled by multi-sig governance.

### 5. Legal & Statutory Scope Boundary
- **Limitation**: Loktantra is a decentralized technological voting platform; it does not replace constitutional statutory election laws without formal legal codification by government election commissions.
- **Mitigation**: The platform is designed as an auditable digital voting blueprint for student body elections, DAO governance, corporate shareholder votes, and civic pilot studies.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Voter eligibility & state machine lifecycle | Section 2 | Verified |
| - Cryptographic election integrity formula | Section 3 | Verified |
| - Thresholding & refusal decision criteria | Section 4 | Verified |
| - Fallback decision mechanism | Section 5 | Verified |
| - Human-in-the-loop governance & multi-sig | Section 6 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested wallet addresses & transaction payloads | Section 1 | Verified |
| - Configuration, ABI & deployment data | Section 2 | Verified |
| - Base model lineage & deterministic EVM core | Section 3 | Verified |
| - Data privacy, zero-PII ledger & GDPR/SOC2 | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Sybil attack bounds & off-chain KYC reliance | Section 1 | Verified |
| - Gas volatility & network congestion | Section 2 | Verified |
| - Private key custody & wallet usability | Section 3 | Verified |
| - Immutability & post-deployment constraints | Section 4 | Verified |
| - Statutory scope & civic pilot boundary | Section 5 | Verified |
