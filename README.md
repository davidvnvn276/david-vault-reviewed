# DAVID: successful settlement call, pending withdrawal

This reviewed package accompanies `article.md`. It preserves the supplied research files unchanged and adds a separate reviewer audit, a fresh live-RPC fork repeat, a saved-state EVM replay and three evidence graphics. All documentation is in English.

**Main finding:** with a 5,445-mREAL redemption-batch requirement, 1,000 mREAL did not settle the batch even though B's individual net amount was 495. Cancelling A before the valuation proposal reduced the batch requirement to 495; B then claimed 495 with the same 1,000 available.

**Scope:** Rayls-deployed code, existing mock mREAL, locally simulated authorities, and local EVM transactions. Mainnet inspection was read-only. This does not establish a real user's production withdrawal, real-credit backing or operator identity.

## 1. Extract and find the package root

Download `david-vault-reviewed.zip`, extract it fully, and open the `david-vault-reviewed` folder. Do not execute programs inside your ZIP application's preview.

You should see:

```text
README.md
article.md
SHA256SUMS-reviewed.txt
original-evidence/
review/
review-results/
figures/
```

Open a terminal in that folder. On Windows, click File Explorer's address bar, type `powershell`, and press Enter. On macOS/Linux, open Terminal and `cd` to the extracted folder.

Run one command box at a time. Text marked “expected” is not a command. Keep new output folders separate from supplied observations.

## 2. Start with the offline package check

