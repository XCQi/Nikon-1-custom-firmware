# Patched-v07 — Nikon 1 J5 C1.01

English | [简体中文](README.zh-CN.md) · [Project home](../../README.md)

Patched-v07 adds focus magnification, A-mode capture and Auto ISO for mechanical lenses on the J5, without a Dandelion chip or electronic adapter. Nikon 1 J5 only; installation has been tested from C1.01.

## Added features

- **Focus magnification in M and A**: press OK to enter and use the directional buttons to move the magnified area.
- **Magnification stays open**: no automatic timeout; half-press the shutter or press OK to exit.
- **A-mode capture**: set aperture on the lens and let the camera choose shutter speed. Photographs can be taken and saved normally.
- **Auto ISO in A**: select an Axxx option to let ISO adjust with brightness. Fixed ISO also works normally.

Aperture is still adjusted with the lens aperture ring. The OK hint or aperture value may be absent from the screen; this does not prevent use.

## Downloads

Download the file needed from the [J5 Patched-v07 release](https://github.com/XCQi/Nikon-1-custom-firmware/releases/tag/j5-patched-v07):

| File | Purpose |
|---|---|
| `Patched-v07_J5_0101.bin` | Install the custom firmware |
| `Stock-restore_J5_0101.bin` | Restore stock functions |

Keep only the firmware being installed on the card, to avoid mixing them up.

## Installation

Have a charged battery and card reader ready, and back up the photographs on the card. If the camera is on C1.00, first update to C1.01 using [Nikon's official instructions](https://downloadcenter.nikonimglib.com/en/download/fw/234.html). Do not interrupt power or remove the card during the update.

1. Download `Patched-v07_J5_0101.bin` and copy it to the root of the memory card.
2. Insert the card, turn the camera on and open Setup → Firmware version → Update.
3. Follow the on-screen instructions to complete the update and turn the camera off.
4. Delete the update file from the card, then turn the camera on again.

With a mechanical lens attached, press OK in M or A for focus magnification. For Auto ISO, select an Axxx option in the ISO menu. EXIF from bright and dark photographs can be used to check ISO changes.

After installation, the version screen still shows C1.01 and the update entry may remain available. Check the actual features to confirm installation.

## Restoring stock functions

Download `Stock-restore_J5_0101.bin`, copy it to the card root and follow the same update steps. Delete the update file when finished and restart. The magnification, A-mode capture and Auto ISO modifications are removed.

The restoration package has been tested. It retains stock functional code and only adjusts update recognition and checksums so it can be installed on C1.01. An untouched official C1.01 file may be rejected because its version matches. The version screen still shows C1.01 after restoration.

This method requires a camera that boots normally and can open the update menu. It cannot repair an unbootable camera.

## Tested scope

The features above have been tested on one J5. Fixed ISO and M-mode capture also worked normally. Electronic-lens combinations, all Auto-ISO caps and long-term reliability have not been fully tested.

This release does not change S/P modes or 4K recording specifications, and cannot be used on other Nikon 1 models. It is unofficial experimental firmware.

## Changelog

### Patched-v07: First public release

- Adds M/A focus magnification without an automatic timeout.
- Adds A-mode capture and Auto ISO for mechanical lenses.
- Includes firmware for restoring stock functions.

v1–v6 were used during research and are not available for download.

## Reporting problems

Include lens and adapter model, shooting mode, ISO setting and steps to reproduce the problem. For exposure issues, include EXIF shutter speed and ISO. The camera serial number is not needed.

See the [development document](../../docs/DEVELOPMENT.md) for the implementation.
