# Nikon 1 J5 — v7

English | [简体中文](README.zh-CN.md) · [Project home](../../README.md)

For mechanical/non-CPU lenses, without a Dandelion chip or electronic adapter. **Nikon 1 J5 only; the tested starting firmware is C1.01.**

## Custom features

- **Focus magnification in M / A**: press OK to enter; directional buttons move the magnified area.
- **Persistent magnification**: no automatic timeout; half-press the shutter or press OK to exit.
- **A-mode capture**: take and save photographs; set aperture on the lens and let the camera select shutter speed.
- **Auto ISO in A**: ISO can adjust with brightness, or you can choose a fixed ISO.

Aperture is still adjusted using the lens aperture ring. The OK hint or aperture value may be absent from the screen; the features work without them.

## Downloads

Open the [J5 v7 release](https://github.com/XCQi/Nikon-1-custom-firmware/releases/tag/j5-v7) and download the attachment you need:

| File | Purpose |
|---|---|
| `J5_v7.bin` | Install the v7 enhancements |
| `J5_stock_restore.bin` | Restore stock functions |

**Rename either file to `J5_0101.bin` before installation. Copy only the one you want to install, to avoid mixing them up.**

## Installing v7

1. Charge the battery, back up your photographs and have a card reader ready. If you are on C1.00, first update to C1.01 using [Nikon's official instructions](https://downloadcenter.nikonimglib.com/en/download/fw/234.html).
2. Download `J5_v7.bin`, rename it to `J5_0101.bin` and copy it to the **root of the memory card**.
3. Insert the card, turn the camera on and open **Setup → Firmware version → Update**. Follow the on-screen instructions. Do not interrupt power or remove the card.
4. Turn the camera off as instructed, delete the update file from the card, then turn it on again.

With a mechanical lens, OK should open persistent magnification in M/A; half-press or OK should exit. A should take and save photographs. For Auto ISO, select an Axxx option and check EXIF ISO in photographs of bright and dark scenes.

The version screen still shows **C1.01** and the update entry may remain available. This is expected; confirm installation by checking the actual features.

## Restoring stock functions

The restoration package has been tested successfully.

1. Download `J5_stock_restore.bin` and rename it to `J5_0101.bin`.
2. Replace the update file at the card root with this file and follow the same camera update steps above.
3. Delete the update file when finished and restart. Mechanical-lens enhancements are removed, returning stock restrictions.

Use this project's restoration package: an untouched official C1.01 file may be rejected because its version matches. The restoration package only adjusts update recognition and checksums; functional code remains stock, and the version screen still shows C1.01.

Restoration requires a camera that boots normally and can open the update menu. It is not a recovery method for an unbootable camera. A complete v7 reinstallation round trip after this particular restoration has not yet been tested.

## Testing and limitations

Testing so far is on one J5: the features above worked, fixed ISO and M-mode regression checks passed, and stock functions were restored successfully.

A-mode Auto ISO still does not vary with an empty FT1. Electronic-lens combinations, all Auto-ISO caps and long-term reliability have not been fully tested. This release does not unlock S/P, change 4K recording or support other Nikon 1 bodies. It is unofficial experimental firmware.

## Changelog

### v7 — First public release

- Combines M/A magnification, persistent magnification, A-mode capture and Auto ISO.
- Provides a separate stock-function restoration file.
- v1–v6 were research versions and are not offered as public installation versions.

## Feedback

Include lens/adapter model, shooting mode, ISO setting, reproduction steps and results. For exposure issues, include EXIF ISO/shutter speed. Do not share the camera serial number.

[Development notes](../../docs/DEVELOPMENT.md)
