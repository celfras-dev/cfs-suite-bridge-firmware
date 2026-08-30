# CFS-NANAODAP bridge firmware

Published firmware for the CFS-NANAODAP bridge board, and the manifest the
PC app reads to find it.

## What is here

| file | what it is |
|---|---|
| `manifest.json` | what the latest version is, and where to get it |
| `CFS-NANAODAP-BRIDGE_V*_IMG.bin` | a bridge firmware image, ready to send over USB |

## How the PC app uses this

After you connect a bridge, the app reads the board's firmware version, then
reads `manifest.json`. If the board is behind it says so in its log and offers
an `UPDATE BRIDGE FW` button. Nothing is ever installed without a click.

The endpoint is configurable, so this repository is not baked into the app:

```bash
NANAODAP_UPDATE_URL=https://raw.githubusercontent.com/celfras-dev/cfs-suite-bridge-firmware/main/manifest.json
```

## About the `_IMG.bin` files

An `_IMG.bin` is a slot image: a 1 KB header page followed by the firmware
body. It is what the board's bootloader accepts over USB.

**Do not program it over SWD.** It starts with the image header rather than a
vector table, so at `0x08000000` the initial stack pointer reads as `0x42534643`
(`'CFSB'`) and the board will not boot. The file to program over SWD is the
`FACTORY` image, which is not published here.

## Integrity

Each release carries a `sha256` and the app refuses an image that does not
match it. Images are **not signed yet** — the manifest and the image header
both reserve the fields for it, currently null. Until signing is switched on,
the trust boundary is HTTPS and this repository's write access.

## If the app refuses to update your board

From **v2.1.15** the PC app asks the board which BOARD it is before it sends
anything, and refuses rather than guess. There is now more than one bridge
board and they share a USB VID:PID, so sending the wrong binary was possible
and only the bootloader's own check stopped it.

A board whose **bootloader** predates command set 2.4.0 cannot answer that
question, so it is refused an update over USB. This is not a fault in the
board and nothing is wrong with it -- OTA never replaces the bootloader, so a
board updated only over USB still has its original one.

The fix is a one-time SWD flash with that board's `FACTORY` image, which
carries both the bootloader and the app. After that, USB updates work as
before. `FACTORY` images are not published here; they come from the firmware
build.

`update_fw.py --status` prints a `board` line when the board can answer, and
the tool says which version it needs when it cannot.

## Publishing a new version

The manifest is generated from the images rather than hand-edited, so the
version, size and hash cannot disagree with the file they describe:

```bash
python tools/mkmanifest.py \
  --base-url https://raw.githubusercontent.com/celfras-dev/cfs-suite-bridge-firmware/main \
  CFS-NANAODAP-BRIDGE_V<x.y.z>_IMG.bin \
  -o manifest.json
```

`mkmanifest.py` lives in the PC app repository.

Note that `raw.githubusercontent.com` caches for about five minutes, so a
freshly pushed manifest is not visible instantly.
