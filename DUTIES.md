# Duties & Role Segregation: Loktantra Agent

To deliver tamper-proof decentralized voting while maintaining absolute ballot privacy, Loktantra Agent segregates duties across four dedicated roles.

## 1. Election Registrar (`maker`)
- Initializes election instances with candidate profiles, official election IDs, and defined start/end timestamps.
- Authorizes voter eligibility by verifying off-chain KYC credentials without logging voting preferences.
- Deploys and initializes verified Solidity ballot contracts on target EVM networks.

## 2. Ballot Dispatcher (`executor`)
- Encodes candidate selection data into ABI-compliant transaction payloads.
- Submits vote transactions to the blockchain via Ethers.js and user-connected Web3 wallets.
- Dispatches real-time transaction confirmation receipts with public blockchain explorer links.

## 3. Smart Contract Auditor (`checker`)
- Inspects contract bytecode and verifies `hasVoted` state mapping logic to prevent double-voting.
- Audits election temporal gates to ensure votes fall strictly within active voting windows.
- Re-calculates and verifies mathematical vote tallies by querying on-chain `Candidate` vote counts.

## 4. Privacy & Compliance Officer (`auditor`)
- Enforces strict decoupling between voter identification records and on-chain wallet addresses.
- Audits system logs to ensure zero voter PII leaves the local environment or leaks into event telemetry.
- Validates system adherence to GDPR (Article 5 & 17) and SOC2 confidentiality criteria.
