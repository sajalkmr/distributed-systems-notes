# Lecture 18: Fork Consistency, Certificate Transparency

## Introduction
- **Certificate Transparency (CT)** is a system designed to improve security in open distributed systems.
- Unlike closed systems (e.g., Raft consensus), open systems have no universally trusted authority.
- The primary concern in open systems: **how to verify that you are communicating with the correct entity**.
- **CT helps ensure that all parties see the same information about certificates**, which is a consistency issue.
- It is one of the few non-cryptocurrency applications of blockchain-like designs.

---

## Web Security Before 1995
- Before the introduction of **certificates**, man-in-the-middle (MITM) attacks were common.
- **MITM Attack:**
  1. A user types `gmail.com` in their browser.
  2. The browser queries DNS for the IP address.
  3. A malicious entity intercepts the DNS response and returns a fake IP.
  4. The user connects to an attacker-controlled server that mimics Gmail.
  5. The user unknowingly enters their credentials into the attacker's fake site.

- **SSL/TLS and Certificates (1995 onwards)**:
  - Websites began using **public-private key pairs** for authentication.
  - **Certificate Authorities (CAs)** verify domain ownership and issue signed certificates.
  - Browsers check certificates against a list of trusted CAs.
  - **Problem:** Any of the hundreds of CAs can issue certificates for any domain.

---

## Problems with Certificate Authorities
- Initially, only a few trusted CAs existed; today, **hundreds exist**.
- Any CA can issue a certificate for any domain, increasing the risk of abuse.
- **Bogus Certificates**:
  - Some CAs have mistakenly (or maliciously) issued certificates for domains they do not own.
  - Attackers can obtain fake certificates and execute MITM attacks.
  - Browsers trust any certificate signed by any CA in their list.

- **Proposed Solution: A Centralized Certificate Database**
  - Challenges:
    - Difficult to determine the rightful owner of a domain.
    - Websites frequently change certificates, requiring constant updates.
    - No single universally trusted authority to manage such a database.

---

## Certificate Transparency (CT)
- **Goal:** Provide an auditable system where all issued certificates are **publicly logged**.
- **CT does not prevent bogus certificates but makes them detectable**.
- Ensures that certificate issuance is transparent and verifiable by anyone.

### How CT Works
1. **Certificate Authority (CA) issues a certificate** for a domain (e.g., `gmail.com`).
2. **The CA submits this certificate to a CT log server**.
3. **The CT log server appends the certificate to a public log**.
4. **A browser, before trusting a certificate, checks if it is in the log**.
5. **Website owners and monitors check logs for unexpected certificates**.
6. **If a bogus certificate is issued, it becomes publicly visible and can be revoked**.

---

## Ensuring Trustworthiness in CT Logs
### Append-Only Logs
- The CT log should be **append-only**:
  - A log server must not be able to **delete or modify** entries.
  - Ensures that once a certificate is logged, it remains visible.

### No Forking (Equivocation Prevention)
- A log server must not **show different logs** to different parties.
- A fork occurs if:
  - A log server presents **one version of the log to browsers**.
  - Another version (excluding some certificates) is shown to **monitors**.
- **Fork prevention ensures that all parties see the same log**.

---

## Technical Implementation: Merkle Trees
- **CT logs use Merkle Trees to ensure integrity and efficiency.**
- **How a Merkle Tree Works:**
  1. Each certificate is **hashed**.
  2. Pairs of hashes are combined and hashed again.
  3. This process continues until a single **root hash** (Merkle Root) is obtained.
  4. **The Merkle Root uniquely represents the log contents**.

- **Signed Tree Heads (STH)**:
  - The Merkle Root is signed by the log server.
  - This prevents tampering and ensures accountability.

### Proofs in Certificate Transparency
1. **Proof of Inclusion**:
   - A browser can verify that a certificate is in the log **without downloading the full log**.
   - The log server provides a Merkle Proof, consisting of the necessary hashes to verify the certificate’s presence.
   - This ensures that certificates cannot be removed after being logged.

2. **Proof of Consistency**:
   - Ensures that **new versions of the log are strict extensions of old versions**.
   - Prevents log servers from rolling back or altering history.

---

## Gossip and Fork Detection
- **CT relies on "gossip" to detect forks.**
- **Gossip Mechanism:**
  1. Browsers, servers, and monitors exchange the latest **Signed Tree Heads (STHs)**.
  2. If two different STHs are detected, it signals a potential fork.
  3. Affected logs are flagged for investigation.

- **Fork Consistency Enforcement**:
  - If a log server presents a browser with an STH, it **must provide proof** that this is an extension of the previous state.
  - Prevents log servers from showing a different version of history to different parties.

---

## Deployment and Real-World Use
- **Google Chrome and Safari enforce CT**.
- **Multiple log servers exist**, maintained by independent organizations.
- **CA Compliance**:
  - Chrome requires CAs to submit certificates to multiple CT logs.
  - Websites must include proof that their certificate exists in a CT log.

---

## Limitations and Open Challenges
- **Delay in Detection**:
  - Bogus certificates may still be trusted until detected.
- **Malicious or Compromised Log Servers**:
  - Attackers could attempt to manipulate logs before they are caught.
- **Reliance on Browser Vendors**:
  - A small number of browser vendors (Google, Apple) control which logs are trusted.

---

## Key Takeaways
- **CT improves security through transparency, not prevention.**
- **Merkle Trees ensure efficient, tamper-proof logging.**
- **Inclusion proofs allow browsers to verify certificates efficiently.**
- **Consistency proofs prevent logs from rewriting history.**
- **Gossip helps detect forks, ensuring all parties see the same data.**
- **CT is a real-world system used by major browsers to improve web security.**

---


