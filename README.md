# 📦 Traceprompt — Open Anchors

Immutable, publicly-verifiable timestamp anchors for every Traceprompt Merkle batch.

> **🔒 Why does this repo exist?**
>
> - To prove that Traceprompt's audit trail hasn't been tampered with.
> - To give regulators and customers a self-service way to verify evidence—even if they don't trust us.

---

## What you'll find here

| File / folder       | Purpose                                              |
| ------------------- | ---------------------------------------------------- |
| `YYYY-MM-DD.csv`    | One CSV per UTC day. Each line = one Merkle batch.   |
| `.gitattributes`    | Ensures the CSVs are treated as LF text.             |
| Git tags (optional) | Monthly signed tags so auditors can checkpoint fast. |

All commits are **GPG-signed** by the Traceprompt anchor worker (`bot@traceprompt.dev`).

---

## CSV format

```
timestamp,org_or_tenant_id,batch_id,merkle_root_hex
```

- `timestamp` – ISO-8601 when the root was sealed.
- `org_or_tenant_id` – UUID (`org_…`) for modern accounts, legacy `tnt_…` for early tenants.
- `batch_id` – Auto-incrementing PK from the `merkle_batches` table.
- `merkle_root_hex` – 64-char SHA-256 of the Merkle tree built from that batch's leaf hashes.

**Example line:**

```
2025-05-25T18:42:02.331Z,org_12b9d2c2,56,8376966c62452b8b623b131b8af3a18d34ff65d9445c52c3b1c19e2c1f9f5b9f
```

---

## How to verify an anchor

### 1. Verify the Git commit itself

```bash
git log -1 --show-signature
# Look for "gpg: Good signature from 'Traceprompt Anchor Bot …'"
```

### 2. Re-compute the Merkle root

1. Download the `batch-56.json.gz` file from the S3 URI in the DB.
2. Decompress and extract the array of hex leaf hashes.
3. Run:

```bash
npm i merkle-tree-solidity --silent
node <<'JS'
import { MerkleTree } from 'merkle-tree-solidity';
import fs from 'fs';
const leaves = JSON.parse(fs.readFileSync('batch-56.json', 'utf8'));
const tree = new MerkleTree(leaves.map(l => Buffer.from(l, 'hex')), 'sha256');
console.log(tree.getRoot().toString('hex'));
JS
```

4. Compare the output to the root in the CSV line. They must match.

### 3. (Optional) Verify OpenTimestamps proof

If the row also has an `ots_txid`, you can verify the OpenTimestamps proof with:

```bash
ots verify batch-56.json.ots
```

---

## Write protection

- `main` is a protected, locked branch.
- Only the GitHub Actions deploy key held by the anchor worker can push.
- Pushing without a valid GPG signature is rejected.

---

## Contributing

This repo is append-only. We don't accept PRs or issues here.

If you discover an inconsistency, open a ticket in [traceprompt/traceprompt](https://github.com/traceprompt/traceprompt) instead.

---

## License

**Copyright © 2025 PAUELLE LIMITED.**

- All CSV data is released under the **CC0 1.0** public-domain waiver.
- Repo configuration & documentation are **MIT-licensed**.
