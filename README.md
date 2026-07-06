<<<<<<< HEAD
<div align="center">

<img src="https://img.shields.io/badge/Rust-1.75%2B-orange?style=flat-square&logo=rust" />
<img src="https://img.shields.io/badge/Platform-Windows%20Only-blue?style=flat-square&logo=windows" />
<img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" />
<img src="https://img.shields.io/badge/Status-V1%20MVP-red?style=flat-square" />
<img src="https://img.shields.io/github/languages/top/aymaan-balbale/folsec_auditorV1?style=flat-square" />

# NTFS Auditor
=======
>>>>>>> aec8b23 (tune: optimize risk scoring to prevent alert fatigue on read-only auth user shares)

**NTFS / ACL Risk Auditor — Point-in-Time Gap Analyzer for Enterprise File Servers**

A single, statically linked `.exe` written in Rust. Scans NTFS directory trees at high speed using direct Win32 API calls, then produces a self-contained HTML report exposing over-permissive ACEs, inheritance breaks, orphaned SIDs, and SIEM blind spots — with a dry-run PowerShell remediation script included.

<<<<<<< HEAD
Built as an open-source gap analysis tool for [FolSec](https://folsec.com), a platform providing continuous NTFS governance and real-time audit visibility. This tool shows you what's broken **right now**. FolSec tells you the moment it changes.

[Quick Start](#quick-start) · [What It Finds](#what-it-finds) · [Build from Source](#building-from-source) · [Anomaly Simulator](#anomaly-simulator----simulate-anomaly) · [Architecture](#architecture)

</div>
=======
Built as an open-source lead-generation tool for [NTFS Auditor](https://NTFS Auditor.com) — a platform providing continuous NTFS governance, real-time audit visibility, and anomaly detection. This tool gives IT administrators a point-in-time "what's broken right now" picture. NTFS Auditor tells you the moment it changes.
>>>>>>> aec8b23 (tune: optimize risk scoring to prevent alert fatigue on read-only auth user shares)

---

## What It Finds

| Risk | Severity | Description |
|------|----------|-------------|
| Over-permissive ACE | `HIGH` / `MEDIUM` | `Everyone`, `Authenticated Users`, or `BUILTIN\Users` with write / modify / full control |
| Inheritance break | `MEDIUM` | Folder explicitly blocks inherited permissions — creates a hidden permission island invisible to top-down audits |
| Orphaned SID | `LOW` | ACE references a SID that cannot be resolved — typically a deleted AD user or group |
| Null DACL | `CRITICAL` | No DACL present at all — Windows interprets this as implicit allow-all for every user on the system |
| Anomaly simulated | `CRITICAL` | 2,500 rapid ACL changes went undetected — your SIEM has no real-time NTFS visibility |

---

## Output

**1. Self-contained HTML report**
A single `.html` file with an embedded interactive findings table, severity filters, path search, and a FolSec CTA. No server, no CDN dependency after first load. Safe to email or copy to USB for air-gapped review.

**2. Dry-run PowerShell remediation script** (`scripts/remediate_dry_run.ps1`)
Every finding maps to a documented `Write-Host "Would remove..."` block. The script never calls `Set-Acl` or `icacls` with side effects — it is a change management artifact, not an execution script.

---

## Quick Start

```powershell
<<<<<<< HEAD
# Scan a UNC share
folsec-auditor.exe scan --root "\\fileserver01\HR" --output report.html

# Scan with a dry-run remediation script
folsec-auditor.exe scan --root "C:\DataFiles" --output report.html --remediation scripts\remediate_dry_run.ps1

# Fast triage — limit depth on a large share
folsec-auditor.exe scan --root "D:\Projects" --output report.html --max-depth 4

# Expose SIEM blind spots with the anomaly simulator
folsec-auditor.exe scan --root "C:\DataFiles" --output report.html --simulate-anomaly

# Reduce CPU impact on a production server
folsec-auditor.exe scan --root "\\nas\share" --output report.html --threads 4
=======
# Scan a share and generate the HTML report
NTFS Auditor-auditor.exe scan --root "\\fileserver01\HR" --output report.html

# Also generate the dry-run remediation script
NTFS Auditor-auditor.exe scan --root "C:\DataFiles" --output report.html --remediation fix.ps1

# Limit depth for a fast triage pass
NTFS Auditor-auditor.exe scan --root "D:\Projects" --output report.html --max-depth 4

# Run the anomaly simulator (see Anomaly Simulator section below)
NTFS Auditor-auditor.exe scan --root "C:\DataFiles" --output report.html --simulate-anomaly

# Restrict CPU usage on production servers
NTFS Auditor-auditor.exe scan --root "\\nas\share" --output report.html --threads 4
>>>>>>> aec8b23 (tune: optimize risk scoring to prevent alert fatigue on read-only auth user shares)
```

---

## Real Scan Output

The screenshots below are from a live run against a test sandbox (`C:\FolSec_Mega_Sandbox`) — not fabricated output.

### Scan startup + console summary

```
FolSec NTFS Auditor v0.1.0 — Starting scan
Root:       C:\FolSec_Mega_Sandbox
Max depth:  unlimited
Threads:    auto (all CPUs)
Output:     C:\Users\Aryaan Balbale\mega_report.html

[00:00:00] Scanned 0 paths — Scan complete — 126 directories examined.

── Scan Complete ───────────────────────────────────────────
  Paths scanned:   126
  Total findings:  544
  CRITICAL:        17
  HIGH:            184
  MEDIUM:          330
  LOW:             13
  Scan errors:     0
────────────────────────────────────────────────────────────

✓ HTML report:    C:\Users\Aryaan Balbale\mega_report.html
✓ Dry-run script: C:\Users\Aryaan Balbale\mega_remediation.ps1

Open the HTML report in any browser to begin triage.
For continuous monitoring: https://folsec.com
```

126 directories. 544 findings. 0 scan errors. Output in under a second.

---

### Top 5 riskiest folders (remediation script header)

The generated `.ps1` opens with a ranked risk leaderboard before listing individual actions:

```
FolSec NTFS Auditor — Remediation Script

Scan Root:        C:\FolSec_Mega_Sandbox
Total Findings:   544
Actionable Fixes: 544
Mode:             DRY-RUN

🔥 TOP 5 RISKIEST FOLDERS

#1 [Score: 29] (5 finding(s))
   C:\FolSec_Mega_Sandbox\Engineering\Top_Secret

#2 [Score: 29] (5 finding(s))
   C:\FolSec_Mega_Sandbox\Engineering\Top_Secret\Final

#3 [Score: 29] (5 finding(s))
   C:\FolSec_Mega_Sandbox\Engineering\Top_Secret\Raw_Data

#4 [Score: 29] (6 finding(s))
   C:\FolSec_Mega_Sandbox\Finance\Project_Beta\Drafts

#5 [Score: 29] (7 finding(s))
   C:\FolSec_Mega_Sandbox\Finance\Top_Secret\Approvals
```

The risk score is computed from finding count × severity weight. Highest-impact folders surface first, so a remediation team knows where to start without reading all 544 entries.

---

### Dry-run block output (per folder)

Each folder gets a numbered block. Over-permissive ACEs, inheritance breaks, and orphaned SIDs are each a distinct action:

```
[DRY-RUN BLOCK #2] Path: C:\FolSec_Mega_Sandbox\Engineering (4 action(s))
  {1}. Would REMOVE explicit Allow ACE for 'Everyone'
         — Read, Write Data, Append Data, Execute, Delete, Read Control (0x001301BF)
  {2}. Would REMOVE explicit Allow ACE for 'BUILTIN\Users'
         — Read, Execute, Read Control (0x001200A9)
  {3}. Would REMOVE explicit Allow ACE for 'NT AUTHORITY\Authenticated Users'
         — Read, Write Data, Append Data, Execute, Delete, Read Control (0x001301BF)
  {4}. Would REMOVE explicit Allow ACE for 'NT AUTHORITY\Authenticated Users'
         — Delete (0xE0010000)

[DRY-RUN BLOCK #4] Path: C:\FolSec_Mega_Sandbox\Engineering\Project_Alpha\Approvals (6 action(s))
  {1}. Would REMOVE explicit Allow ACE for 'Everyone'
         — Read, Write Data, Append Data, Execute, Delete, Read Control (0x001301BF)
  {2}. Would RE-ENABLE INHERITANCE (had_protected_copy=true)          ← inheritance break
  {3}. Would REMOVE explicit Allow ACE for 'BUILTIN\Users'
         — Read, Execute, Read Control (0x001200A9)
  {4}. Would REMOVE explicit Allow ACE for 'NT AUTHORITY\Authenticated Users'
         — Read, Write Data, Append Data, Execute, Delete, Read Control (0x001301BF)
  {5}. Would REMOVE explicit Allow ACE for 'NT AUTHORITY\Authenticated Users'
         — Delete (0xE0010000)
  {6}. Would REMOVE orphaned SID: S-1-5-21-461125421-364847304-616652533-1000
```

Colours in terminal output: yellow = ACE removal, magenta = inheritance break, dark yellow = orphaned SID.

---

### Execution summary (end of script run)

```
Execution Summary
─────────────────────────────────────────
Total actions:   535 (actionable findings)
Processed:       535
Skipped:         0
Mode:            DRY-RUN (no changes were made)

Next steps:
  1. Review each action above with your AD team lead.
  2. Submit this output through your change management process.
  3. Back up current ACLs:  icacls '<root>' /save acl_backup.txt /T
  4. Set $LiveMode = $true and re-run to apply (with -WhatIf safety).
  5. Remove -WhatIf from Set-Acl calls only after final approval.

Transcript saved to: C:\Users\Aryaan Balbale\FolSec_Remediation_20260624_164009.log
Report & docs: https://folsec.com/remediation
```

The script auto-saves a timestamped transcript of its own output to disk — useful for change management paper trails.

---

## CLI Reference

```
<<<<<<< HEAD
folsec-auditor.exe <COMMAND>

Commands:
  scan     Scan a directory tree for NTFS ACL risks and generate a report
  version  Print version and build information

scan options:
  -r, --root <PATH>           Root path to scan — local (C:\) or UNC (\\server\share)
  -o, --output <FILE>         HTML report output path [default: folsec_report.html]
=======
NTFS Auditor-auditor.exe scan [OPTIONS]

Options:
  -r, --root <PATH>           Root path to scan (local or UNC)
  -o, --output <FILE>         HTML report output path [default: NTFS Auditor_report.html]
>>>>>>> aec8b23 (tune: optimize risk scoring to prevent alert fatigue on read-only auth user shares)
      --remediation <FILE>    Dry-run PowerShell script output path (optional)
      --max-depth <N>         Max recursion depth; 0 = unlimited [default: 0]
      --follow-symlinks       Follow symbolic links
                              ⚠ Use with caution on SMB shares with junction points
      --skip-access-denied    Skip access-denied paths silently instead of logging them
      --simulate-anomaly      Run the ransomware ACL cycling simulator (see below)
      --threads <N>           Rayon worker threads; 0 = all logical CPUs [default: 0]
  -h, --help                  Print help
  -V, --version               Print version
```

---

## Building from Source

**Requirements:** Rust stable (`rustup`), Windows host or MSVC cross-compilation setup.

```powershell
# Clone
git clone https://github.com/aymaan-balbale/NTFS Auditor_auditorV1.git
cd NTFS Auditor_auditorV1

# Build a statically linked release binary (no MSVC runtime dependency)
$env:RUSTFLAGS = "-C target-feature=+crt-static"
cargo build --release --target x86_64-pc-windows-msvc

<<<<<<< HEAD
# Binary output
.\target\x86_64-pc-windows-msvc\release\folsec-auditor.exe
=======
# Binary location
.\target\x86_64-pc-windows-msvc\release\NTFS Auditor-auditor.exe
>>>>>>> aec8b23 (tune: optimize risk scoring to prevent alert fatigue on read-only auth user shares)
```

> **CI note:** `cargo check` and `cargo clippy` run cleanly on Linux/macOS — the Win32 API calls are gated behind `#[cfg(target_os = "windows")]`. Non-Windows builds produce an empty-findings report, which is useful for testing report output in CI.

### Release Profile

`Cargo.toml` configures the release build for a minimal, deployment-ready binary:

```toml
[profile.release]
opt-level     = 3
lto           = true        # Link-Time Optimization — reduces binary ~30%
codegen-units = 1           # Max optimization per compilation unit
strip         = true        # Strip debug symbols
panic         = "abort"     # No stack unwinding — smaller binary, faster exit
```

---

## Architecture

```
folsec_auditorV1/
├── Cargo.toml
├── LICENSE
├── scripts/
│   └── remediate_dry_run.ps1     # Auto-generated dry-run remediation script
└── src/
    ├── main.rs                   # CLI entry point (clap)
    ├── errors/
    │   └── mod.rs                # Typed error hierarchy — no panics in production
    ├── scanner/
    │   ├── mod.rs                # Parallel traversal engine (walkdir + rayon)
    │   ├── acl.rs                # Win32 DACL extraction (GetNamedSecurityInfoW)
    │   ├── sid.rs                # SID → account name (LookupAccountSidW)
    │   └── risk.rs               # Data model: RiskFinding, Severity, AuditSummary
    ├── simulator/
    │   └── mod.rs                # Ransomware ACL cycling simulator
    ├── reporter/
    │   └── mod.rs                # Self-contained HTML report generator
    └── remediation/
        └── mod.rs                # Dry-run PowerShell .ps1 generator
```

### How the Scanner Works

1. **`walkdir`** streams directory entries lazily — no full-tree RAM load, handles deeply nested SMB shares without locking memory.
2. **`rayon::par_bridge()`** turns that sequential stream into a parallel one, distributing entries across the thread pool. On a 16-core server this is 8–12× faster than single-threaded scanning.
3. **`acl.rs`** calls `GetNamedSecurityInfoW` for each directory, extracts the DACL, and iterates ACEs via `GetAce`. A `SecurityDescriptorGuard` RAII wrapper ensures `LocalFree` is always called — even if an error short-circuits traversal mid-ACE.
4. **`sid.rs`** resolves each SID to an account name via the two-call `LookupAccountSidW` pattern (size probe → allocate → resolve), with explicit `ERROR_NONE_MAPPED` handling for orphaned SIDs.
5. **`dashmap`** accumulates findings across threads without a global lock.
6. Results are sorted by severity descending, then written to the HTML report and optional `.ps1` script.

---

## Anomaly Simulator — `--simulate-anomaly`

Proves in real time whether your SIEM has NTFS audit visibility.

**What it does:**
<<<<<<< HEAD

1. Creates a hidden temporary directory (`.tmp_folsec_audit`) under the scan root.
2. Generates **50 dummy files** inside it.
3. Cycles each file's ACL **50 times** in rapid succession — **2,500 permission-change events in under 2 seconds** — replicating the ACL manipulation pattern used by ransomware before encrypting files.
4. Deletes the directory and all dummy files.
5. Produces a `CRITICAL` finding in the report: *did your SIEM alert during step 3?*
=======
1. Creates a hidden temporary directory (`.tmp_NTFS Auditor_audit`) under the scan root.
2. Generates 50 dummy files inside it.
3. Cycles each file's ACL 50 times in rapid succession — **2,500 permission-change events in under 2 seconds** — mimicking the ACL manipulation pattern used by ransomware during encryption.
4. Deletes the directory.
5. Asks: *did your SIEM alert?*
>>>>>>> aec8b23 (tune: optimize risk scoring to prevent alert fatigue on read-only auth user shares)

A correctly configured environment (Windows Security Audit Policy → "Audit Object Access", forwarding Event IDs **4656 / 4663 / 4670** to your SIEM) will fire during step 3. Silence means a blind spot.

> ⚠️ **Authorized use only.** Run this only on systems you own or have explicit written permission to test. The simulator creates and deletes only its own temporary files — no production data is read, modified, or encrypted.

---

## The Dry-Run Remediation Script

Every finding maps to a self-documenting block in the generated `.ps1`. Example output:

```powershell
# Finding #1: [HIGH] \\fileserver01\HR\Payroll
# Risk: Trustee 'Everyone' has over-permissive access (Full Control).
#       This creates a wide blast radius for ransomware or insider threats.

# LIVE COMMAND (review before use):
# icacls '\\fileserver01\HR\Payroll' /remove:g 'Everyone'
# Or with Set-Acl:
# $acl = Get-Acl -Path '\\fileserver01\HR\Payroll'
# $ace = $acl.Access | Where-Object { $_.IdentityReference -eq 'Everyone' }
# $acl.RemoveAccessRule($ace)
# Set-Acl -Path '\\fileserver01\HR\Payroll' -AclObject $acl

$ActionCount++
Write-Host "[DRY-RUN] #1: Would REMOVE ACE for 'Everyone' (Full Control) on path:" -ForegroundColor Yellow
Write-Host "  >> '\\fileserver01\HR\Payroll'" -ForegroundColor Yellow
```

The script never calls `Set-Acl` or `icacls` with side effects. Each block includes the live command as a comment so the change management team knows exactly what production execution would do.

---

## Core Dependencies

| Crate | Version | Purpose |
|-------|---------|---------|
| `windows` | 0.58 | Win32 API bindings: `GetNamedSecurityInfoW`, `GetSecurityDescriptorDacl`, `LookupAccountSidW` |
| `walkdir` | 2.5 | Lazy depth-first directory traversal with graceful error handling |
| `rayon` | 1.10 | Work-stealing thread pool for parallel ACL extraction |
| `clap` | 4.5 | Typed CLI argument parsing with derive macros and `--help` generation |
| `serde` + `serde_json` | 1.0 | Serializes findings into the HTML report's embedded JSON data island |
| `dashmap` | 6.0 | Lock-minimizing concurrent HashMap for cross-thread accumulation |
| `thiserror` | 1.0 | Typed error enum — every failure mode is named, never `.unwrap()` |
| `indicatif` | 0.17 | Live progress spinner during long scans |
| `chrono` | 0.4 | Report timestamps |

---

## Why Not Just Use `icacls` / `Get-Acl`?

| | `icacls` / `Get-Acl` | NTFS Auditor-auditor |
|---|---|---|
| Speed on 500k+ dirs | Minutes–hours (single-threaded) | Seconds–minutes (all CPU cores) |
| Risk classification | None — raw output only | Automatic severity scoring |
| Orphaned SID detection | Manual cross-reference required | Built-in via `LookupAccountSidW` |
| Inheritance break detection | Requires custom scripting | Automatic via SD control flags |
| Report output | Terminal text / CSV | Filterable interactive HTML |
| Remediation script | Manual authoring | Auto-generated dry-run `.ps1` |
| SIEM gap detection | Not possible | Anomaly simulator built-in |
| Deployment | Requires PowerShell / RSAT | Single `.exe`, zero dependencies |

---

## Known Limitations (V1)

- **Windows only.** NTFS ACL scanning uses Win32 APIs unavailable on other platforms.
- **Directories only.** Files inherit permissions from their parent folder — scanning every file would multiply Win32 API calls with minimal additional signal.
- **DACL only.** No SACL (audit policy), owner SID, or group SID analysis in V1.
- **Anomaly simulator spawns `icacls` child processes.** V2 will use `SetNamedSecurityInfoW` directly for sub-500ms cycling and no process spawn overhead.
- **No AD connectivity.** Orphaned SID detection is local — the tool cannot distinguish a deleted account from a merely disabled one without LDAP access.
- **No incremental/delta scanning.** Every run is a full point-in-time pass. Continuous monitoring requires [FolSec](https://folsec.com).

---

## Contributing

Issues and pull requests are welcome. For significant changes, open an issue first.

```bash
# Compile check + lint (works on Linux/macOS)
cargo check
cargo clippy -- -D warnings

# Run unit tests
cargo test
```

Please keep all Win32 API calls inside `#[cfg(target_os = "windows")]` guards so CI on Linux remains clean.

---

## License

MIT — see [LICENSE](LICENSE).

---

<<<<<<< HEAD
<div align="center">

*folsec-auditor surfaces what exists right now.*
*[FolSec](https://folsec.com) tells you the moment anything changes.*

</div>
=======
*NTFS Auditor-auditor surfaces what exists right now. [NTFS Auditor](https://NTFS Auditor.com) tells you the moment anything changes.*
>>>>>>> aec8b23 (tune: optimize risk scoring to prevent alert fatigue on read-only auth user shares)
