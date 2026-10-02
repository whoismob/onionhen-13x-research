# OnionHEN 13.x port: findings (UNTESTED ON HARDWARE)

Patch: 0001-libhijacker-add-PS5-13.x-allproc-offsets... (applies to aydencharles/onionHEN @ b23ffe6)

## Done
- source/libhijacker/source/offsets.cpp: add V1300/V1320/V1340/V1342/V1360 and allproc:
  13.00,13.20 -> 0x28C5E00 ; 13.40,13.42,13.60 -> 0x28C9E80
  (values from EchoStretch/kstuff-lite prosper0gdb/offsets/13_*.h)
- Verified: file compiles (g++ -std=c++20, stubbed sysctl/log) and a harness faking kern.sdk_version
  returns the right values; before patch 13.60 returned -1 (Unsupported firmware).

## Open items before it can run on 13.x
1. source/shellui/src/dynamic_ui_xml.cpp FirmwareProfile::for_system_version only enables
   the legacy settings UI for 2.30..12.ffffff; 13.x returns {} (feature off). Need someone on 13.x
   to confirm whether the 12.x settings-page layout still applies before widening it.
2. security_flags/qa_flags/utoken_flags/root_vnode tables stop at 10.60 (11.x/12.x were never added,
   so these are presumably unused on 11+). Confirm they are not needed on 13.x.
3. shellui/src/homeui_top_nav_patch.cpp: HomeUI patch signatures are per-firmware; unsupported
   builds are skipped with a warning, so toolbox injection may be missing on 13.x.
4. Needs a loader that provides kernel R/W on 13.x (Relapse) + kstuff-lite 1.11 beta.
5. Official etaHEN 2.5B libhijacker tables stop at 10.60; 11.x-13.x there are a separate, larger job.
6. 13.42 kern.sdk_version value is assumed to be 0x13420000 (kstuff uses 0x1342): verify.
