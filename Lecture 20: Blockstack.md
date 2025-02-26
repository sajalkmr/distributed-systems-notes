# Lecture 20: Blockstack

## Introduction

### Key Questions Addressed
1. **Building a Naming System** - A global Public Key Infrastructure (PKI) that maps names to public keys.
2. **Non-Cryptocurrency Use of Blockchain** - Exploring blockchain utility beyond financial applications.
3. **Decentralized Internet Services** - A new way to construct websites and applications.

## Centralized Web Architecture

### Traditional Web Model
- Users access websites via browsers.
- Websites have centralized servers with databases.
- User data is controlled by the website (e.g., Gmail, Facebook, Reddit).
- Convenient for developers, but has privacy and control issues.

### Problems with the Traditional Model
1. **Lack of User Control** - Users depend on platform-specific interfaces.
2. **Privacy Issues** - Websites can access and analyze user data.
3. **Security Risks** - Employees or attackers could exploit data.

## Decentralized Web with Blockstack

### Key Principles
- Users control their own data.
- Data is stored in personal cloud storage (e.g., AWS, Google Cloud).
- Applications run on client machines instead of centralized web servers.
- Access control is enforced cryptographically.

### Decentralized Architecture
- Users store data on their chosen cloud storage.
- Applications fetch data directly from user storage.
- Web applications function like traditional desktop applications.
- Storage system must support **sharing** and **access control**.

## Challenges of Decentralization

### Performance Issues
- Querying data across the internet is slower than querying a local database.
- Lack of efficient indexing and aggregation.
- Increased bandwidth usage due to data transfer overhead.

### Data Ownership and Privacy
- Users pay for their own storage.
- Cryptographic encryption ensures privacy.
- Managing shared access without a central authority is complex.

## Naming System in Blockstack

### Purpose of Naming
- Map user identities to storage locations and public keys.
- Enable verification of signed data.
- Support encryption-based access control.

### Key Properties of a Naming System
1. **Uniqueness** - Each name has a global, consistent meaning.
2. **Human Readability** - Names should be easy to recognize and remember.
3. **Decentralization** - No central authority should control name allocation.

### Implementation using Bitcoin
- Blockstack leverages the Bitcoin blockchain for name registration.
- Transactions contain name registrations.
- Naming is **first-come, first-served**.
- Blockstack servers watch the blockchain and maintain mappings.

### Limitations of Blockstack Naming
- Hard to find the correct name for a person.
- Human-readable names can be deceptive.
- No built-in identity verification.

## Blockstack System Components

### 1. **Bitcoin Blockchain**
- Stores name registration transactions.
- Ensures global uniqueness and order.

### 2. **Atlas Network**
- Maps hashed name records to actual metadata.

### 3. **Gaia Storage**
- Decentralized storage system where users store data.

### 4. **Blockstack Naming Servers**
- Index Bitcoin transactions to maintain an updated name database.

### 5. **Blockstack Browser**
- Manages user keys securely.
- Acts as an authentication layer for applications.

## Security and Usability Concerns

### Security Risks
- Users must safeguard their private keys.
- Key loss results in permanent data inaccessibility.
- No built-in key recovery mechanisms.

### Usability Challenges
- Developers find it harder to build applications.
- Users may be unwilling to pay for storage.
- Interoperability requires data format standardization.
