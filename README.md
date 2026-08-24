<div align="center">

# 🛡️ ProofCore Release Notary

**Decentralized Software Supply Chain Security & Binary Notarization for GitHub Releases**

[![GitHub Marketplace](https://img.shields.io/badge/GitHub%20Marketplace-Release%20Notary-2088FF?logo=githubactions&logoColor=white)](https://github.com/marketplace/actions/proofcore-release-notary)
[![TON Blockchain](https://img.shields.io/badge/Blockchain-TON%20Testnet%2FMainnet-0098EA?logo=ton&logoColor=white)](https://ton.org)
[![Zero--Storage](https://img.shields.io/badge/Architecture-Zero--Storage%20(Client--Side)-00f298)](https://proofcore.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

[🌐 Protocol Website](https://proofcore.org) • [📖 OpenAPI Specification](https://proofcore.org/openapi.json) • [📦 PyPI Package](https://pypi.org/project/proofcore/)

</div>

---

## 🚨 The Threat: Software Supply Chain Attacks

In modern software distribution, GitHub Releases are inherently mutable:
- **Silent Binary Replacement:** If a maintainer account or CI pipeline is compromised, an attacker can silently replace compiled binaries (`.exe`, `.dmg`, `.apk`, `.whl`) with trojanized payloads via the GitHub API without altering a single line of git history.
- **Forged Git Timestamps:** Git commit dates are easily forged (`GIT_COMMITTER_DATE="2020-01-01"`), providing zero legal proof of existence.
- **Single Point of Failure:** Centralized hosting platforms can be subpoenaed, compromised, or experience silent data corruption.

**ProofCore Release Notary** creates an immutable, cryptographically verifiable record of your software releases anchored directly onto **The Open Network (TON) Blockchain** using binary Merkle Tree batching.

---

## 🔒 Zero-Storage Client-Side Architecture

ProofCore operates on a strict **Zero-Storage Principle**:
1. **Local Checksums:** The Action runs inside your GitHub Runner (`ubuntu-latest`) and computes `sha256sum` for all attached release assets **locally in ephemeral memory**.
2. **Zero Uploads:** Your heavy binaries (`.exe`, `.tar.gz`, `.apk`) are **NEVER uploaded to ProofCore servers**.
3. **Cryptographic Manifest:** Only a lightweight manifest containing the `Target Commit SHA`, `Asset SHA-256 digests`, and `Release Notes` is transmitted to the Zero-Auth API.

```text
┌─────────────────────────────────────────────────────────────┐
│             GITHUB RUNNER (Local Ephemeral Compute)         │
├─────────────────────────────────────────────────────────────┤
│ • Read Git Commit SHA: 7c3fbce...                           │
│ • Compute Checksums  : SHA-256(setup.exe), SHA-256(app.apk) │
│ • Build Manifest     : Commit + Checksums + Release Notes   │
└──────────────────────────────┬──────────────────────────────┘
                               │
               POST Manifest Digest (2 KB Payload)
                               │
                               ▼
                   ┌───────────────────────┐
                   │ ProofCore API Gateway │
                   └───────────┬───────────┘
                               │
                    SHA-256 Merkle Batching
                               │
                               ▼
                   ┌───────────────────────┐
                   │    TON Blockchain     │
                   │ (Immutable Commitment)│
                   └───────────────────────┘
```

---

## 🚀 1-Minute Quickstart

Add this step to your release workflow (e.g., `.github/workflows/release.yml`). It executes automatically whenever a new release is published.

```yaml
name: Release Notary
on:
  release:
    types: [published]

jobs:
  seal-release:
    runs-on: ubuntu-latest
    permissions:
      contents: write  # Required to update the release notes with the badge
    steps:
      - name: 🛡️ Cryptographically Notarize Release on TON
        uses: ProofCore-Protocol/proofcore-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

---

## 🔍 How End-Users Verify Binary Integrity

When users download binaries from your GitHub Releases:

1. They download the compiled binary (e.g., `wallet-setup.exe`) and click the **`ProofCore | Anchored on TON`** badge in the release notes.
2. In the **ProofCore Web Explorer**, they drag-and-drop the downloaded binary into the **«OPTIONAL: EXACT MATCH TEST»** box.
3. The browser independently computes the `SHA-256` hash client-side using the native **WebCrypto API** and compares it against the on-chain Merkle proof.
4. If a malicious actor modified even a single byte of the binary, the verification immediately fails with a cryptographic mismatch alert.

> *"Don't trust the release host. Verify the cryptographic proof."*

---

## 📜 Verified Release Manifest Schema

Every notarized release anchors a deterministic manifest:

| Field | Description | Cryptographic Mechanism |
| :--- | :--- | :--- |
| **Repository** | Full GitHub repository name | Plaintext context |
| **Release Tag** | Immutable tag reference (e.g., `v1.2.0`) | Git Ref binding |
| **Target Commit** | Exact Git commit hash of the release build | SHA-1 / SHA-256 Git Tree Hash |
| **Artifact Checksums** | SHA-256 digests of all attached binaries/packages | SHA-256 byte digest |
| **Blockchain Anchor** | Merkle Root committed in a TON block transaction | Opcode `0x0` `MR: <root>` payload |

---

## 📄 License

Distributed under the **MIT License**. See `LICENSE` for more information.
