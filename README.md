# TVRa1n

<p align="center">
  <img src="icon.png" width="180" alt="TVRa1n icon">
</p>

A native macOS app that automates the full tethered jailbreak chain for the
**Apple TV 4K (2nd generation)** — `AppleTV11,1`, A12 / T8020 — on tvOS 26.6.

One button: DFU detection → checkm8 boot chain → bootstrap → package
installation. Fully offline — every artifact ships inside the app bundle.

![platform](https://img.shields.io/badge/platform-macOS-lightgrey)
![device](https://img.shields.io/badge/device-AppleTV11%2C1-blue)
![tvOS](https://img.shields.io/badge/tvOS-26.6-orange)

## What it does

```
DFU detect → iBSS (patched) → yoloDFU → PongoOS → KPF → jbinit → bootstrap → packages
```

1. **Prepare** — verifies all bundled components are present
2. **DFU Mode** — detects the Apple TV in PWND DFU over USB (IOKit)
3. **Boot Chain** — runs the checkm8 boot chain: patched iBSS → yoloDFU →
   PongoOS → KPF → `bootx`
4. **Bootstrap** — installs a rootless Procursus bootstrap into `/var/jb`
   via preboot, with automatic retry on transient failures
5. **Packages** — installs bundled `.deb` packages over SSH, auto-detecting
   and converting rootful packages on the fly

Everything after DFU entry runs unattended.

## Requirements

**Hardware:**
- Apple TV 4K 2nd gen (`AppleTV11,1`, A12/T8020) on tvOS 26.6
- A DCSD cable (or equivalent) to enter DFU
- A Foxlink X892 (or similar) breakout adapter for the hidden Lightning port
- A Waveshare RP2350 USB-A board, flashed with `usbliter8`
  (T8020 trampoline-preserve build — included/documented in this repo)

**Software:**
- macOS with `irecovery`, `python3`, and `pyusb` installed
- No Xcode required to run the pre-built app

## Installation

Download the latest release, then:

```sh
xattr -cr TVRa1n.app
```

(The app isn't notarized — macOS Gatekeeper will otherwise refuse to open it.
Alternatively: right-click → Open → Open Anyway on first launch.)

Move it to `/Applications` and launch.

## Usage

1. Enter your Apple TV into **PWND DFU**:
   - DCSD → Foxlink → Apple TV → power on (enters DFU)
   - Disconnect DCSD, connect the flashed RP2350 → wait for the green LED
   - Disconnect the RP2350, connect the Mac directly via Lightning → Foxlink
2. Open **TVRa1n**, click **Run all**
3. Once the boot chain completes, enter your Apple TV's IP address and press
   Enter (or click the arrow) to run Bootstrap + Packages automatically
4. Done — SSH is available on port 44, default password `alpine`

The jailbreak is **tethered**: repeat step 1–2 after every reboot.

## Bundled components

All artifacts are bundled inside the app — nothing is downloaded at runtime
except the rootless bootstrap on first use (also cacheable offline):

| Component | Purpose |
|---|---|
| `ibss.yolodfu.bin` | Patched iBSS with yoloDFU wrapper/runtime |
| `pongo-container.bin` | Compressed PongoOS payload |
| `checkra1n-kpf-pongo` | Kernel patchfinder module |
| `ramdisk.dmg` / `binpack.dmg` | jbinit userspace bootstrap payloads |
| `usbliter8.uf2` | Firmware for the RP2350 checkm8 delivery board |
| `bootstrap-ssh-iphoneos-arm64.tar.zst` | Rootless Procursus bootstrap |

## Building the boot chain artifacts yourself

If you want to rebuild the artifacts from source (e.g. for a different tvOS
build), see [atv2nd-jailbreak](https://github.com/Fauxly/atv2nd-jailbreak)
for the underlying component repositories (`usbliter8`, `yolodfu`, `PongoOS`,
`jbinit`) and the `Makefile`/`boot.sh` this app wraps.

## Credits

Built on top of:
- [usbliter8](https://github.com/Fauxly/usbliter8) — checkm8 delivery via
  RP2350, with T8020 trampoline preservation
- [yoloDFU](https://github.com/Fauxly/yolodfu) — EL1 runtime and PongoOS
  transport
- [PongoOS](https://github.com/Fauxly/PongoOS) / checkra1n KPF — kernel
  patchfinder
- [jbinit](https://github.com/Fauxly/jbinit) / palera1n — rootless userspace
  bootstrap

## Disclaimer

For security research, device recovery research, and devices you own or
have permission to test. Use responsibly and follow local laws.
