# 🛡️ ProofCore Release Notary (GitHub Action)

Protect your software supply chain by cryptographically anchoring your GitHub Release Notes into the **TON Blockchain**. 

If a malicious actor gains access to your repository and modifies release links, hashes, or contract addresses, the ProofCore integrity badge will expose the tampering, as the original release signature remains immutable on-chain.

## 🚀 Usage

Add this step to your existing Release workflow (e.g., `.github/workflows/release.yml`). It runs automatically when a new release is published.

```yaml
name: Release Notary
on:
  release:
    types: [published]

jobs:
  seal-release:
    runs-on: ubuntu-latest
    permissions:
      contents: write  # Required to update the release notes
    steps:
      - name: 🛡️ Cryptographically Seal Release on TON
        uses: ProofCore-Protocol/proofcore-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

## ✨ What happens under the hood?
1. The Action securely reads your published Release Notes.
2. It sends the payload to the Zero-Auth [ProofCore Protocol](https://proofcore.org) API.
3. The content is hashed (SHA-256) and queued for Merkle Tree batching on the TON Blockchain.
4. The Action automatically appends a live dynamic SVG badge to your GitHub Release description:
   `[![ProofCore](https://api.proofcore.org/api/badge/...)](https://proofcore.org/app/...)`