Install Python 3.12 or another supported Python 3 from [python.org](https://www.python.org/downloads/) if needed. The supplied validation used Python 3.12.14 on Linux. The commands below use `python`; on Windows, use `py -3.12` instead if that is how Python is installed.

```bash
python --version
python review/verify_package.py
```

Expected: a package-manifest PASS and the original verifier's four PASS lines. No pip package, Node.js, RPC connection or wallet is needed. The script checks SHA-256 identity and runs the original offline verifier without changing its files.

If a file is missing or a hash differs, stop and obtain a clean package. Do not regenerate a manifest to hide an unexpected change. Checksums establish identity relative to the distributed manifest, not who produced an RPC reply or whether it represents independently authenticated consensus state.

The original verifier can also be run directly:

```bash
python original-evidence/verify_saved.py
```

It targets the original `original-evidence/data/` records. It does not verify arbitrary new fork-output folders. Do not use Python's `-O` switch, which disables its assertions.

## 3. Inspect the finding without running an EVM

Open these files in a text editor without saving changes:

| File | What to inspect |
|---|---|
| `original-evidence/data/local-fork/summary.json` | Initial state; five `scenarios`; each case's `before`, `settled`, `final` and decoded events |
| `original-evidence/data/local-fork/cash-1000-settle-receipt.json` | `status` is `0x1`, but no raw SettleRedeem event |
| `original-evidence/data/local-fork/cash-threshold-settle-receipt.json` | Batch settlement, underlying transfer, share burn and fee mints |
| `original-evidence/data/local-fork/cash-threshold-claim-B-receipt.json` | Actual subsequent 495-mREAL transfer |
| `original-evidence/data/local-fork/cash-1000-after-cancel-A-after-claims.json` | B's payment and 505 mREAL left in the Safe |
| `review-results/independent-audit.json` | Separate reviewer checks, derived exact amounts and raw-event results |
| `review/REVIEW.md` | Full findings, corrections, evidence boundaries and graphic placement |

All asset/share raw integers use 18 decimals. Divide by 10^18 to display whole units. Preserve the integers as strings; floating-point conversions and spreadsheet numeric cells can lose precision. `5444999999999999999999` is 5,444.999999999999999999 mREAL, one smallest asset unit below 5,445.

The three successful settlement indicators are different: the receipt has status 1; the batch emitted SettleRedeem and moved pending requests to claimable; separate claims actually transferred tokens. Do not substitute one for another.

## 4. Install the locked EVM dependencies

Install Node.js and npm from [nodejs.org](https://nodejs.org/en/download). This review ran Node v24.19.0 and npm 11.9.0 on Linux. Native Windows execution was not tested here.

From the package root:

```bash
node --version
npm --version
npm ci --prefix original-evidence/fork --ignore-scripts --no-audit --no-fund
```

On PowerShell, use `npm.cmd` instead of `npm` if execution policy blocks `npm.ps1`. No global execution-policy change is needed. The supplied lockfile has the original documented optional-fsevents metadata correction; its prior copy remains in `original-evidence/provenance/`.

This exact clean install with `--ignore-scripts` completed for this review. Ganache may report that its native µWS binary is incompatible and fall back to JavaScript. That warning is not a test outcome: inspect the process exit code and result files.

## 5. Re-execute the original block using saved state

This route runs EVM code again rather than only reading a saved result table. After dependency installation it needs no live RPC access.

Choose an output folder that does not already exist:

```bash
node review/replay_saved.cjs replay-your-check-01
```

Expected: the original harness prints its summary, followed by a PASS saying that five EVM scenarios matched the original state and economic settlement events. Confirm that `replay-your-check-01/replay-comparison.json` contains `status: "passed"`, the original fork block **12557717**, and `no_network_fallback: true`.

The wrapper supplies only replies actually preserved in the upstream read journal. Missing replies fail; there is no fallback that invents zero balances or queries a newer state. The original harness bytes are unchanged. A provider option omits its unused, randomly generated default wallets; all explicitly unlocked controller, Safe, manager and outsider identities remain. Local timestamps, receipt hashes and valuation-expiration timestamps can differ; scenario states and economic settlement events are compared.

**A saved-state replay checks execution conditional on supplied state. It is not a fresh independent mainnet observation.**

## 6. Run the separate deep verifier

```bash
node review/verify_deep.cjs
```

Expected: PASS for source bytes, RPC bindings, deployed runtimes, raw events and exact cash/share accounting. It checks the original fork, the supplied saved-state replay and the fresh review fork, and updates `review-results/independent-audit.json`.

To verify your new replay directory as well:

```bash
node review/verify_deep.cjs replay-your-check-01
```

This writes `review-results/additional-fork-audit.json`. Its checks still use the distributed source and mainnet-read records for context. It is not a tool for certifying an unrelated vault or fixture.

The verifier derives event topics from the published library declaration, checks raw receipts rather than trusting only event-name strings, decodes RPC returns with the ABI, and recomputes cash/share deltas. Program assertions are not independent experiments and do not increase the evidential sample size.

## 7. Obtain a fresh read-only mainnet capture

This optional route requires an internet connection. From the package root, create a Python virtual environment:

```bash
python -m venv .venv
```

On Windows PowerShell:

```powershell
.\.venv\Scripts\python.exe -m pip install pycryptodome==3.23.0
.\.venv\Scripts\python.exe original-evidence/read_vault.py --vault 0xcc180cb4031de7c2ec3d4c16e36ede4c6550eda0 --holder 0x4d76d208692c1b1c22d77350b89396d76c947e22 --holder 0xc7e33057e72a5331e9ecbdd8159995de6c78f554 --output fresh-your-read-01
```

On macOS/Linux:

```bash
.venv/bin/python -m pip install pycryptodome==3.23.0
.venv/bin/python original-evidence/read_vault.py --vault 0xcc180cb4031de7c2ec3d4c16e36ede4c6550eda0 --holder 0x4d76d208692c1b1c22d77350b89396d76c947e22 --holder 0xc7e33057e72a5331e9ecbdd8159995de6c78f554 --output fresh-your-read-01
```

Expected: a new folder containing raw requests/responses, metadata and summary. It pins the block read at run time. The implementation, pending requests or token state may legitimately differ from this package.

To ask for a particular historical block, append `--block 12556781` to the reader command. Historical availability is provider-dependent. An unavailable state, null response or pruning error is not a zero balance.

The Python reader does not sign or send transactions. `from` on an eth_call simulates a caller and does not prove key possession. Its `--block` option does not apply to the original Node fork harness.

## 8. Repeat the five scenarios from fresh live RPC state

After step 4:

```bash
node original-evidence/fork/fork_probe.cjs fresh-your-fork-01
```

This original harness starts from the latest Rayls block available at run time. It runs mutations only in an in-process Ganache VM with chain ID 31337. The upstream adapter has a read-method allowlist and rejects transaction-submission methods. There is no separate local server to start, and no seed phrase or private key is needed.

Expect a `summary.json` containing status `passed`, five scenarios, a new fork block/hash, and no `failure.json`. Inspect raw receipts and exact claim deltas as well. A fresh run must retain the existing fixture: 5,000/500 pending shares, the 1% exit fee, correct runtime and 5,445-mREAL requirement. If a guard fails, retain the failure/journal. Do not disable guards to manufacture PASS.

To audit this fresh folder with the separate reviewer script:

```bash
node review/verify_deep.cjs fresh-your-fork-01
```

These commands do not reproduce the historical block automatically. Use step 5 for saved-state execution at the original fork origin. See `original-evidence/README.md` for the original detailed request-probe and manual PowerShell walkthrough.

## 9. What the supplied review captures contain

| Directory | Meaning |
|---|---|
| `original-evidence/` | Supplied package bytes; original 174-entry manifest retained |
| `review/sources/` | Four core source files freshly fetched at the pinned GitHub commit |
| `review-results/replay-final/` | Actual EVM re-execution using the original saved RPC replies |
| `review-results/fresh-mainnet/` | New read-only capture at Rayls block 12,602,387 |
| `review-results/fresh-local-fork/` | New five-scenario run from Rayls block 12,602,402 |
| `review-results/independent-audit.json` | Independent saved-record review outputs |
| `figures/` | Three rendered PNGs, vector SVGs, unchanged mark and figure specifications |

The source comparison covers four selected core files. The runtime checks establish byte identity with Explorer metadata and stability for the captured executed code across snapshots. They are not an independent compiler rebuild or source-to-runtime verification for every external library. Separate Explorer metadata queries for the ERC7540Lib and FeeLib addresses returned 404.

The original request-silo accounting and initial mainnet read were at separate blocks, listed in `review/REVIEW.md`. A new fork run does not prove unchanged state throughout the time between captures. No observation authenticates a named issuer from a vault name alone.

## 10. Troubleshooting

| Problem | What to do |
|---|---|
| File not found | Extract fully and return to the package root, where this README sits |
| Existing output directory | Keep it; choose a new name ending in `02` |
| `Cannot find module` | Finish step 4 using the supplied lockfile |
| `No module named Crypto` | Use the virtual environment's Python for both install and read command |
| Native µWS warning | Wait for the actual result; JavaScript fallback was used successfully here |
| Fixture/runtime assertion fails | Preserve failure and journal; the new state may differ |
| Missing saved replay response | Stop; do not fabricate a response or silently use live state |
| HTTP error, timeout or pruning | Preserve the error and report unavailable evidence, not a zero balance |
| Manifest mismatch after deliberately running a verifier | Verifiers can refresh reports; compare a clean distribution copy before interpreting changed generated outputs |

## 11. Publishing and graphic placement

Publish this exact reviewed ZIP, or the readable package contents, at a public URL before submitting the article. Link to the current reviewed file; an older fee/redemption package is not evidence for this study. Use a commit permalink or release asset so the version remains identifiable.

After uploading, add this sentence to the article with your real URL:

> Reproducibility package: [download the evidence and step-by-step checks](YOUR_ACTUAL_PUBLIC_PACKAGE_URL). It includes the original observations, five-scenario harness, fresh fork repeat, raw receipts and a saved-state EVM replay. The README distinguishes internal consistency checks from fresh observations and local execution.

`YOUR_ACTUAL_PUBLIC_PACKAGE_URL` is an editing instruction here, not a published link. No GitHub repository was modified or release uploaded by this review.

- Cover: `figures/01-banner.png` below the article title/subtitle.
- Five-scenario evidence: `figures/02-scenario-matrix.png` after the scenario section.
- Token/share accounting: `figures/03-cash-and-shares.png` after reconciliation.

The figures contain no snapshot dates or account watermark. Block numbers and mock/local labels retain their evidential scope. They were drawn from exact data as SVG and rendered to PNG; no browser screenshot or generated test output is fabricated.

## 12. Integrity and limitations

`SHA256SUMS-reviewed.txt` covers distributed files except itself. `review/verify_package.py` checks it using Python's standard library. Installed dependencies, new user outputs and Python caches are excluded from the distribution.

A hash manifest, passing program checks, source matches and RPC/Explorer agreement do not independently authenticate an Axyl consensus certificate. They do not establish real backing, solvency or production withdrawals. The project's primary references and links are in `article.md`; review findings and scope are in `review/REVIEW.md`.
