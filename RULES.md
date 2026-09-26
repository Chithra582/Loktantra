# Rules: Loktantra Agent

These are immutable operational boundaries and safety constraints for Loktantra Agent.

## MUST ALWAYS
1. **MUST ALWAYS enforce atomic double-vote prevention**: Check the `hasVoted[voterAddress]` mapping and set it to `true` within the same transaction to guarantee that each authorized address can vote exactly once.
2. **MUST ALWAYS isolate off-chain PII from on-chain ballots**: Never publish voter legal names, phone numbers, government IDs, or IP addresses to smart contracts or public event logs.
3. **MUST ALWAYS verify election temporal boundaries**: Require that votes are submitted strictly between `startTime` and `endTime`; reject any transactions submitted before initialization or after election closure.
4. **MUST ALWAYS protect smart contract administrative keys**: Enforce multi-signature governance or time-locked access for election lifecycle management (`startElection`, `endElection`).
5. **MUST ALWAYS provide gas-optimized contract interactions**: Optimize ballot storage structures (e.g. packing candidate IDs into `uint8` or `uint16`) to minimize transaction fees for voters.

## MUST NEVER
1. **MUST NEVER store voter choice alongside personal identity in off-chain databases**: Database schemas must strictly segregate authentication verification tokens from candidate choices.
2. **MUST NEVER permit ballot retraction or retroactive alteration**: Votes committed to the blockchain are immutable; contracts must contain no administrative backdoors to alter existing tallies.
3. **MUST NEVER expose wallet private keys or mnemonic phrases**: Never prompt for, log, or transmit raw private keys; always interact via standard Web3 providers (MetaMask, WalletConnect).
4. **MUST NEVER allow reentrancy attacks in voting contracts**: Apply the Checks-Effects-Interactions pattern and OpenZeppelin `ReentrancyGuard` on all state-modifying functions.
