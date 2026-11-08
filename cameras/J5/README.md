# Nikon 1 J5 C1.01 — patched v11

English | [简体中文](README.zh-CN.md) · [Project home](../../README.md)

patched v11 adds shooting-only assignments for the Wi-Fi and Delete buttons, configurable MF magnification behavior, and a language-menu unlock. It retains all v10 features. Nikon 1 J5 only, based on C1.01.

## New in v11

Open **Setup → Custom**, at the end of the Setup menu:

| Setting | What it does |
|---|---|
| Assign Wi-Fi button | Choose one of the seven original Fn functions, or Wi-Fi to restore its original role. |
| Assign Delete button | Choose one of the seven original Fn functions. Only the shooting-screen behavior changes. |
| Half-press cancel zoom (MF) | **On**: half-press exits MF magnification. **Off**: half-press and release keep it open; a full press takes a photograph. |
| Keep MF zoom | **On**: disables the automatic timeout. **Off**: uses the original timeout of about five seconds. |

The original Fn button keeps its own assignment. Wi-Fi and Delete remapping applies only on the shooting screen; playback retains their original functions. The assignments and both MF settings are saved across power cycles.

On first initialization, **Half-press cancel zoom (MF) is On** and **Keep MF zoom is Off**. Previously saved choices are retained when updating. Unlike earlier versions' mechanical-lens-only hold behavior, v11 uses the same hold setting for native lenses, FT1 and mechanical lenses. Select **Keep MF zoom → On** if you want unlimited focusing time. Press OK to exit magnification.

With half-press cancellation Off, a registered full press completes a single-shot request even if the button is released quickly. Magnification ends for capture; it is not restored automatically after the photograph. This behavior was confirmed in the test15 build used as the release basis. The release removes its diagnostic captions; the shooting logic is unchanged.

**Language menu:** removes the regional visibility restriction on the firmware's 32 language choices, intended to make additional languages available on Japanese-market cameras. Japanese-market hardware has not yet been tested; feedback is welcome. Button-assignment labels use the firmware's existing language resources. The two new MF setting names are provided in Simplified Chinese, Traditional Chinese, Japanese and English; other languages currently use English for those names. Menu headings and selection summaries have also been corrected.

## Features retained from v09

- **Partial FT1 AF-area unlock**: with a lens that supports autofocus through FT1, press OK and use the directional buttons, or use touch selection, to move the single AF point. With touch shutter enabled, tapping the screen focuses and takes a photograph at the selected point. This build confines FT1 single-point AF selection to the firmware’s existing central set of 105 points: an out-of-range touch snaps to the nearest point, directional movement stops at the boundary, and step selection skips out-of-range positions. A J5 with FT1 and a Nikon 16–85mm has focused and saved photographs at central and some non-central points; not all 105 points have been tested individually. Auto-area, face-priority and tracking modes are not unlocked.
- **Intervals as short as one second**: adds 1–4-second settings while retaining longer intervals. In the standalone test firmware, both 1-second and 2-second sequences saved the requested 20 RAW photographs with short exposures and MF, at the expected intervals. Sustained speed still depends on buffering, card writing, exposure and focusing time. Card-write performance is the same as v08.
- **FT1 lens-restriction unlock (preview)**: some lenses previously rejected by the body can be tried. A forum tester reported MF photography in M/A and video with the Nikon 200–500mm, with saved photographs, VR and electronic aperture stop-down during capture working. AF remains unavailable; set the lens to MF. The Sigma 105mm f/2.8 OS still reports “Lens error”; this is unresolved. Other lenses need individual compatibility testing.

**Known FT1 limitation:** with AF-S and touch shutter enabled, a low-contrast subject can leave the camera waiting for focus. MENU and another screen touch may not cancel the wait. Point snapping does not fix this. A higher-contrast subject may allow capture to complete; if the controls remain unresponsive, turn the camera off and on. This issue is deferred to a later release.

## Retained mechanical-lens features

These features do not require a Dandelion chip or electronic adapter.

- **Focus magnification in M and A**: press OK to enter and use the directional buttons to move the magnified area.
- **Configurable magnification duration and exit behavior**: use the two MF settings described above.
- **A-mode capture**: set aperture on the lens and let the camera choose shutter speed. Photographs can be taken and saved normally.
- **Auto ISO in M and A**: select an Axxx option to let ISO adjust with brightness. In M, shutter speed stays at the value set on the camera and aperture stays under manual control on the lens. Fixed ISO also works normally.

Aperture is still adjusted with the lens aperture ring. The OK hint or aperture value may be absent from the screen; this does not prevent use.

## Downloads

