<div align="center">

# 🛡️ ProofCore Release Notary

**Decentralized Software Supply Chain Security & Binary Notarization for GitHub Releases**

[![GitHub Marketplace](https://img.shields.io/badge/GitHub%20Marketplace-Release%20Notary-2088FF?logo=githubactions&logoColor=white)](https://github.com/marketplace/actions/proofcore-release-notary)
[![TON Blockchain](https://img.shields.io/badge/Blockchain-TON%20Testnet%2FMainnet-0098EA?logo=ton&logoColor=white)](https://ton.org)
[![Zero-Storage](https://img.shields.io/badge/Architecture-Zero--Storage%20(Client--Side)-00f298)](https://proofcore.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

[🌐 Protocol Website](https://proofcore.org) • [📖 OpenAPI Specification](https://proofcore.org/openapi.json) • [📦 PyPI Package](https://pypi.org/project/proofcore/)

</div>

---

## 🛑 Why ProofCore?

> **GitHub Releases tell users *where* a binary was published.**  
> **ProofCore lets them cryptographically verify *which exact bytes* were published.**

ProofCore creates an immutable, timestamped record of your release artifacts and anchors the cryptographic commitment directly to **The Open Network (TON) Blockchain** via binary Merkle Tree batching.

No binary uploads. No long-lived API keys. No blockchain gas management required.

---

### Real-World Use Cases

1. **Crypto Wallets & Web3 Software (Anti-Drainer Protection)**  
   Protects against silent `.exe`, `.apk`, or `.dmg` replacements. If a compromised maintainer or CI workflow replaces a release binary with malware, its SHA-256 digest will no longer match the notarized release. Users can detect the tampering.

2. **Software Agencies & Outsourced Development**  
   Creates a timestamped cryptographic record proving exactly which files were delivered or published, reducing disputes over builds, versions, and delivery dates.

3. **Compliance & Auditing (SOC 2, ISO 27001, PCI-DSS)**  
   Provides publicly verifiable release-integrity evidence that can complement existing change-management and software supply-chain controls.

4. **Open-Source Software**  
   Gives users an independent mechanism to verify that the binary they downloaded matches the artifact originally notarized by the automated CI/CD pipeline.

---

## 🔒 Zero-Access & Zero-Storage Architecture

> **🔒 Zero-Access Guarantee:** GitHub OIDC is used strictly to prove repository and workflow identity. ProofCore does not use the OIDC token to access your private source code, repositories, or secrets.

ProofCore operates on a strict **Zero-Storage Principle**:

1. **Local Hashing:** Checksums (`SHA-256`) for all release assets are computed **locally in the runner's ephemeral memory**.
2. **Zero Uploads:** Your heavy binaries and proprietary code are **NEVER uploaded to ProofCore**.
3. **Lightweight Manifest Only:** ProofCore receives only the metadata required to create the cryptographic proof (Commit SHA, repository identity, artifact digests).

---

## 🚀 1-Minute Quickstart

Add this step to your release workflow (e.g. `.github/workflows/release.yml`). It executes automatically whenever a new release is published.

```yaml
name: Release Notary

on:
  release:
    types: [published]

jobs:
  seal-release:
    runs-on: ubuntu-latest

    permissions:
      contents: write
      id-token: write

    steps:
      - name: 🛡️ Cryptographically Notarize Release on TON
        uses: ProofCore-Protocol/proofcore-action@main
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Which version should I use?

ProofCore supports three ways to select the Action version:

* **`@main` — latest development version**  
  `uses: ProofCore-Protocol/proofcore-action@main`  
  Best for trying ProofCore and getting the latest features and fixes. *(Quick start)*

* **`@v1` — stable major version**  
  `uses: ProofCore-Protocol/proofcore-action@v1`  
  Recommended for production workflows. Your workflow stays on the latest compatible `v1` release while receiving backward-compatible updates. *(Production)*

* **`@<commit-sha>` — exact revision**  
  `uses: ProofCore-Protocol/proofcore-action@<commit-sha>`  
  Recommended for security-sensitive or highly reproducible environments where you want to pin the Action to one exact revision. *(Maximum reproducibility)*

### Required Permissions

* `contents: write`: Allows ProofCore to update the GitHub Release with the verification badge.
* `id-token: write`: Allows the workflow to request a GitHub OIDC token. ProofCore uses this token to cryptographically verify the identity of the repository and workflow submitting the notarization request.

---

## 🔍 How End-Users Verify Binary Integrity

When users download assets from your GitHub Release:

1. They download the compiled binary (e.g. `wallet.apk`) and click the **`ProofCore | Anchored on TON`** badge embedded in the release notes.
2. In the **ProofCore Web Explorer**, they can drag-and-drop the downloaded binary into the **«OPTIONAL: EXACT MATCH TEST»** box.
3. The browser independently computes the `SHA-256` hash client-side using the native **WebCrypto API** and compares it against the on-chain Merkle proof.
4. If a malicious actor modified even a single byte of the binary, the verification immediately fails with a cryptographic mismatch alert.

Users can also download a **self-authenticating Evidence ZIP** from the explorer, which contains the original manifest, a PDF certificate, and 100% offline verification Python/HTML scripts.

> *"Don't trust the release host alone. Verify the cryptographic proof."*

---

## 📜 Verified Release Manifest Schema

Every notarized release anchors a deterministic manifest:

| Field | Description | Cryptographic Mechanism |
| :--- | :--- | :--- |
| **Repository** | Full GitHub repository name | Repository identity |
| **Release Tag** | Immutable tag reference (e.g. `v1.2.0`) | Git Ref binding |
| **Target Commit** | Commit associated with the release | Git commit reference |
| **Artifact Checksums** | SHA-256 digests of release artifacts | SHA-256 |
| **Source Identity** | GitHub Actions workflow identity | GitHub OIDC |
| **Blockchain Anchor** | Merkle root recorded on TON | TON transaction |

---

## 🧩 What ProofCore Does & Doesn't Do

### ✅ ProofCore DOES:
* Hash release artifacts locally on the GitHub runner.
* Bind commit metadata, release notes, and binary checksums into a deterministic manifest.
* Verify runner authenticity using GitHub OIDC signatures to prevent spoofing.
* Anchor the resulting Merkle Root to the TON Blockchain with immutable timestamps.
* Provide an independent verification path for end-users.

### ❌ ProofCore DOES NOT:
* Upload, store, or inspect your compiled binaries.
* Replace GitHub Releases.
* Scan binaries for malware or zero-day vulnerabilities.
* Guarantee that your build process itself was uncompromised.
* Replace code signing, reproducible builds, or SBOMs.

ProofCore acts as an **additional cryptographic integrity and timestamping layer** for published release artifacts.

---

## 📄 License

Distributed under the **MIT License**. See `LICENSE` for more information.
