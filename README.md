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
   Protects users from silent binary replacements if a maintainer account or CI secret is compromised. If a release `.exe`, `.apk`, or `.dmg` is altered with malware or a drainer, its SHA-256 digest immediately mismatches the immutable blockchain anchor.

2. **Software Agencies & Contractors (Cryptographic Proof of Delivery)**  
   Generates a self-authenticating, timestamped digital record (FRE 902 compliant) proving *exactly* which build and source commit were delivered and when. Eliminates client disputes over delivery deadlines and build versions.

3. **Compliance, SOC 2 & ISO 27001 (Audit Trail for Release Integrity)**  
   Provides external auditors with tamper-proof evidence of release integrity and change management controls without maintaining complex internal transparency log infrastructure.

4. **Open-Source & CLI Tools (Defeating Supply Chain Attacks)**  
   Gives developers downloading release binaries (`curl | bash`, `.tar.gz`, `.deb`) an independent, client-side mechanism to verify that the downloaded package matches the automated CI/CD build.

---

## 🔒 Zero-Access & Zero-Storage Architecture

> **🔒 Zero-Access Guarantee:** ProofCore uses GitHub Actions OIDC strictly as a cryptographic ID badge. The token contains **0 permissions** to access your private code, repositories, or secrets. It only proves the authenticity of the release workflow runner.

```text
┌─────────────────────────────────────────────────────────────┐
│             GITHUB RUNNER (Local Ephemeral Compute)         │
├─────────────────────────────────────────────────────────────┤
│ 1. Read Git Commit SHA: 04abb6c...                          │
│ 2. Compute Checksums  : SHA-256(setup.exe), SHA-256(app.apk)│
│ 3. Request OIDC Token : Audience = "api.proofcore.org"      │
│ 4. Build Manifest     : Commit + Checksums + Release Notes  │
└──────────────────────────────┬──────────────────────────────┘
                               │
            POST Manifest + OIDC JWT (Lightweight ~2 KB)
                               │
                               ▼
                   ┌───────────────────────┐
                   │ ProofCore API Gateway │
                   │ (OIDC Signature Check)│
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

1. **Local Hashing:** Checksums (`SHA-256`) for all release assets are computed **locally in runner memory**.
2. **Zero Uploads:** Your heavy binaries and proprietary code are **NEVER uploaded to ProofCore**.
3. **Lightweight Manifest Only:** ProofCore receives only the metadata (Commit SHA, tag, artifact checksums, release notes) and the OIDC token.

---

## 🚀 1-Minute Quickstart

Add ProofCore to your release workflow (e.g. `.github/workflows/release.yml`):

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
      id-token: write  # REQUIRED: Generates OIDC token to prove repository identity

    steps:
      - name: 🛡️ Cryptographically Notarize Release on TON
        uses: ProofCore-Protocol/proofcore-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

When a release is published, ProofCore automatically hashes the release assets, creates an immutable on-chain record, and appends a dynamic verification badge to your GitHub Release notes.

---

### Action Versioning Best Practices

* **`uses: ProofCore-Protocol/proofcore-action@v1`** *(Recommended)*: Tracks the latest stable releases within the `v1` major version.
* **`uses: ProofCore-Protocol/proofcore-action@main`**: Follows the development branch with the newest experimental features.
* **`uses: ProofCore-Protocol/proofcore-action@<commit-sha>`**: Pins the Action to an exact, immutable commit for high-security environments.

---

### Required Permissions

* `contents: write`: Allows the Action to append the verification badge to your release notes via the GitHub API.
* `id-token: write`: Allows the workflow to request a short-lived GitHub OIDC token. ProofCore validates this token against GitHub's public keys to verify repository identity and issue the green **`✓ VERIFIED BY GITHUB OIDC`** badge.

---

## 🔍 How End-Users Verify Binary Integrity

When users download assets from your GitHub Release:

1. They download the compiled binary (e.g. `wallet.apk`) and click the **`ProofCore | Anchored on TON`** badge.
2. In the **ProofCore Web Explorer**, they can drag-and-drop the downloaded binary into the **«OPTIONAL: EXACT MATCH TEST»** box.
3. The browser calculates the binary's SHA-256 hash locally using the native **WebCrypto API** (no file is uploaded).
4. The digest is matched against the on-chain Merkle proof:
   * **Local Hash == Manifest Hash == TON Blockchain Transaction (3-Way Match).**

If even a single byte of the binary has been tampered with, the verification immediately fails with a cryptographic mismatch warning.

### 📦 Standalone Evidence Package (Offline ZIP)

Users can also download a self-authenticating **Evidence ZIP** from the Explorer containing:
* The original release manifest (`release_manifest.txt`).
* A legal forensic PDF certificate.
* Standalone `verify.py` and `verify.html` scripts that perform a **100% independent 3-Way verification** directly against TON nodes without relying on ProofCore servers.

---

## 📜 Verified Release Manifest Schema

Every notarized release produces a deterministic manifest:

| Field | Description | Cryptographic Mechanism |
| :--- | :--- | :--- |
| **Repository** | Verified GitHub repository name | GitHub OIDC `repository` claim |
| **Release Tag** | Immutable tag reference (e.g. `v1.2.0`) | Git Ref binding |
| **Target Commit** | Exact Git commit hash of the build | Git commit tree hash |
| **Artifact Checksums** | SHA-256 digests of all attached packages | SHA-256 byte digest |
| **Source Identity** | Runner provenance verification | RS256 JWT signature via GitHub JWKS |
| **Blockchain Anchor** | Merkle Root committed on-chain | TON block transaction (`MR: <root>`) |

---

## 🧩 Threat Model: What ProofCore Does & Doesn't Do

### ✅ ProofCore DOES:
* Compute release asset hashes client-side on the ephemeral GitHub runner.
* Bind commit metadata, release notes, and binary checksums into a deterministic manifest.
* Verify runner authenticity using GitHub OIDC signatures to prevent spoofing.
* Anchor the resulting Merkle Root to the TON Blockchain with immutable timestamps.
* Provide standalone offline verification tools.

### ❌ ProofCore DOES NOT:
* Upload, store, or inspect your compiled binaries.
* Scan binaries for malware or zero-day vulnerabilities.
* Guarantee that the source code was free of bugs before compilation.
* Replace code signing (e.g. Apple Developer ID, Windows Authenticode) or SBOM generation.

ProofCore acts as an **independent, publicly verifiable integrity and timestamping layer** for software distribution.

---

## 📄 License

Distributed under the **MIT License**. See `LICENSE` for more information.