Download the file needed from the [J5 patched v11 release](https://github.com/XCQi/Nikon-1-custom-firmware/releases/tag/j5-patched-v11):

| File | Purpose |
|---|---|
| `J5_0101_patched_v11.bin` | Install the custom firmware |
| `J5_0101_stock_restore.bin` | Restore stock functions |

Keep only the firmware being installed on the card, to avoid mixing them up. To upgrade from patched v07, v08, v09 or v10, install v11 using the same steps; there is no need to restore stock functions first.

## Installation

Have a charged battery and card reader ready, and back up the photographs on the card. If the camera is on C1.00, first update to C1.01 using [Nikon's official instructions](https://downloadcenter.nikonimglib.com/en/download/fw/234.html). Do not interrupt power or remove the card during the update.

1. Download `J5_0101_patched_v11.bin` and copy it to the root of the memory card.
2. Insert the card, turn the camera on and open Setup → Firmware version → Update.
3. Follow the on-screen instructions to complete the update and turn the camera off.
4. Delete the update file from the card, then turn the camera on again.

With a mechanical lens attached, press OK in M or A for focus magnification. For Auto ISO in M or A, select an Axxx option in the ISO menu. EXIF from bright and dark photographs can be used to check ISO changes.

After installation, the version screen still shows C1.01 and the update entry may remain available. Check the actual features to confirm installation.

## Restoring stock functions

Exit interval shooting and set the interval back to at least five seconds first. Download `J5_0101_stock_restore.bin`, copy it to the card root and follow the same update steps. Delete the update file when finished and restart. All custom features are removed. The restoration file is unchanged from v07 and v08.

To return to v08, also set the interval back to at least five seconds first, then install `J5_0101_patched_v08.bin` from the [v08 release](https://github.com/XCQi/Nikon-1-custom-firmware/releases/tag/j5-patched-v08).

The restoration package has been tested. It retains stock functional code and only adjusts update recognition and checksums so it can be installed on C1.01. An untouched official C1.01 file may be rejected because its version matches. The version screen still shows C1.01 after restoration.

This method requires a camera that boots normally and can open the update menu. It cannot repair an unbootable camera.

## Tested scope

The mechanical-lens features have been tested on one J5; fixed ISO and M-mode capture also worked normally. The known FT1 and short-interval test conditions are listed above. They do not establish performance with other lenses, longer sequences or different settings. All Auto-ISO caps and long-term reliability have not been fully tested.

Mechanical-lens support in S/P is unchanged from v08. This firmware cannot be used on other Nikon 1 models. It is unofficial experimental firmware.

## Changelog

### patched v11: Custom controls and MF magnification settings

- Groups the new settings under Setup → Custom, after Firmware version.
- Adds a shooting-only Delete assignment and makes Wi-Fi remapping shooting-only.
- Adds persistent half-press cancellation and MF timeout settings for all lens types.
- Completes registered short full-press requests when half-press cancellation is Off.
- Unlocks regional language-menu choices; Japanese-market testing is still pending.
- Corrects custom menu headings, state summaries and Chinese glyphs; removes diagnostic captions.
- Retains all v10 features and the same restoration file.

### patched v10: Independently assignable Wi-Fi/Fn2 button

- Adds a separate Fn2 assignment menu with the seven original Fn choices and a Wi-Fi option.
- Saves the assignment across power cycles and includes labels for all 32 languages.
- Retains all v09 features and the same restoration file.

### patched v09: FT1 extensions and shorter intervals

- Adds FT1 single-point selection by buttons and touch, bounded to the central set of 105 points.
- Lowers the interval minimum from five seconds to one and adjusts preparation timing for 1- and 2-second settings.
- Adds the FT1 lens-restriction unlock preview, with the known compatibility limits above.
- Retains the v08 mechanical-lens features and the existing stock-functions restoration firmware.

### patched v08: M-mode Auto ISO

- Adds Auto ISO (Axxx) for mechanical lenses in M mode; tested on the camera.
- Retains v07 focus magnification, A-mode capture and A-mode Auto ISO.
- Uses the same stock-functions restoration firmware as v07.

### patched v07: First public release

- Adds M/A focus magnification without an automatic timeout.
- Adds A-mode capture and Auto ISO for mechanical lenses.
- Includes firmware for restoring stock functions.

v1–v6 were used during research and are not available for download.

## Reporting problems

Include lens and adapter model, shooting mode, ISO setting and steps to reproduce the problem. For exposure issues, include EXIF shutter speed and ISO. The camera serial number is not needed.

See the [development document](../../docs/DEVELOPMENT.md) for the implementation.
