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

## 🛑 Why do you need this? (Real-World Use Cases)

Nobody integrates a tool just because it uses "Merkle Trees" or "Blockchains". People integrate tools to solve real problems and sleep well at night. Here is exactly what ProofCore solves for you:

1. **Crypto Wallets & Web3 Apps:** Protects against silent `.exe`/`.apk` replacements by hackers. If a compromised maintainer replaces a release binary with a drainer, the checksums will mismatch the blockchain anchor. Your users are safe.
2. **Outsource & Software Agencies:** Delivers a mathematical, timestamped PDF receipt (FRE 902 legally compliant) proving *exactly* what files you delivered to the client and at what time. No more "you missed the Friday deadline" or "this is the wrong build" disputes.
3. **Compliance & Audits (SOC 2, ISO 27001, PCI-DSS):** Instantly satisfies auditors requiring strict "Release Integrity" and "Change Management" controls without building complex internal infrastructure.
4. **Open-Source Maintainers:** Protects your community from supply chain attacks (like the `xz-utils` backdoor) by providing an immutable, publicly verifiable provenance layer.

---

## 🔒 Zero-Access Guarantee & Security

> **🔒 Zero-Access Guarantee:** ProofCore uses GitHub OIDC strictly as a cryptographic ID badge. The token contains **0 permissions** to access your private code, repositories, or secrets. It only proves the authenticity of the release runner to prevent spoofing.

ProofCore operates on a strict **Zero-Storage Principle**:
1. **Local Checksums:** The Action runs inside your GitHub Runner (`ubuntu-latest`) and computes `sha256sum` for all attached release assets **locally in ephemeral memory**.
2. **Zero Uploads:** Your heavy binaries (`.exe`, `.tar.gz`, `.apk`) are **NEVER uploaded to ProofCore servers**.
3. **Lightweight Manifest:** Only a lightweight manifest containing the `Target Commit SHA`, `Asset SHA-256 digests`, and `Release Notes` is transmitted to our Zero-Auth API.

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
      id-token: write  # REQUIRED: Generates OIDC token to cryptographically prove repository identity
    steps:
      - name: 🛡️ Cryptographically Notarize Release on TON
        uses: ProofCore-Protocol/proofcore-action@main
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

*(Note: `id-token: write` is strictly required to generate the GitHub OIDC token that gives your release the green "Verified by GitHub" checkmark).*

---

## 🔍 How End-Users Verify Binary Integrity

When users download binaries from your GitHub Releases:

1. They download the compiled binary (e.g., `wallet-setup.exe`) and click the **`ProofCore | Anchored on TON`** badge embedded in the release notes.
2. In the **ProofCore Web Explorer**, they can drag-and-drop the downloaded binary into the **«OPTIONAL: EXACT MATCH TEST»** box.
3. The browser independently computes the `SHA-256` hash client-side using the native **WebCrypto API** and compares it against the on-chain Merkle proof.
4. If a malicious actor modified even a single byte of the binary, the verification immediately fails with a cryptographic mismatch alert.

You can also download a **self-authenticating Evidence ZIP** from the explorer, which contains the original manifest, a PDF certificate, and 100% offline verification Python/HTML scripts.

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
| **Source Identity** | Proof of Runner Authenticity | GitHub Actions OIDC Signature |
| **Blockchain Anchor** | Merkle Root committed in a TON block transaction | Opcode `0x0` `MR: <root>` payload |

---

## 📄 License

Distributed under the **MIT License**. See `LICENSE` for more information.
