Readme · MDCopy<div align="center">
<img src="https://img.shields.io/badge/Rust-1.75%2B-orange?style=flat-square&logo=rust" />
<img src="https://img.shields.io/badge/Platform-Windows%20Only-blue?style=flat-square&logo=windows" />
<img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" />
<img src="https://img.shields.io/badge/Status-V1%20MVP-red?style=flat-square" />
<img src="https://img.shields.io/github/languages/top/aymaan-balbale/folsec_auditorV1?style=flat-square" />
folsec-auditor

NTFS / ACL Risk Auditor — Point-in-Time Gap Analyzer for Enterprise File Servers

A single, statically linked .exe written in Rust. Scans NTFS directory trees at high speed using direct Win32 API calls, then produces a self-contained HTML report exposing over-permissive ACEs, inheritance breaks, orphaned SIDs, and SIEM blind spots — with a dry-run PowerShell remediation script included.

Built as an open-source gap analysis tool for FolSec, a platform providing continuous NTFS governance and real-time audit visibility. This tool shows you what's broken right now. FolSec tells you the moment it changes.

Quick Start · What It Finds · Build from Source · Anomaly Simulator · Architecture

</div>

What It Finds

RiskSeverityDescriptionOver-permissive ACEHIGH / MEDIUMEveryone, Authenticated Users, or BUILTIN\Users with write / modify / full controlInheritance breakMEDIUMFolder explicitly blocks inherited permissions — creates a hidden permission island invisible to top-down auditsOrphaned SIDLOWACE references a SID that cannot be resolved — typically a deleted AD user or groupNull DACLCRITICALNo DACL present at all — Windows interprets this as implicit allow-all for every user on the systemAnomaly simulatedCRITICAL2,500 rapid ACL changes went undetected — your SIEM has no real-time NTFS visibility


Output

1. Self-contained HTML report
A single .html file with an embedded interactive findings table, severity filters, path search, and a FolSec CTA. No server, no CDN dependency after first load. Safe to email or copy to USB for air-gapped review.

2. Dry-run PowerShell remediation script (scripts/remediate_dry_run.ps1)
Every finding maps to a documented Write-Host "Would remove..." block. The script never calls Set-Acl or icacls with side effects — it is a change management artifact, not an execution script.


Quick Start

powershell# Scan a UNC share
folsec-auditor.exe scan --root "\\fileserver01\HR" --output report.html

# Scan with a dry-run remediation script
folsec-auditor.exe scan --root "C:\DataFiles" --output report.html --remediation scripts\remediate_dry_run.ps1

# Fast triage — limit depth on a large share
folsec-auditor.exe scan --root "D:\Projects" --output report.html --max-depth 4

# Expose SIEM blind spots with the anomaly simulator
folsec-auditor.exe scan --root "C:\DataFiles" --output report.html --simulate-anomaly

# Reduce CPU impact on a production server
folsec-auditor.exe scan --root "\\nas\share" --output report.html --threads 4


CLI Reference

folsec-auditor.exe <COMMAND>

Commands:
  scan     Scan a directory tree for NTFS ACL risks and generate a report
  version  Print version and build information

scan options:
  -r, --root <PATH>           Root path to scan — local (C:\) or UNC (\\server\share)
  -o, --output <FILE>         HTML report output path [default: folsec_report.html]
      --remediation <FILE>    Dry-run PowerShell script output path (optional)
      --max-depth <N>         Max recursion depth; 0 = unlimited [default: 0]
      --follow-symlinks       Follow symbolic links
                              ⚠ Use with caution on SMB shares with junction points
      --skip-access-denied    Skip access-denied paths silently instead of logging them
      --simulate-anomaly      Run the ransomware ACL cycling simulator (see below)
      --threads <N>           Rayon worker threads; 0 = all logical CPUs [default: 0]
  -h, --help                  Print help
  -V, --version               Print version


Building from Source

Requirements: Rust stable (rustup), Windows host or MSVC cross-compilation setup.

powershell# Clone
git clone https://github.com/aymaan-balbale/folsec_auditorV1.git
cd folsec_auditorV1

# Build a statically linked release binary (no MSVC runtime dependency)
$env:RUSTFLAGS = "-C target-feature=+crt-static"
cargo build --release --target x86_64-pc-windows-msvc

# Binary output
.\target\x86_64-pc-windows-msvc\release\folsec-auditor.exe


CI note: cargo check and cargo clippy run cleanly on Linux/macOS — the Win32 API calls are gated behind #[cfg(target_os = "windows")]. Non-Windows builds produce an empty-findings report, which is useful for testing report output in CI.



Release Profile

Cargo.toml configures the release build for a minimal, deployment-ready binary:

