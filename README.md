# OnionHEN 13.60 Research

Firmware: PS5 13.60 (0x1360)
Profile: 13.60 NPXS40002 HomeUI (hbc_version 89, file_length 0x1b7240)
Source hash: `f73c3e0102be92a4558baa510bd00c17f6fe5f96`
Build: OnionHEN with 13.x allproc offsets (kstuff-lite v1.11) + RNPS decrypt dump hook
Files: patches (0001, 0002), profile notes, build artifacts (ELF), logs, captured bundle fingerprints
No personal/user identifying information included.

## DPI v2 external plugin

`dpiv2-13.60-1.00.elf` is a build of the upstream
[OnionHEN DPI v2 plugin](https://github.com/OnionBuddies/onionHEN-dpiv2-plugin)
(GPLv3, unmodified source): a browser-based remote `.pkg` installer. Built and
hardware-tested on 13.60.

See [DPI-V2-PLUGIN.md](DPI-V2-PLUGIN.md) for install steps, port notes,
portability limits, and the security warning — the plugin has no
authentication and runs with SYSTEM authid.
