# RestoreProof

**Does your backup actually restore?** Point RestoreProof at a ZIP backup and get a one-page HTML report with a clear verdict: `RESTORABLE`, `CORRUPT`, or `INCONCLUSIVE`.

Built for sysadmins, homelabbers and small MSPs who need proof — not just a green checkmark in a backup console.

[![Download Free](https://img.shields.io/badge/Download-Free%20Edition-blue)](https://github.com/klebercete/restoreproof/releases/latest)
[![Pro $29](https://img.shields.io/badge/Pro-%2429%20one--time-green)](https://kleber-forge.lemonsqueezy.com/checkout/buy/XXXXX)

## Why

A backup job that says "completed" only proves the file was written. It does not prove the archive opens, the paths are safe, or the files are readable. RestoreProof reads every entry, extracts it to an isolated temp directory, computes its SHA-256, and tells you what would really happen on restore day.

## What it checks

- The ZIP opens and every entry decompresses.
- Entry paths cannot escape the extraction directory (zip-slip protection).
- A SHA-256 hash per file, with an optional JSON manifest for archiving.
- Zero-byte files, reported as findings (a backup can "restore" and still be useless).
- Per-entry size limit (default 2 GB) plus total budgets — 20 GiB and 1,000,000 entries per archive — so a zip bomb cannot fill your disk or burn unbounded time.
- Open-time honesty: the ZIP index (central directory) is loaded into RAM when the archive opens — roughly 100 bytes per entry — before the entry budget applies. The budgets cap processing, not that first load. Extracted files are deleted right after hashing, so peak temp usage is the largest entry, not the archive sum.

## What it does **not** do

- It never executes anything from the backup. No scripts, no binaries, no network.
- It does not read TAR, RAR, 7z, or proprietary backup stores (Veeam, Acronis, etc.). ZIP only in v1.
- It does not verify live databases or application consistency — it verifies the archive.

## Usage

Single backup:

```powershell
verify.exe --backup C:\backups\week-42.zip
```

With a hash manifest for your records:

```powershell
verify.exe --backup C:\backups\week-42.zip --out week-42.html --manifest week-42.sha256.json
```

Batch mode (Pro):

```powershell
verify-pro.exe --dir D:\backups --out D:\reports
```

Batch mode verifies every `.zip` in the folder, writes `<name>-report.html` for each backup plus a `summary.html`, and never stops on the first broken archive. `--out` is a folder in batch mode (default `restoreproof-reports`). `--manifest` is not available with `--dir`; each backup already has its own report.

| Flag | Default | Description |
|---|---|---|
| `--backup` | — | Path to the `.zip` to verify (single mode) |
| `--dir` | — | Folder of `.zip` files to verify (Pro, batch mode) |
| `--out` | `report.html` / `restoreproof-reports` | Report file in single mode, output folder in batch mode |
| `--manifest` | — | Optional JSON output with per-file hashes (single mode) |
| `--max-entry-mb` | `2048` | Max uncompressed size per entry, in MB |
| `--version` | — | Print version and edition, then exit |

### Exit codes

| Code | Meaning |
|---|---|
| `0` | Restorable, no findings |
| `1` | Corrupt archive or execution error |
| `2` | Wrong usage |
| `3` | Restorable with findings (zero-byte files) or inconclusive (empty archive) |

In batch mode the codes aggregate: any corrupt or unreadable archive gives `1`, otherwise any zero-byte finding or inconclusive archive gives `3`, otherwise `0`.

Exit codes make it usable in a scheduled task or CI job:

```powershell
verify.exe --backup D:\backups\daily.zip --out D:\reports\daily.html
if ($LASTEXITCODE -ne 0) { Send-MailMessage -Subject "Backup verify failed" ... }
```

## Requirements

Windows 10/11 or Windows Server 2016+, x64. No installer, no runtime, no network access. Single `.exe`.

## Editions

| | Free | Pro |
|---|---|---|
| Verify one backup per run | Yes | Yes |
| SHA-256 per file + JSON manifest | Yes | Yes |
| HTML report | Watermarked | Clean |
| Batch mode (`--dir`) | — | Yes |
| Price | Free | USD 29, one-time |

The Free edition is a full verifier, not a demo: one backup per run, no expiry. The Pro edition adds batch mode and removes the footer watermark. Both use the same engine and the same reports.

## Verification hash (0.2.1)

```
312bb1a20323060fcbaeafc84385a7d73309ea56c1c87f99299eb5fd76aa8816
```

Verify your download with `Get-FileHash .\restoreproof-0.2.1-free-windows-amd64.zip -Algorithm SHA256`.

## License

Proprietary. See `LICENSE.txt`. The Free edition may be used for personal or commercial internal purposes; it may not be redistributed or resold.

## Support

Reply to your purchase receipt for the fastest answer.
```