toml[profile.release]
opt-level     = 3
lto           = true        # Link-Time Optimization — reduces binary ~30%
codegen-units = 1           # Max optimization per compilation unit
strip         = true        # Strip debug symbols
panic         = "abort"     # No stack unwinding — smaller binary, faster exit


Architecture

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

How the Scanner Works


walkdir streams directory entries lazily — no full-tree RAM load, handles deeply nested SMB shares without locking memory.
rayon::par_bridge() turns that sequential stream into a parallel one, distributing entries across the thread pool. On a 16-core server this is 8–12× faster than single-threaded scanning.
acl.rs calls GetNamedSecurityInfoW for each directory, extracts the DACL, and iterates ACEs via GetAce. A SecurityDescriptorGuard RAII wrapper ensures LocalFree is always called — even if an error short-circuits traversal mid-ACE.
sid.rs resolves each SID to an account name via the two-call LookupAccountSidW pattern (size probe → allocate → resolve), with explicit ERROR_NONE_MAPPED handling for orphaned SIDs.
dashmap accumulates findings across threads without a global lock.
Results are sorted by severity descending, then written to the HTML report and optional .ps1 script.



Anomaly Simulator — --simulate-anomaly

Proves in real time whether your SIEM has NTFS audit visibility.

What it does:


Creates a hidden temporary directory (.tmp_folsec_audit) under the scan root.
Generates 50 dummy files inside it.
Cycles each file's ACL 50 times in rapid succession — 2,500 permission-change events in under 2 seconds — replicating the ACL manipulation pattern used by ransomware before encrypting files.
Deletes the directory and all dummy files.
Produces a CRITICAL finding in the report: did your SIEM alert during step 3?


A correctly configured environment (Windows Security Audit Policy → "Audit Object Access", forwarding Event IDs 4656 / 4663 / 4670 to your SIEM) will fire during step 3. Silence means a blind spot.


⚠️ Authorized use only. Run this only on systems you own or have explicit written permission to test. The simulator creates and deletes only its own temporary files — no production data is read, modified, or encrypted.




The Dry-Run Remediation Script

Every finding maps to a self-documenting block in the generated .ps1. Example output:

powershell# Finding #1: [HIGH] \\fileserver01\HR\Payroll
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

The script never calls Set-Acl or icacls with side effects. Each block includes the live command as a comment so the change management team knows exactly what production execution would do.


Core Dependencies

CrateVersionPurposewindows0.58Win32 API bindings: GetNamedSecurityInfoW, GetSecurityDescriptorDacl, LookupAccountSidWwalkdir2.5Lazy depth-first directory traversal with graceful error handlingrayon1.10Work-stealing thread pool for parallel ACL extractionclap4.5Typed CLI argument parsing with derive macros and --help generationserde + serde_json1.0Serializes findings into the HTML report's embedded JSON data islanddashmap6.0Lock-minimizing concurrent HashMap for cross-thread accumulationthiserror1.0Typed error enum — every failure mode is named, never .unwrap()indicatif0.17Live progress spinner during long scanschrono0.4Report timestamps


Why Not Just Use icacls / Get-Acl?

icacls / Get-Aclfolsec-auditorSpeed on 500k+ dirsMinutes–hours (single-threaded)Seconds–minutes (all CPU cores)Risk classificationNone — raw output onlyAutomatic severity scoringOrphaned SID detectionManual cross-reference requiredBuilt-in via LookupAccountSidWInheritance break detectionRequires custom scriptingAutomatic via SD control flagsReport outputTerminal text / CSVFilterable interactive HTMLRemediation scriptManual authoringAuto-generated dry-run .ps1SIEM gap detectionNot possibleAnomaly simulator built-inDeploymentRequires PowerShell / RSATSingle .exe, zero dependencies


Known Limitations (V1)


Windows only. NTFS ACL scanning uses Win32 APIs unavailable on other platforms.
Directories only. Files inherit permissions from their parent folder — scanning every file would multiply Win32 API calls with minimal additional signal.
DACL only. No SACL (audit policy), owner SID, or group SID analysis in V1.
Anomaly simulator spawns icacls child processes. V2 will use SetNamedSecurityInfoW directly for sub-500ms cycling and no process spawn overhead.
No AD connectivity. Orphaned SID detection is local — the tool cannot distinguish a deleted account from a merely disabled one without LDAP access.
No incremental/delta scanning. Every run is a full point-in-time pass. Continuous monitoring requires FolSec.



Contributing

Issues and pull requests are welcome. For significant changes, open an issue first.

bash# Compile check + lint (works on Linux/macOS)
cargo check
cargo clippy -- -D warnings

# Run unit tests
cargo test

Please keep all Win32 API calls inside #[cfg(target_os = "windows")] guards so CI on Linux remains clean.


License

MIT — see LICENSE.


<div align="center">
folsec-auditor surfaces what exists right now.
FolSec tells you the moment anything changes.

</div>
