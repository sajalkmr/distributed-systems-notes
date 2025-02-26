# Lecture 19: Bitcoin

## Introduction

Bitcoin is a decentralized digital currency that operates without a central authority. This lecture explores its design, transaction validation, consensus mechanism, and security considerations.

## Public Ledger and Decentralization

Bitcoin requires a global public ledger to record all transactions.

This is similar to certificate transparency, ensuring that everyone agrees on the transactions and their order.

A simple solution is to have a trusted entity maintain the ledger, but this was rejected because:

- No single entity is universally trusted.
- The system needs to be resilient to corruption and failure.
- A decentralized approach is preferred.

## Peer-to-Peer Network

Instead of a central authority, Bitcoin operates on a peer-to-peer network:

- Thousands of nodes (peers) maintain the ledger.
- Transactions are broadcasted across the network.
- Each peer maintains a complete copy of the transaction log.
- The goal is consensus: all honest peers should have identical transaction logs.

## Consensus Mechanism Challenges

Simple voting mechanisms are unreliable due to:

- Uncertainty about the number of peers.
- Risk of Sybil attacks (one entity controlling multiple nodes).
- Inability to prevent fake identities from influencing votes.

## Blockchain and Proof-of-Work

### Blockchain Structure

Bitcoin transactions are recorded in blocks.

Each block contains:

- Hash of the previous block (ensuring immutability).
- List of transactions.
- Nonce (used in proof-of-work computation).
- Timestamp.

### Mining and Proof-of-Work

**Mining:** The process of creating a new block.

**Proof-of-Work:**

- Miners must find a nonce that, when hashed with the block data, produces a hash with a required number of leading zeros.
- This ensures computational effort and prevents spam or Sybil attacks.
- Takes approximately 10 minutes per block.
- Difficulty adjusts dynamically to maintain the block creation rate.
- Once a miner finds a valid block, it is broadcasted and added to the blockchain.

## Handling Forks

A fork occurs when two miners create competing blocks.

**Rules to resolve forks:**

- Peers always mine on the longest chain.
- Transactions on abandoned forks become void unless reissued.

This mechanism ensures eventual consistency and prevents double-spending.

## Security and Attack Resistance

### Double Spending

- If an attacker tries to double spend, forks might temporarily allow it.
- To mitigate risk, merchants should wait for multiple confirmations (typically 6 blocks).

### 51% Attack

If an attacker controls more than 50% of the network’s mining power, they can:

- Rewrite history and double spend transactions.
- Potentially disrupt the network.

However, achieving 51% control is extremely expensive and counterproductive, as it would erode trust in Bitcoin.

## Practical Challenges

### Transaction Speed

- Bitcoin’s 10-minute block time makes transactions slow.
- Faster confirmations are impractical due to network latency and mining dynamics.

### Storage and Scalability

- Every peer must maintain the entire transaction history.
- The Bitcoin blockchain is already hundreds of gigabytes in size.
- Limited block size restricts transaction throughput.

### Anonymity Considerations

Bitcoin transactions do not require real-world identities but are not fully anonymous:

- Public keys and transaction histories are visible.
- Chain analysis can link transactions to real-world identities.
- Privacy can be improved using techniques like new public keys per transaction.
