# DPI v2 — OnionHEN external plugin build

Browser-based remote `.pkg` installer for OnionHEN. Drop a package on a web page
from any device on the local network; it transfers to the console in chunks and
hands off to the PS5's own system installer.

Upstream source (GPLv3):
<https://github.com/OnionBuddies/onionHEN-dpiv2-plugin>

## Artifact

| | |
|---|---|
| File | `dpiv2-13.60-1.00.elf` |
| Size | 325,848 bytes |
| sha256 | `b5ed9abec10bfd6fa33b8dd263c98ac7e55211f540da86a9c8719fb445db6776` |
| Plugin ID | `DPIV00001` |
| Plugin version | 1.00 |
| Built | 2026-10-03, from upstream commit `4e80389` |

## Test status

Verified on PS5 firmware 13.60 (0x1360):

- `[plugins] started DPIV00001 1.00 (DPI v2) pid=105`, `running=1 failed=0`
- `DPIV00001.log`: `ready; API=9090 WebUI=12800 language=en`
- `DPIV00001-server.log`: `pkg-server 1.0.0 starting (fw major 13)`,
  `sceAppInstUtilInitialize -> 0x00000000`
- WebUI served HTTP 200 on `:12800`; `/api/config` returned
  `{"ok":true,"api_port":9090,"webui_port":12800}`

Hardware-tested on 13.60 only. Other firmware is untested.

## Security — read before using

- **No authentication.** The plugin accepts package uploads from any client
  that can reach it. Anyone on the same network can push a package to the
  console.
- **Runs with SYSTEM authid** and escalates to root plus sandbox escape
  (`escalated: root + sandbox escape + SYSTEM_AUTHID`).
- Listens on all console interfaces, so do not expose `9090` or `12800` to the
  internet, and do not port-forward them.
- Not security-audited.

Use on a trusted local network only, and stop the service when it is not
needed. OnionHEN is an unofficial homebrew project, not affiliated with Sony
Interactive Entertainment. Use at your own risk.

## Portability

The ELF contains no firmware offsets or hardcoded kernel addresses. It works
through Sony's public SCE APIs (`sceAppInstUtilInitialize`,
`sceAppInstUtilInstallByPackage`, `sceAppInstUtilGetInstallStatus`) under SYSTEM
authid. Firmware major version is detected and logged but never branched on, so
there is no per-firmware code path to break.

Practical limits:

- **PS5 only.** Built for `x86_64-sie-ps5`; will not load on PS4. (PS4 `.pkg`
  *content* is supported — that is not the same as running on PS4 hardware.)
- **Not PS5 Pro** — different SoC.
- **Requires a jailbroken console** running an OnionHEN build with external
  plugin support.
- The host OnionHEN plugin SDK must be compatible with upstream's pinned
  revision (`0cf4346`). An older custom build may discover the ELF and then
  fail to start it.
- Plugin ID `DPIV00001` must be free on the target console.

## Install

Upload the ELF atomically as `/data/OnionHEN/plugins/DPIV00001.installing`,
then rename to `/data/OnionHEN/plugins/DPIV00001.elf` after the transfer
completes. OnionHEN discovers it automatically.

Over anonymous FTP (ftpsrv payload on the console):

```python
import ftplib, hashlib, io

LOCAL = "dpiv2-13.60-1.00.elf"
DEST  = "/data/OnionHEN/plugins/DPIV00001"
TMP   = DEST + ".installing"

ftp = ftplib.FTP()
ftp.connect("<PS5-IP>", 2121, timeout=30)
ftp.login("anonymous", "")
ftp.voidcmd("TYPE I")

with open(LOCAL, "rb") as fh:
    ftp.storbinary(f"STOR {TMP}", fh, blocksize=64 * 1024)

# verify before activating: size + hash pulled back off the console
local_sha = hashlib.sha256(open(LOCAL, "rb").read()).hexdigest()
remote = hashlib.sha256()
ftp.retrbinary(f"RETR {TMP}", remote.update, 64 * 1024)
assert ftp.size(TMP) == len(open(LOCAL, "rb").read())
assert remote.hexdigest() == local_sha, "hash mismatch, not activating"

ftp.rename(TMP, f"{DEST}.elf")
ftp.quit()
```

Verify both size and hash. A truncated upload yields a plugin that is
discovered but never starts.

Then open **★ OnionHEN Plugins → DPI v2** and enable the server.

Note: the autostart gate is a separate `DPIV00001.elf.auto_start` marker file
that OnionHEN writes when you enable the plugin from the UI. The `.ini` holds
only `enabled`, `api_port`, and `webui_port` — editing it alone will not start
the plugin.

## Ports

| Port | Serves |
|---|---|
| `9090` | DPI transfer API |
| `12800` | WebUI and SSE progress stream |

The two must differ. Config lives in
`/data/OnionHEN/plugins/DPIV00001.ini`.

Then open `http://<PS5-IP>:12800` from any device on the same network.

## On-console paths

| Path | Purpose |
|---|---|
| `/data/OnionHEN/pkgs/` | staged package uploads, retained for retry/reuse |
| `/data/OnionHEN/plugins/DPIV00001.ini` | enabled state and listener ports |
| `/data/OnionHEN/DPIV00001.log` | plugin lifecycle and dynamic UI errors |
| `/data/OnionHEN/DPIV00001-server.log` | DPI transfer and installer log |

## Build recipe

Requirements: CMake 3.20+, Ninja, Git, Python 3.9+, and the
[PS5 Payload SDK](https://github.com/ps5-payload-dev/sdk). Node/npm only if the
WebUI is being rebuilt.

The SDK ships a prebuilt toolchain; no SDK build step is required:

```bash
curl -fsSL -o sdk.zip \
  https://github.com/ps5-payload-dev/sdk/releases/latest/download/ps5-payload-sdk.zip
unzip -q sdk.zip -d sdkroot      # -> sdkroot/ps5-payload-sdk
```

The host needs clang and `llvm-config` on `PATH`; `bin/prospero-llvm-config`
probes `llvm-config-21` down to `llvm-config-18`. A container with
clang-18/lld-18/llvm-18, cmake, ninja, and python3-pyelftools is sufficient.

```bash
git clone https://github.com/OnionBuddies/onionHEN-dpiv2-plugin.git
cd onionHEN-dpiv2-plugin

export PS5_PAYLOAD_SDK=/path/to/sdkroot/ps5-payload-sdk
cmake --preset ps5
cmake --build --preset ps5
# -> build-ps5/bin/dpiv2.elf
```

The build validates the embedded descriptor automatically. Upstream pins the
OnionHEN plugin SDK to a tested commit and fetches it via CMake `FetchContent`
(full clone), so the first configure needs network access. To use a local
checkout instead:

```bash
cmake --preset ps5 -DONIONHEN_PLUGIN_SDK_SOURCE=/path/to/onionHEN-plugin-sdk
```

To rebuild the embedded browser bundle:

```bash
cd webui && npm ci && npm run build
```

`webui/dist/index.html` is a single-file production bundle embedded in the ELF
at build time. Upstream commits the bundle, so a frontend change is not required
for a normal build.

## Attribution

This ELF is an unmodified-source build of
[OnionBuddies/onionHEN-dpiv2-plugin](https://github.com/OnionBuddies/onionHEN-dpiv2-plugin),
licensed GPLv3. No source modifications were made. Upstream retains all rights to
the plugin; this repository redistributes a compiled artifact only.

The plugin embeds `third_party/pkgserver`, also GPLv3 per upstream notices.

OnionHEN: <https://github.com/aydencharles/onionHEN>