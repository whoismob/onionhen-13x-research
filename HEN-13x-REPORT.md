# OnionHEN 13.60 / PS5 Firmware 13.60 Report
Prepared for aydencharles/onionHEN maintainers

## User / System
- Firmware: 13.60 (confirmed in OnionHEN.log: "starting OnionHEN (0x1360)")
- Build: patched with 13.x allproc offsets (kstuff-lite v1.11, covers 1.00-13.60)
- Profile: 13.60 entry added to profiles.inc (best-effort, source_hash f73c3e..., file_length 0x1b7240, hbc_version 89)
- Build artifacts served: http://100.100.146.108:8765/

## Key SHA-256 Hashes
OnionHEN-13x-fixed.elf: ca4818e9bc3ad9ab (build with profile patch, 4.6MB)
OnionHEN-13x-dump2.elf: f155f6df0c5c33e774ccfc22147b8b9332d610e7e767fa041f26410224c9232f
upload verified full-size (4,608,016 bytes, no truncation)

## Build / Patch Files (in ~/workspace/downloads/)
- 0001-*.patch (allproc offset patch: V1300-V1360 -> offsets.cpp)
- 0002-*.patch (RNPS runtime dump + SHELL_DEBUG notification)
- NOTES-13x-onionhen.md

## Decrypt Capture Results
- RNPS hook installed and firing (user confirmed notification visible)
- 7 full 1.8MB bundle dumps in /data and /user/data (file_length 0x1b7240, matches on-disk bundle)
- Source hash from captured HBC: f73c3e0102be92a4558baa510bd00c17f6fe5f96
- HBC version: 89
- Current profile entry uses these fingerprints but toolbox remains invisible (profile may still need exact offset tuning from disassembled HBC)

## Unresolved
- 13.60 profile offsets derived from 12.20 (best-effort), not from full decrypted HBC disassembly
- Full HBC offset derivation needs decrypt bundle analysis beyond this session's capabilities
- Suggest upstream review of captured dumps at /data/rnps_dump_01_3664512.bin etc.

## Commands / Logs
- Build log: ~/workspace/downloads/build-profile.log
- OnionHEN.log (debug): ~/workspace/downloads/OnionHEN.log.new (restart at 61s uptime, toolbox online for ShellUI pid=59, no crash)
- FTP access: anonymous@10.10.0.239:2121 (/data/OnionHEN/, /data/pldmgr/payloads/OnionHEN/)
